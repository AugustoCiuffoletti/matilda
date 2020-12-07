# How to run the Matilda server

This HOWTO refers to installing the Matilda inter-annotation service for personal use on a PC, possibly on a VM hosted locally: the user will access the Matilda service using a browser at URL `https://127.0.0.1`. See the Matilda user manual for further reference.

Being based on Dockers, the service is portable across Operating Systems, and system requirements are inherited from the Docker toolset. In particular, we recall that 32-bits PCs are not supported by Docker. However Docker is available for Linux, MacOS and Windows on 64-bit machines. The following installation instructions depend on the *git* command, which is also available for the major OSs.

The following refers to a local installation, but there are other installation options: see the Appendix for

- Installation on a remote server
- Using an external database
- Deployment on an OpenStack infrastructure

## Install and run the Matilda inter-annotator service locally

Before proceeding, check the availability of the *docker* and *docker-compose* commands. See below OS-specific suggestions.

Using the *git* command, clone this repository (or download and uncompress the zipfile), and enter the *matilda* directory.

    $ git clone https://github.com/AugustoCiuffoletti/matilda
    $ cd matilda

Create a self-signed certificate to implement a basic security measure: for this, enter the *nginx_conf* directory and issue the following openssl command:

    $ cd nginx_conf
    $ openssl req -subj '/CN=localhost' -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -days 365
    
which creates a self-signed certificate to encrypt client-server communication. This does not preclude a "men in the middle" attack, allowing an intruder to take the place of your server. However, this is a reasonable tradeoff between security and simplicity. The certificate must be renewed after one year with the same command.

Go back to the root and descend the *lida2_conf*. Here you find a *conf.json* file that customizes the Matilda server operation. In a local installation you do not need to modify this file.

Go back to the `matilda` directory and launch the Matilda server with a *docker-compose* command:

    $ cd ..
    $ sudo docker-compose up -d
    
Unless you manually stop the service for some reason, it will be automatically restarted at every boot. So the server can be switched off/on without intervention of the administrator.

To manually stop the service use the command:

    $ sudo docker-compose kill

If you manually stop the service, you need to restart it before shutting down the server in order to enable the auto-start feature.

The Dockers are configured to automaticaly upgrade to the last released version of the dockers. If you want to disable such feature, remove the *watchtower* stanza from the *docker-compose.yml* file. 

To have an instant check of system state use `docker-compose ps`:

    $ sudo docker-compose ps
    Name                 Command                         State   Ports
    ------------------------------------------------------------------
    matilda_lida_1      ./gunicorn_run.sh                Up           
    matilda_mongodb_1   docker-entrypoint.sh mongod      Up           
    matilda_nginx_1     /docker-entrypoint.sh ngin ...   Up           

For a more detailed view of the log use `docker-compose logs`.

On the first run, the server creates an administration account for you: the username is "admin", the password is *admin*. You are prevented from removing the *admin* account, but you can change the password.

## Improving security

Use a certificate provided by a Certification Authority (like [Letsencrypt](https://letsencrypt.org), which provides free certificates which must be renewed every 90 days) and copy it in the *nginx_conf* directory.

# OS specific suggestions

## Ubuntu

The server has been tested with Ubuntu 18.04 (Bionic Beaver). We advice using the snap distribution of the docker package, since the apt one gave us minor problems. Therefore, after installing a bare minimal Ubuntu, install the needed software with:

    # sudo apt update
    # sudo apt upgrade
    # sudo apt install git snapd
    # sudo snap install docker

# APPENDIX: alternate deployments

The following deployment alternatives allow some degree of concurrent operation on the server and the database: this feature is currently under test.

Since the installation is exposed to remote access the administrator has to change the `admin` password.

### Installation on a remote server

The installation path is the same as for the local installation: the Matilda service will be reachable on port 80 (HTTP) and 443 (HTTPS) with the FQDN corresponding to the host where it is installed.

The administrator needs to tweak the `lida2_conf/conf.json` file. The default content, as found in this repo, is:

    {
        "app": {
		    "address":"localhost",
		    "port":5000
	    },
	    "database": {
		    "name": "lida",
		    "legacy_configuration": {
			    "address": "localhost",
			    "port": 27017,
			    "username": null,
			    "password": null
		    },
		    "optional_uri": null
	    }
    }

You need to modify the`"address":"localhost"` line by replacing `localhost` with the address of the remote host, or `0.0.0.0` to indicate *any* network interface)

### Using an external database

The following instructions apply both to the remote and to the local server installation, and allow to use an external MongoDB server, either on user premises or in the cloud (e.g., the Atlas MongoDB service). This option provides extra reliability, and expandibility.

To configure Matilda for this kind of operation the administrator needs to tweak the file lida2_conf/conf.json file. The default configuration is reported below:

    {
        "app": {
		    "address":"localhost",
		    "port":5000
	    },
	    "database": {
		    "name": "lida",
		    "legacy_configuration": {
			    "address": "localhost",
			    "port": 27017,
			    "username": null,
			    "password": null
		    },
		    
	    }
    }

You need to modify the`"optional_uri": null` line by replacing `null` the address of the URI OF THE MongoDB service.

To avoid running a useless MongoDB server on the server host, you should remove the `MOngoDB` stanza from the `docker-compose.yml`, corresponding to the lines:

    mongodb:
      image: mongo:bionic
      volumes:
        - ./configdb:/data/configdb
        - ./db:/data/db
      network_mode: "host"
      restart: always
      logging:
        driver: "json-file"

# Deploy on an OpenStack 

OpenStack is a open cloud infrastructures: many universities and research institutes offer cloud resources using local or federated OpenStack deployments.

An OpenStack-based deployment of Matilda usually addresses the use of two instances: one for the Matilda server and proxy, another for the MongoDB server. Optionally, a public MongoDB service can be used, by configuring the conf.json file as seen above. The following applies to the allocation of both services inside OpenStack.

For the creation of the MongoDB server instance you can refer to this [guide](https://gist.github.com/AugustoCiuffoletti/8218b9deb993834bc30bc048df1f4d62). In principle this instance does not need a Floating IP, since it is accessed only by the server, and should have a specific Security Group with only the MongoDB ports enabled (27017-27019) as Ingress from the Security Group of the Matilda server.

The Matilda server instance needs to have a Floating IP in order to be accessible from the outside. Its Security Group has ports 22 (SSH), 80 (HTTP) and 443 (HTTPS) enabled as Ingress from Remote IP Prefixes. Its configuration follows the guidelines given above for the installation on a remote server across the SSH link. You can use an Ubuntu server image and follow the OS specific instructions given above in order to install `git` and the Docker support. 

