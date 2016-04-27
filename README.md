# Udacity's_FSND_P5_Server

Starting from a ubuntu 14.04 server base installation. Perform necessary
operations to serve FSND-P3 Catalog site.

**Note:** My apologies for the poor soul that has to read through this. I suck at explaining myself, so if anything is unclear(probably lots of things), don't hesitate on sending me an email to `danielcarrilloharris@gmail.com`.

**ServerIP:** http://52.35.218.155
**SSHPort:** 2200

Project Rubric: [Rubric][1]

**Recomendation:** Before doing anything `apt-get update && apt-get dist-upgrade` to have everything up to date.

### Make user grader and give it sudo permission

  1. `adduser grader`
  2. It will ask you for some info just press enter to pass, or input that info and set a password.
  3. `visudo` to edit sudoers file
  4. look for something there like `root ALL=(ALL) ALL` and insert below it `grader ALL=(ALL) ALL`.

### Config ssh to run on port 2200

  1. `nano /etc/ssh/sshd_config`
  2. Look for a line that says something like Port 22 and just change it to Port 2200
  3. Do `service ssh restart`
  4. Check that you can ssh from there adding the argument `-p 2200` when you ssh into the machine.

### Set up the UncomplicatedFirewall (UFW)

  1. `ufw allow 2200`
  2. `ufw allow 80`
  3. `ufw allow 123`
  4. `ufw deny 22`
  5. `ufw enable`
  6. Check everything is correct`ufw status verbose`

### Set machine time to UTC
Source: [askubuntu][2]

  1. `dpkg-reconfigure tzdata`
  2. scroll to the bottom of the Continents list and select Etc; in the second list, select UTC.

### Install python-pip and your app's needed dependencies and packets
Maybe you'll want to install some specific packets inside the virtual environment
later on instead of installing them on the main system.
  1. `apt-get install python-pip`
  2. Install dependencies and packets thru pip or apt-get like `pip install Flask` or `pip install virtualenv`
  **Note:** I needed to install:
    1. python-pip "You'll need it too"
    2. flask "You'll need it too"
    3. sqlalchemy "You'll probably need it too"
    4. oauth2client "Maybe you'll need it"
    5. virtualenv "You'll probably need it too"
    6. libpq-dev "You'll probably need it too" (its from apt-get)
    7. psycopg2 "You'll probably need it too"
    8. plus postgresql, etc... "Well... Its mandatory"

### Install Apache and mod_wsgi

  1. `apt-get install apache2 && apt-get install libapache2-mod-wsgi`
  2. Do `a2enmod wsgi` to enable wsgi
  3. Will configure the rest later when we get repository.

### Install postgreSQL
  1. `apt-get install postgresql`
  2. As apache, will configure later.

### Install git and clone your repository

  1. `apt-get install git`
  2. Go to www directory`cd /var/www/`
  3. Create folder with your repo_name and go inside it.
  4. `git clone https://github.com/{yourusername}/{your_repo}`
  5. As we are here hide your .git from the server `nano .htaccess` and
  inside of it `RedirectMatch 404 /\.git`.

### Make your wsgi file and add an __init__ in your file (if you don't have one already)
Source: [DigitalOcean][3]

  1. Go to `/var/www/{your_repo}/`
  2. Create a file called `{your_repo}.wsgi`
  3. Edit that file with
  ` #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/{your_repo}/")

    from {your_repo} import app as application
    application.secret_key = 'Add your secret key'`
  4. cd into your inner {your_repo} folder `cd {your_repo}`
  5. Make an empty init `touch __init__.py`

### Virtualenv your site
  1. Once Virtualenv is installed and your repo is in place you'll want to virtualenv your site
  2. Go to `/var/www/{your_repo}/{your_repo}`
  3. Do `virtualenv venv` where venv could stay as venv or your repo name or whatever you feel is a right name
  4. Then you can go inside the Virtualenv doing `source venv/bin/activate`
  5. Inside of it you may install things to isolate them from the base system.
  6. Exit it by doing `deactivate`

