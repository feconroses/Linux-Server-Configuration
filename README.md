# About

This project took a baseline installation of a Linux distribution on a virtual machine and prepared it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application can be seen live in the following URL: https://catalog.growthandtraction.com/

![](https://i.ibb.co/VpXVPPq/Screen-Shot-2019-06-09-at-12-11-27-PM.png)


## Table of Contents

- [Information for the Grader User](#information-for-the-grader-user)
- [Selecting a server host](#selecting-a-server-host)
- [Setting up the server](#setting-up-the-server)
- [Deploying the catalog app](#deploying-the-catalog-app)
- [Domain name](#setting-up-a-subdomain)
- [Configuring SSL](#configuring-ssl)
- [Adjusting OAuth from Google](#adjusting-oauth-from-google)
- [Third party resources](#third-party-resources)


## Information for the grader user

i. The IP address and SSH port so the reviewer can access the server: 
IP: 206.189.173.114
Port: 2241

ii. Information to login to `grader` user:
User: grader
Paswword: udacityrocks1!
SSH Keys: the SSH key is submitted with the project.

iii. The complete URL to the hosted web application: 
URL: https://catalog.growthandtraction.com/


## Selecting a server host

Tried Amazon Lightsail but it didn't seem as intuitive as I would have expected. So, I preferred to use Digital Ocean which I was already familiar with.


## Setting up the server

The following is a summary of the configuration steps made to the server:

1. Updated available package lists
2. Upgraded installed packages
3. Removed unneeded packages
4. Installed finger
5. Create a new user called `grader`
6. Used `usermod` command to add the `grader` user to the sudo group
7. Creaded .ssh folder and authorized_keys file for the user `grader`
8. Changed .ssh folder permissions to 700 and authorized_keys file to 644 and `grader` user as the owner
9. Created a SSH key pairs for `grader`
10. Copied SSH public keys to the server using `ssh-copy-id`
11. Disabled password-based authentication to the server to enforce key-based SSH authentication
12. Disabled remote login of the root user
13. Changed the default port to 2241
14. Restarted SSH
14. Created a server firewall to only allow incoming connections for SSH (port 2200 and 2241), WWW, HTTP (port 80), and NTP (port 123). Also allowed outgoing as default.


## Deploying the Catalog App

Next, I needed to get the catalog app up and running on the Ubuntu server. For this, I followed these steps:

1. Installed apache
2. Installed python 3.7 and pip3
3. Installed Flask and mod_wsgi. 
4. Created a Flask application following the steps on this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps). This includes creating a __init__.py file, a virtual host config file `FlaskApp.conf` in the folder /etc/apache2/sites-available and enabling the virtual host, and also creating a `flaskapp.wsgi` file.
5. Restarted Apache
5. Git cloned the catalog app from my [repository](https://github.com/feconroses/item_catalog)
6. Installed needed modules and packages for the catalog app (httplib2, requests, sqlalchemy, oauth2client, and httplib2)
6. Renamed application.py as __init__.py
7. Deleted previous catalog.db file (which was cloned from the repo)
8. Installed and configured PostgreSQL. Logged in as default user postregs, connnected to the system using `psql`, created a postgres user `catalog` with password. Give the user permissions to create a database. Then, created a database named `catalog` with the owner `catalog`. Connected to the database catalog using `\c catalog` and revoked all rights `REVOKE ALL ON SCHEMA public FROM public;`. Also, granted only access to the catalog role `GRANT ALL ON SCHEMA public TO catalog;`. Finally exited PostresSQL.
9. Created a new PostgreSQL database schema using `python3 database_setup.py`
10. Installed psycopg2. 
11. Changed the line starting with "engine" to `engine = create_engine('postgresql://catalog:DB-PASSWORD@localhost/catalog')`
12. Changed the relative URLs from the __init__.py to absolute (e.g. from `client_secrets.json` to `/var/www/FlaskApp/FlaskApp/client_secrets.json`)
13. Added a `.` to the relative URLs from the import packages in __init__.py (e.g. changed `from database_setup import Base, Category, CategoryItem, User` to `from .database_setup import Base, Category, CategoryItem, User`)
14. Restarted apache. 


## Setting up a subdomain

Already had the domain https://growthandtraction.com, so I decided to use a subdomain catalog.growthandtraction for serving this catalog app. Created a A record and pointed it to the server in Digital Ocean.


## Configuring SSL

Now, I needed to configure SSL so users can access to the https://catalog.growthandtraction.com subdomain using HTTPS. 

1. First, I created a self-signed SSL certificate and configured the `default-ssl.conf` 
following the steps in this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-ubuntu-14-04). 
2. Then, I activated the SSL virtual host using `sudo a2ensite default-ssl.conf` and restarted Apache to load the new virtual host file using `sudo service apache2 restart`
3. Finally, to avoid getting a warning that the browser cannot verify the identity of my server because it has not been signed by one of the certificate authorities that it trusts, I got a third-party certificate using `Let's Encrypt` (used this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04) as a reference).


## Adjusting OAuth from Google

Finally, I needed to re-configure the OAuth from Google so it works for the new sub-domain. So, I adjusted the **Authorized JavaScript origins** in Google Console APIs so the Google OAuth can support the http://catalog.growthandtraction.com and https://catalog.growthandtraction.com. Also, added http://catalog.growthandtraction.com/login, https://catalog.growthandtraction.com/login, http://catalog.growthandtraction.com/gconnect, and https://catalog.growthandtraction.com/gconnect as **Authorized redirect URIs**. Then, I replaced the old client_secrets.json file in the server with the new client_secrets.json file.


## Third party resources

Used the following tutorials for completing this project:

* [How to deploy a Flask application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [SSL on apache tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-ubuntu-14-04)
* [Let's Encrypt tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04)