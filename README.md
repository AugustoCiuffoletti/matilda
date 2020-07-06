## How to run the Lida V2.0 server

This HOWTO refers to running the Lida service on a generic host, not dependent on the Operating System provided it supports the Docker toolset, using simple terminal commands. Linux find a *lida.sh* script that automates them.

### Database installation

It is appropriate to install the database first, with the command:

    $ sudo docker run -d --name mongodb --network=host  \
	  -v "$(pwd)"/configdb:/data/configdb \
	  -v "$(pwd)"/db:/data/db \
	  mongo:bionic

which is a standard Docker container for mongodb. In case you have the mongo CLI interface installed, you can try connecting to the mongodb server by issuing the `mongo` command. The reply should start with a `connecting to: mongodb://127.0.0.1:27017`, which means that the database server is up and running.

### Lida server installation

Next you install the Lida server, with the command:

    $ sudo docker run -d --name lida --network=host mastrogeppetto/dockerlida:letstry

After that point the Lida service is up and running: you can test by accessing the host with a browser.

### Stop/start the service

To stop the service (e.g., for power saving) use the following command:

    $ sudo docker kill lida mongodb

To restart the service use the following command:

    $ sudo docker start lida mongodb

### Using the service

The first time you install and run the service, the systems will create an admin password for you: the admin account is root, the password is *admin*. You are prevented to remove the "root" account, but you can change the password. This is strongly encouraged if you are not planning to use this service as a sandbox.

### Backup, restore, share the database

The two directories named *db* and *dbconfig* contain your database. To make a backup copy you can create a zip archive containing the two directories, using a timestamp in the filename. To restore the database, unzip the archive. In the same way the database can be shared with collaborators. **Always kill the service while performing such operations**.