### Setting up apache the crappy way (how i did it)
'Kind of' Source: [DigitalOcean][3]

  1. Edit `/etc/apache2/sites-available/000-default.conf`
  2. `<VirtualHost *:80>
		    ServerName mywebsite.com
		    ServerAdmin admin@mywebsite.com
		    WSGIScriptAlias / /var/www/{your_repo}/{your_repo}.wsgi
		    <Directory /var/www/{your_repo}/{your_repo}/>
			     Order allow,deny
			      Allow from all
		    </Directory>
		    Alias /static /var/www/{your_repo}/{your_repo}/static
		    <Directory /var/www/{your_repo}/{your_repo}/static/>
			     Order allow,deny
			     Allow from all
		    </Directory>
		    ErrorLog ${APACHE_LOG_DIR}/error.log
		    LogLevel warn
		    CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>`
  3. Restart apache `service apache2 restart`

### Setting up apache nicely
Source: [DigitalOcean][3]
  1. Create a file in `/etc/apache2/sites-available/` called `{yoursite}.conf`.
  2. In that file put:
    `<VirtualHost *:80>
        ServerName mywebsite.com
        ServerAdmin admin@mywebsite.com
        WSGIScriptAlias / /var/www/{your_repo}/{your_repo}.wsgi
        <Directory /var/www/{your_repo}/{your_repo}/>
           Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/{your_repo}/{your_repo}/static
        <Directory /var/www/{your_repo}/{your_repo}/static/>
           Order allow,deny
           Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>`
  3. Do `a2ensite {yoursite}`
  4. Restart apache `service apache2 restart`

### Set up postgreSQL
Source: [Strueken][4]
**Note:** He's much better than me at explaining all this, I should have found him earlier.

  1. Create linux catalog login `adduser catalog`.
  2. Change to user postgres `su - postgres`.
  3. Go into psql and make a user 'catalog' `CREATE USER catalog WITH password your_pass;`.
  4. Allow catalog to make Db's `ALTER USER catalog CREATEDB;`.
  5. Make database owned by catalog `CREATE DATABASE catalog WITH OWNER catalog`.
  6. Connect to db `\c catalog`.
  7. Revoke rights from public `REVOKE ALL ON SCHEMA public FROM public`.
  8. Give rights to catalog `GRANT ALL ON SCHEMA public TO catalog;`.
  9. Exit psql `\q` and exit postgres login `exit`.
  10. Check Known Hassles below if you are currently using a db thats not postgres.



## Known Hassles

### psql verification user postgres fails

  1. Do `nano /etc/postgresql/9.3/main/pg_hba.conf`
  2. Look for user postgres and change Authentication method to "Trust" instead of peer or md5
  3. Try to connect now `psql -U postgres`

### NOT using postgres in my db_model.py YET

  1. You'll have to change your create_engine variable inside db_model.py and in your {your_app}.py
  2. Change it to `create_engine('postgresql://catalog:{your_pass}@localhost/catalog')`.

### Permission hell

  1. If stuff complains about not being reachable or modifiable is probably a permission problem.
  3. Some or all files you may want to be owned by www-data
  3. Check the folders and files with `ls -la`
  4. If needed change permissions or ownership with `chmod 000 file` where 000 is the permission code and file is the name of the file or directory, or `chown user file`where user and file are pretty much self explanatory.
  5. If in doubt there are great articles of 'chmod' and 'chown' in wikipedia.

### I changed something and it's still not working
  1. Maybe you'll have to restart apache `service apache2 restart`.


[1]:https://docs.google.com/document/d/1J0gpbuSlcFa2IQScrTIqI6o3dice-9T7v8EDNjJDfUI/pub?embedded=true "Project Rubric"
[2]:http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt "Change timezone to UTC"
[3]:https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "Flask Apache WSGI"
[4]:https://raw.githubusercontent.com/stueken/FSND-P5_Linux-Server-Configuration/master/README.md "Awesome tutorial to do all this"  
