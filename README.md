# Linux Server and Web App Configuration
## Project Scope
Take a baseline installation of a Linux distribution on a virtual 
machine and prepare it to host your web applications, to include installing updates, 
securing it from a number of attack vectors and installing/configuring web and database servers.

## Server Access
- IP address: `52.11.253.245`
- URL: `http://ec2-52-11-253-245.us-west-2.compute.amazonaws.com`
- ssh login: ssh -i ~/.ssh/udacity_key.rsa grader@52.11.253.245 -p 2200

## Software installed
- Updated Ubuntu sources and upgraded packages:
```sh
$ apt-get update
$ apt-get upgrade
$ apt-get autoremove
```

- Additional software installed (`apt-get install`):
    - apache2
    - libapache2-mod-wsgi
    - git
    - postgresql
    - postgresql-client
    - python-psycopg2
    - python-pip 
    - python-dev
    - htop
- Python related software (`pip install`)
    - virtualenv 
    - Flask (in virtualenv)
    - flask-seasurf (in virtualenv)
    - SQLAlchemy (in virtualenv)
    - oauth2client (upgrade, in virtualenv)


## Server and Web App Configurations 


### User and SSH configuration
- created a user 'grader'
```sh
$ adduser grader
```
- set password
```sh
$ passwd grader
```
- added the public key (in the home dir of the user)
```sh
$ nano .ssh/authorized_keys
```
- changed ownership and permissions of the authorized_keys file and .ssh dir
```sh
$ chown grader ~grader/.ssh/authorized_keys 
$ chmod 644 ~grader/.ssh/authorized_keys 
$ chmod 700 ~grader/.ssh
```
- added user configuration to to sudoers.d 
```sh
$ visudo -f /etc/sudoers.d/grader 
```
 - file content `grader ALL=(ALL:ALL) ALL`

- added the host name to avoid warning message when using `sudo` command
```
$ nano /etc/hostname
ip-10-20-45-81
```

```
$ nano /etc/hosts
127.0.0.1 localhost
127.0.0.1 ip-10-20-45-81
```

- changed ssh configuration
```sh
$ nano /etc/ssh/sshd_config
```

   - set `PasswordAuthentication no` to enforce key-based SSH authentication
   - set `Port 2200` to change SSH port from default to 2200


### Server and Firewall configuration
- confirmed the timezone on the server was UTC
```sh
$ date +%Z
```
- Firewall configuration. Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```sh
$ ufw default deny incoming
$ ufw default allow outgoing
$ ufw allow 2200/tcp
$ ufw allow www
$ ufw allow ntp
$ ufw enable
```

### PostgreSQL configuration
- login as postgres user, set password, enable psw logins in /etc/postgresql/9.3/main/pg_hba.conf
- create user catalog, set password, enable psw logins in /etc/postgresql/9.3/main/pg_hba.conf
- confirmed only local connections are allowed in /etc/postgresql/9.3/main/pg_hba.conf
- create database trivia and give rights to the catalog user
```
REVOKE CONNECT ON DATABASE trivia FROM PUBLIC;
```
```
GRANT ALL PRIVILEGES
ON DATABASE trivia
TO catalog;
```
```
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO catalog;
```

- check if rights were granted:
```
SELECT * FROM information_schema.role_table_grants
WHERE grantee='catalog';
```


### git configuration
- ```git config --global user.name USERNAME```
- ```git config --global user.email EMAIL```
- ```git clone https://github.com/daliavi/trivia-catalog-postgres.git```

### Apache configuration
- set apache user www-data in `/etc/apache2/envvar`
- removed default config file `000-default.conf` from `/etc/apache2/sites-enabled`

### App configuration 
Web-server has been configured to serve the Item Catalog application as a wsgi app.
- created wsgi file `nano /var/www/trivia-catalog-app/trivia-catalog-app.wsgi`
- added apache configuration file for the web app `nano /etc/apache2/sites-available/trivia-catalog-app.conf`
- changed the owner of the app files: chown -R www-data *
- adjusted read/write permissions for static/avatar directories
- updated Google and Facebook oauth with valid URIs with the new IP and URL
- changed connection from SQLite to PostgreSQL
- changed file paths to credential files and in photo upload method
- renamed project.py to __init__.py


# Resources:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
http://flask.pocoo.org/docs/0.11/deploying/mod_wsgi
https://httpd.apache.org/docs/2.4/vhosts/examples.html
https://www.postgresql.org/docs/9.3/static
http://askubuntu.com
http://stackoverflow.com
http://dba.stackexchange.com/questions/33943/granting-access-to-all-tables-for-a-user
Udacity classes and forums

