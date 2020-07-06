# How to run the Matilda server

This HOWTO refers to installing the Matilda inter-annotation service on a generic host, not dependent on the Operating System, provided it supports the *git* command and the Docker toolset. Both the *docker* and *docker-compose* commands must be available.

## Install and run the Matilda inter-annotator service

Using the *git* command, clone this repository (or download and uncompress the zipfile), and enter the *matilda* directory.

    $ git clone https://github.com/AugustoCiuffoletti/matilda
    $ cd matilda

Create a self-signed certificate to implement a basic security measure: for this, enter the *nginx_conf* directory and issue the following openssl command:

    $ cd nginx_conf
    $ openssl req -subj '/CN=localhost' -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -days 365
    
which creates a self-signed certificate to encrypt the user-server communication. This does not preclude a "men in the middle" attack, allowing an intruder to take the place of your server. This is a reasonable tradeoff between security and simplicity.

Go back to the root and run Matilda with a *docker-compose* command:

    $ cd ..
    $ sudo docker-compose up

On the first run, the server creates an administration account for you: the username is root, the password is *admin*. You are prevented from removing the *root* account, but you can change the password. This is strongly encouraged if you are not planning to use this service as a sandbox.

To access the service with your browser, use the URL `https://<address>` replacing *address* with the IP address or the hostname of the server (*localhost* included).

## Improving security

Use a certificate provided by a Certification Authority (like [Letsencrypt](https://letsencrypt.org), which provides free certificates which must be renewed every 90 days) and copy it in the *nginx_conf* directory.

## Backup, restore, share the database

*DA TESTARE - TO BE TESTED*

The two directories named *db* and *dbconfig* contain your database. To make a backup copy you can create a zip archive containing the two directories, using a timestamp in the filename. To restore the database, unzip the archive. In the same way the database can be shared with collaborators. **Always kill the service while performing such operations**.
