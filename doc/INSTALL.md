Redmine installation
====================

Redmine - project management software
Copyright (C) 2006-2013  Jean-Philippe Lang
http://www.redmine.org/


Requirements
============

* Ruby 1.8.7, 1.9.2, 1.9.3 or 2.0.0
* RubyGems
* Bundler >= 1.0.21

* A database:
  * MySQL (tested with MySQL 5.1)
  * PostgreSQL (tested with PostgreSQL 9.1)
  * SQLite3 (tested with SQLite 3.7)
  * SQLServer (tested with SQLServer 2012)

Optional:
* SCM binaries (e.g. svn, git...), for repository browsing (must be available in PATH)
* ImageMagick (to enable Gantt export to png images)


    apt-get install imagemagick
    apt-get install libmagickwand-dev

Installation
============

## System Users

Create a `redmine` user for Redmine:

    sudo adduser --disabled-login --gecos 'Redmine' redmine


    # We'll install Redmine into home directory of the user "redmine"
    cd /home/redmine

## Clone the Source

    # Clone Redmine repository
    sudo -u redmine -H git clone https://github.com/anezi/redmine.git redmine


    # Go to redmine dir
    cd /home/redmine/redmine


    # Checkout to stable release
    sudo -u redmine -H git checkout 2.3-stable

**Note:**
You can change `2.3-stable` to `master` if you want the *bleeding edge* version, but never install master on a production server!

# Setup Database

Redmine supports the following databases:

* MySQL (preferred)
* PostgreSQL

## MySQL

    # Install the database packages
    sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

    # Pick a database root password (can be anything), type it and press enter
    # Retype the database root password and press enter

    # Secure your installation.
    sudo mysql_secure_installation
    
    # Login to MySQL
    mysql -u root -p

    # Type the database root password

    # Create a user for Redmine
    # do not type the 'mysql>', this is part of the prompt
    # change $password in the command below to a real password you pick
    mysql> CREATE USER 'redmine'@'localhost' IDENTIFIED BY '$password';

    # Create the Redmine production database
    mysql> CREATE DATABASE IF NOT EXISTS `redmine_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

    # Grant the Redmine user necessary permissions on the table.
    mysql> GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `redmine_production`.* TO 'redmine'@'localhost';

    # Quit the database session
    mysql> \q

    # Try connecting to the new database with the new user
    sudo -u redmine -H mysql -u redmine -p -D redmine_production

    # Type the password you replaced $password with earlier

    # You should now see a 'mysql>' prompt

    # Quit the database session
    mysql> \q

    # You are done installing the database and can continue the installation.


## Configure the database parameters

    cd /home/redmine/redmine

    # Copy the example database parameters
    sudo -u redmine -H cp config/database.yml.example config/database.yml

    # for the "production" environment (default database is MySQL and ruby1.9)
    # If you're running Redmine with MySQL and ruby1.8, replace the adapter name
    # with `mysql`
    #
    sudo -u redmine -H vim config/database.yml


## Install the required gems by running:

    sudo -u redmine -H bundle install --without development test

   If ImageMagick is not installed on your system, you should skip the installation
   of the rmagick gem using:
     bundle install --without development test rmagick

   Only the gems that are needed by the adapters you've specified in your database
   configuration file are actually installed (eg. if your config/database.yml
   uses the 'mysql2' adapter, then only the mysql2 gem will be installed). Don't
   forget to re-run `bundle install` when you change config/database.yml for using
   other database adapters.

   If you need to load some gems that are not required by Redmine core (eg. fcgi),
   you can create a file named Gemfile.local at the root of your redmine directory.
   It will be loaded automatically when running `bundle install`.

5. Generate a session store secret
   
   Redmine stores session data in cookies by default, which requires
   a secret to be generated. Under the application main directory run:


    sudo -u redmine -H rake generate_secret_token

6. Create the database structure
   
   Under the application main directory run:


    sudo -u redmine -H rake db:migrate RAILS_ENV="production"
   
   It will create all the tables and an administrator account.

7. Setting up permissions (Windows users have to skip this section)
   
   The user who runs Redmine must have write permission on the following
   subdirectories: files, log, tmp & public/plugin_assets.
   
   Assuming you run Redmine with a user named "redmine":


    sudo -u redmine -H mkdir public/plugin_assets
    
    
     sudo chown -R redmine:redmine files log tmp public/plugin_assets
     sudo chmod -R 755 files log tmp public/plugin_assets

8. Test the installation by running the WEBrick web server
   
   Under the main application directory run:


    sudo -u redmine -H ruby script/rails server -e production
   
   Once WEBrick has started, point your browser to http://localhost:3000/
   You should now see the application welcome page.

9. Use the default administrator account to log in:
   login: admin
   password: admin
   
   Go to "Administration" to load the default configuration data (roles,
   trackers, statuses, workflow) and to adjust the application settings

SMTP server Configuration
=========================

Copy config/configuration.yml.example to config/configuration.yml and
edit this file to adjust your SMTP settings.
Do not forget to restart the application after any change to this file.

Please do not enter your SMTP settings in environment.rb.

References
==========

* http://www.redmine.org/wiki/redmine/RedmineInstall
* http://www.redmine.org/wiki/redmine/EmailConfiguration
* http://www.redmine.org/wiki/redmine/RedmineSettings
* http://www.redmine.org/wiki/redmine/RedmineRepositories
* http://www.redmine.org/wiki/redmine/RedmineReceivingEmails
* http://www.redmine.org/wiki/redmine/RedmineReminderEmails
* http://www.redmine.org/wiki/redmine/RedmineLDAP
