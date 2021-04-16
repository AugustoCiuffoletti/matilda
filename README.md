# Running Matilda with an external database

This HOWTO refers to an installation of the Matilda service using a MongoDB service run by an independent host. We tested such configuration in a Openstack IaaS infrastructure, an we will make reference to this use case, but the same should be applicable also to other scenarios, like a Matilda service taht uses the Atlas MongodDB, or a shared server.

## Configuring the infrastructure

We used two c1.small instances: one for matilda with the nginx proxy, the other with the MongoDB server.

The two hosts are connected by the OpenStack intranet. Only one floating IP is needed to have access to the matilda instance: its "security group" should grant access to port 22 (SSH), 80 (HTTP) and 443 (HTTPS). The "security group" of the MongoDB instance has the 27017-9 ports configured as ingress from the "security group of the matilda security group. Otionally, another floating IP can be associated to the MongoDB instance, and the corresponding port should be added to the "security group" as outbound.

The matilda instance hosts two dockers: one for matilda, the other for nginx; the two images are the same of the all-in-one solution. Besides the Dockers, the matilda instance depends on two packages: the docker package from the snap repository, and the mongo-clients package from an apt repository. The former is needed to run the repositories, the second to interact with the database for administration.

The mongodb instance hosts the mongodb server. The installation of the MongoDB service over an OpenStack instance is covered in a [gist] (https://gist.github.com/AugustoCiuffoletti/8218b9deb993834bc30bc048df1f4d62). 


You should necessarily configure the user for matilda access, and an optional user to perform database dumps.



This HOWTO refers to installing the Matilda inter-annotation service on a generic host, not dependent on the Operating System, provided it supports the *git* command and the Docker toolset. Both the *docker* and *docker-compose* commands must be available. See OS specific suggestions below.

## Install and run the Matilda inter-annotator service

Using the *git* command, clone this repository (or download and uncompress the zipfile), and enter the *matilda* directory.

    $ git clone https://github.com/AugustoCiuffoletti/matilda
    $ cd matilda

Create a self-signed certificate to implement a basic security measure: for this, enter the *nginx_conf* directory and issue the following openssl command:

    $ cd nginx_conf
    $ openssl req -subj '/CN=localhost' -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -days 365
    
which creates a self-signed certificate to encrypt client-server communication. This does not preclude a "men in the middle" attack, allowing an intruder to take the place of your server. However, this is a reasonable tradeoff between security and simplicity. The certificate must be renewed after one year with the same command.

Go back to the root and run Matilda with a *docker-compose* command:

    $ cd ..
    $ sudo docker-compose up -d
    
Unless you manually stop the service for some reason, it will be automatically started at the next boot. So the server cab be switched off/on without intervention of the administrator.

To manually stop the service use the command:

    $ sudo docker-compose kill

If you manually stop the service, you need to restart it before shutting down the server in order to auto-start it at boot.

To have an instant check of system state use `docker-compose ps`:

    $ sudo docker-compose ps
    Name                 Command                         State   Ports
    ------------------------------------------------------------------
    matilda_lida_1      ./gunicorn_run.sh                Up           
    matilda_mongodb_1   docker-entrypoint.sh mongod      Up           
    matilda_nginx_1     /docker-entrypoint.sh ngin ...   Up           

For a more detailed view of the log use `docker-compose logs`.

On the first run, the server creates an administration account for you: the username is root, the password is *admin*. You are prevented from removing the *root* account, but you can change the password. This is strongly encouraged if you are not planning to use this service as a sandbox.

To access the service with your browser, use the URL `https://<address>` replacing *address* with the IP address or the hostname of the server (*localhost* included). You can use `http.//` as well, with reduced security.

## Improving security

Use a certificate provided by a Certification Authority (like [Letsencrypt](https://letsencrypt.org), which provides free certificates which must be renewed every 90 days) and copy it in the *nginx_conf* directory.

## Backup, restore, share the database

*DA TESTARE - TO BE TESTED*

The two directories named *db* and *dbconfig* contain your database. To make a backup copy you can create a zip archive containing the two directories, using a timestamp in the filename. To restore the database, unzip the archive. In the same way the database can be shared with collaborators. **Always kill the service while performing such operations**.

# OS specific suggestions

## Ubuntu

The server has been tested with ubuntu 18.04 (Bionic Beaver). We advice using the snap distribution of the docker package, since the apt one gave us minor problems. Therefore after installing a bare minimal Ubuntu, install the needed software with:

    # sudo apt update
    # sudo apt upgrade
    # sudo apt install git snapd
    # sudo snap install docker
    
The server needs 5GB on the hard disk, plus the space needed by the database. 
