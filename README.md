# Server Configuration for Recipe Server
This repo features server code of a recipe server. The server is hosted by Amazon Lightsails (part of AWS) at 18.216.83.166 (or with the DNS http://ec2-18-216-83-166.us-east-2.compute.amazonaws.com/). While a repo exist for the local development of this recipe server [here](https://github.com/osa10928/Udacity-recipe_server), this repo details the configuration process needed to attach the recipe server code to a dedicated server. This repo was created as part of Udacity's Fullstack Development course and those interested in the viewing the server themselves can do so by using ssh protocol as 'grader' on port 2200 (with the proper credentials of course).

##Environment
The environment for the server development was provided by Amazon Web Services Amazon Lightsail. Amazon Lightsail provides a standard installation of Ubuntu 16.04 LTS for those looking to use their services.

##Security
As a best practice, the server can only be accessed via ssh on port 2200 and through standard http protocol on port 80. Seperate users were created to complete this project in accordance with Udacity Project instructions. Namely, one may log in as the user 'catalog' or the user 'grader' but only with the proper private keys which have been supplied to relevant parties.

## Specifications
As mentioned before, this application uses server code from [this](https://github.com/osa10928/Udacity-recipe_server) repo and thus requires the same utilities, namely Python3, Flask, and Flask-Sqlalchemy.

`python >= '3.5.2'`
`Flask >= '0.12.2'`
`Flask-SQLAlchemy >= '2.3.1'`

While the original repo uses SQlite as a relational database, this production level project moves to the more advanced database PostgreSQL.

`psql (PostgreSQL) >= '9.5.10'`

Finally, the server is built using Apache2 and the application handler mod-wsgi.

`Apache >= 2.4.18 (Ubuntu)`
`libapache2-mod-wsgi-py3 >= 4.3.0-1.1build1`

## Server Configuration
This repo's goal is to go over the configuration steps necessary to host a python application and complete necessary server request. This readme will detail the steps taken to get this project up and running using an application from another repo. Details about the application that is running on this server can be found on that repo's readme [here](https://github.com/osa10928/Udacity-recipe_server). This readme will only concern itself with steps taken to configure the server.
### Amazon Lightsail
The first step to get this project up and running is registering with Amazon Web Services and logging into Lightsail. One logging in, create an instance of an Ubuntu 16.04 LTS image. At this point you can log into you VM through ssh on the browser. It is recommended that you take these security steps to secure your server:
1. Create another super user.
2. Create key pairs for user.
3. Log in with user via ssh using the generated key pairs from your terminal as opposed to the browser.
4. Prevent ssh access to root user. Prevent ssh access using a password. Never show you ssh private key.
5. Configure fire wall to limit ssh ports to 2200
After you've taken whatever steps needed to secure your server, its time to instasll apache.
### Install Apache2
Run `sudo apt-get install apache2` to install Apache2
### Configure Apache2 with mod-wsgi
This repo uses mod-wsgi under Apache2 to handle request. Run `sudo apt-get install libapache2-mod-wsgi` to install mod-wsgi. Set up wsgi to handle Apache2 request by opening the `/etc/apache2/sites-enabled/000-default.conf`. Configure the document like so:
```
<VirtualHost *:80>
        ServerName [IPAddress]
        ServerAlias [DNS]

        WSGIDaemonProcess recipe_server user=catalog[userToRunProcess] group=catalog[groupToRunProcess] threads=5
        WSGIScriptAlias / /var/www/recipe_server/recipe_server.wsgi(location of wsgi file)

        <Directory /var/www/recipe_server(directory of Application)>
            WSGIProcessGroup recipe_server
            WSGIApplicationGroup %{GLOBAL}
            Require all granted
        </Directory>
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
### Configure WSGI to run Recipe server code
The wsgi file imports the application and passes http request to the application to be processed. Normally a .wsgi file would have to be created manually, but this repo ships with a .wsgi file on board. 

### Place Application code on server
Now its time to actually place the application on the server. Navigate to the /var/www directory by running `cd /var/www`. Clone the repository into this directory. At this point Apache is ready to use WSGI to load you application using the .wsgi file and process request. The application is configured to use postgresql to store data delivered by request. The final step will be to configure psql to execute query directives.

### Configure ProgreSQL
The application uses ProgreSQL as a relational database. A user must be created on the psql server (separate from the users of Ubuntu) in order to execute database directives. Follow these steps to install postgresql and create a user:
1. Run `sudo apt-get install postgresql libpq-dev postgresql-client postgresql-client-common`
2. When postgresql is installed it automatically creates a postgresql super user named postgres. sudo into postgres by running `sudo su postgres`
3. Create user by running `createuser (username)`. Make the username the same as the ubuntu user, in our case, 'catalog'. Give the user only the ability to create databases.
4. Exit out of postgres user. Create db by running `createdb (dbname)`. Make the dbname the same as the ubuntu user, in our case, 'catalog'.


