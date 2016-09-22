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
- changed ownership and permissions of the `authorized_keys` file and `.ssh` dir
```sh
$ chown grader ~grader/.ssh/authorized_keys 
$ chmod 644 ~grader/.ssh/authorized_keys 
$ chmod 700 ~grader/.ssh
```
- added user configuration to `sudoers.d`
```sh
$ visudo -f /etc/sudoers.d/grader 
grader ALL=(ALL:ALL) ALL
```
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
   * set `PasswordAuthentication no` to enforce key-based SSH authentication
   
   * set `Port 2200` to change SSH port from default to 2200


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
- login as postgres user, set password
```
sudo -u postgres psql
\password
```
- create user catalog, set password
``` 
CREATE USER catalog WITH PASSWORD 'PASSWORD_HERE';
```

- created database `trivia`
```
CREATE DATABASE trivia OWNER catalog;
```

- gave rights to the catalog user
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

- enabled psw logins (md5) for `catalog` and `posgres` users, confirmed that only local connections are allowed
```
$ nano /etc/postgresql/9.3/main/pg_hba.conf
```

```
# Database administrative login by Unix domain socket
local   all             postgres                                md5
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   trivia          catalog                                 md5
# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5

```


### Git configuration
- ```git config --global user.name USERNAME```
- ```git config --global user.email EMAIL```
- ```git clone https://github.com/daliavi/trivia-catalog-postgres.git```

### Apache configuration
- set apache user `www-data` in `/etc/apache2/envvar`
- removed default config file `000-default.conf` from `/etc/apache2/sites-enabled`

### App configuration 
Web-server has been configured to serve the Item Catalog application as a wsgi app.
- created wsgi file 
```$ nano /var/www/trivia-catalog-app/trivia-catalog-app.wsgi```
- added apache configuration file for the web app 
```$ nano /etc/apache2/sites-available/trivia-catalog-app.conf```
- changed the owner of the app files
```$ chown -R www-data *```
- adjusted read/write permissions for static/avatar directories
(it was probably not needed, but I missed them up while trying to run the app as grader)
- updated Google and Facebook oauth with valid URIs with the new IP and URL
- changed connection from SQLite to PostgreSQL
- renamed `project.py` to `__init__.py`
```$mv project.py __init__.py ```
- changed file paths to the credential files and in the photo upload method in `__init__.py`

```python
dir_path = os.path.dirname(os.path.realpath(__file__))
fb_secret_path = '/'.join([dir_path, 'fb_client_secrets.json'])
g_secret_path = '/'.join([dir_path, 'client_secrets.json'])
... 
app.config['UPLOAD_FOLDER_ABS'] =''.join([dir_path, UPLOAD_FOLDER])
...
file.save(os.path.join(app.config['UPLOAD_FOLDER_ABS'], new_filename))

```




# Resources:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

http://flask.pocoo.org/docs/0.11/deploying/mod_wsgi

https://httpd.apache.org/docs/2.4/vhosts/examples.html

https://www.postgresql.org/docs/9.3/static

http://newcoder.io/scrape/part-3/

http://askubuntu.com

http://stackoverflow.com

http://dba.stackexchange.com/questions/33943/granting-access-to-all-tables-for-a-user

Udacity classes and forums

