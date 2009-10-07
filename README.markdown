rails-nginx-passenger-ubuntu-postgres
============================

My notes on setting up a simple production server with ubuntu 8.04 (hardy), nginx, passenger and postgres for rails.

Check Ubuntu Version with
    cat /etc/issue

Setup ssh public key authentification

    mkdir ~/.ssh
    touch ~/.ssh/authorized_keys
    chmod go-rwx ~/.ssh/authorized_keys

On your local machine:
  ssh-keygen

Copy public key to remote machine:
    cd $HOME/.ssh
    cat identity.pub | ssh remoteuser@remotehost "cat >> .ssh/authorized_keys"

Now you should be able to log in to your server with a simple: 
    ssh remoteuser@remotehost

Aliases
-------
My favourite aliases:

    alias ls="ls -FG"
    alias ll="ls -l"
    alias la="ls -a"
    alias lll="ls -al"
    alias topc="top -o cpu"
    alias psgrep="ps aux | grep"
    alias cdw="cd $WORK_DIR"

    alias gemi="sudo gem install --no-ri --no-rdoc"
    alias gemu="sudo gem uninstall"
    alias gem_search="gem search -r"

Add your aliases to .bash_aliases. 
Edit .bashrc and uncomment the loadig of .bash_aliases

Update and upgrade the system
-------------------------------

    sudo apt-get update
    sudo apt-get upgrade

Verify that you have to correct date and time with

    date

Configure hostname
-------------------

    sudo hostname your-hostname

Add 127.0.0.1 your-hostname

    sudo vim /etc/hosts
    
Write your-hostname in 
    
    (sudo) vim /etc/hostname
    
Verify that hostname is set
    
    hostname

Install postgres https://help.ubuntu.com/community/PostgreSQL#Hardy
----------------

    (sudo) apt-get install postgresql postgresql-client postgresql-contrib postgresql-server-dev-8.3

Setup Database
---------------    
    (sudo) -u postgres psql postgres

Set a password for the "postgres" database role using the command:

    \password postgres

Create database

     sudo -u postgres createdb <mydb>
     
Alternative Server Setup with 'ident sameuser' authentication
==============================================================

     sudo -u postgres createuser --superuser $USER

     createdb $USER
     Connecting to your own database to try out some SQL should now be as easy as:

     psql
     Creating additional database is just as easy, so for example, after running this:

     createdb amarokdb    
    
Gemrc
-------

Add the following lines to ~/.gemrc, this will speed up gem installation and prevent rdoc and ri from being generated, this is not nessesary in the production environment.

    ---
    :sources:
    - http://gems.rubyforge.org
    - http://gems.github.com
    gem: --no-ri --no-rdoc


Ruby Enterprise Edition
------------------------

Install package required by ruby enterprise, C compiler, Zlib development headers, OpenSSL development headers, GNU Readline development headers

    sudo apt-get install build-essential zlib1g-dev libssl-dev libreadline5-dev

Download and install Ruby Enterprise Edition
=============================================
Ruby Enterprise Edition provides Debian install packages.
Check for a current version at http://www.rubyenterpriseedition.com/download.html

Check out your System Architecture. There are Packages for 64 and 32 bit Ubuntu Systems.
    getconf LONG_BIT
     
    wget http://rubyforge.org/frs/download.php/64478/ruby-enterprise_1.8.7-20090928_amd64.deb    
    dpkg -i ruby-enterprise_1.8.7-20090928_amd64.deb
    
    
Verify the ruby installation

    ruby -v
    ruby 1.8.7 (2009-06-12 patchlevel 174) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 20090928


Installing git
----------------

    sudo apt-get install git-core

Nginx
-------

    passenger-install-nginx-module

Select option 1. Yes: download, compile and install Nginx for me. (recommended)

When finished, verify nginx source code is located under /tmp

    ll /tmp/
    drwxr-xr-x 8 deploy deploy    4096 2009-04-18 17:48 nginx-0.6.36
    -rw-r--r-- 1 root   root    528425 2009-04-02 08:49 nginx-0.6.36.tar.gz
    drwxrwxrwx 7   1169   1169    4096 2009-04-18 17:56 pcre-7.8
    -rw-r--r-- 1 root   root   1168513 2009-04-18 17:51 pcre-7.8.tar.gz
    
Run the passenger-install-nginx-module once more if you want to add --with-http_ssl_module 

    passenger-install-nginx-module
    
Select option 2. No: I want to customize my Nginx installation. (for advanced users)

When installation script ask, "Where is your Nginx source code located?" Enter:

    /tmp/nginx-0.6.36

On, extra arguments to pass to configure script add

     --with-http_ssl_module
     
     
Nginx init script
-------------------

More information on http://wiki.nginx.org/Nginx-init-ubuntu

    cd
    git clone git://github.com/jnstq/rails-nginx-passenger-ubuntu.git
    sudo mv rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/nginx
    sudo chown root:root /etc/init.d/nginx
    
Verify that you can start and stop nginx with init script

    sudo /etc/init.d/nginx start
    
      * Starting Nginx Server...
      ...done.
    
    sudo /etc/init.d/nginx status
    
      nginx found running with processes:  11511 11510
    
    sudo /etc/init.d/nginx stop
    
      * Stopping Nginx Server...
      ...done.
    
    sudo /usr/sbin/update-rc.d -f nginx defaults
    
If you want, reboot and see so the webserver is starting as it should.

Installing the Ruby Postgres Adapter
-------------------------------------
 sudo gem install pg

Installning ImageMagick and RMagick
-----------------------------------

If you want to install the latest version of ImageMagick. I used MiniMagick that shell-out to the mogrify command, worked really well for me.

    # If you already installed imagemagick from apt-get
    sudo apt-get remove imagemagick

    sudo apt-get install libperl-dev gcc libjpeg62-dev libbz2-dev libtiff4-dev libwmf-dev libz-dev libpng12-dev libx11-dev libxt-dev libxext-dev libxml2-dev libfreetype6-dev liblcms1-dev libexif-dev perl libjasper-dev libltdl3-dev graphviz gs-gpl pkg-config

Use wget to grab the source from ImageMagick.org.

Once the source is downloaded, uncompress it:


    tar xvfz ImageMagick.tar.gz


Now configure and make:

    cd ImageMagick-6.5.0-0
    ./configure
    make
    sudo make install

To avoid an error such as:

convert: error while loading shared libraries: libMagickCore.so.2: cannot open shared object file: No such file or directory

    sudo ldconfig

Install RMagick
 
    sudo /opt/ruby-enterprise-1.8.6-20090421/bin/ruby /opt/ruby-enterprise-1.8.6-20090421/bin/gem install rmagick

Test a rails applicaton with nginx
----------------------------------

    rails -d postgresql testapp  (leave -d blank for sqlite)
    cd testapp
    
Enter your postgres password
    
    vim database.yml
    rake db:create:all
    ruby script/generate scaffold post title:string body:text
    rake db:migrate RAILS_ENV=production
    
Check so the rails app start as normal
    
    ruby script/server -e production

Copy the file "nginx" from this project to /etc/init.d (or get a new one from http://wiki.nginx.org/Nginx-init-ubuntu)
    /usr/sbin/update-rc.d -f nginx defaults
    sudo vim /opt/nginx/conf/nginx.conf
    
Add a new virutal host

    server {
        listen 80;
        # server_name www.mycook.com;
        root /home/deploy/testapp/public;
        passenger_enabled on;
    }
    
Restart nginx

    sudo /etc/init.d/nginx restart
    
Check you ipadress and see if you can acess the rails application
        













