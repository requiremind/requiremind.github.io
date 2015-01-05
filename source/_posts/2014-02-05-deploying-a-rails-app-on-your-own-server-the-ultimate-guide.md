---
layout: post
title: "Deploying a Rails App on Your Own Server - The Ultimate Guide"
date: 2014-02-05 23:28
comments: true
author: Neil Rosenstech
categories:
- ruby on rails
- ubuntu server
- unicorn
- nginx
- mongodb
- redis
- sidekiq
- ssl
- security
---

Recently I've been giving life to a project of my own and I had to go through some steps to get the Rails app up and 
running with the best possible conditions.

In this tutorial, I'll try to show you how to deploy your Rails app on a *secured* Ubuntu Server 12.04 using *Capistrano*, *Unicorn*, *Nginx*. Part of my 
setup includes *MongoDB* and *Sidekiq* so I figured I would also explain how I configured those two.

<!-- more -->

Here I'll be assuming that you already have a basic understanding of Rails, git and rvm, and that you have a working application (running on your machine) 
that also has a Github or Bitbucket repo (or any other code hosting service).

Here are the tutorials I used myself. Please note that this guide will be highly based on those other tutorials, my primary goal was to have all the needed
information in one unique place.

- [Initial Server Setup with Ubuntu 12.04](https://www.digitalocean.com/community/articles/initial-server-setup-with-ubuntu-12-04)
- [How to Secure Ubuntu 12.04 Server](http://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-1-basics)
- [How to Add Swap on Ubuntu 12.04](https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04)
- [Deploying Rails app using Nginx, Unicorn and Capistrano](https://coderwall.com/p/yz8cha)
- [Setting up HTTPS with Nginx and StartSSL](http://www.westphahl.net/blog/2012/01/03/setting-up-https-with-nginx-and-startssl/)

First of all you need to have a private server on which you have a ssh access. If you do not have one yet, I suggest taking a VPS at
[Digital Ocean](https://www.digitalocean.com/), they have SSDs in every server and you can have one in just under one minute, starting from 5$/month.

*A lot of the following commands need `sudo` to work.*

You can start by updating packages with `apt-get update`.

Annnnnnddd that's a go!

## Setting up some basic security rules

### Users management

First, change the **root password**:

    passwd

Then you can add a new user so that after we can allow this new user to log in via **ssh** and **disallow root login**.

    adduser johnsnow

We now have to give some permissions to this new user. Edit the corresponding file with:

    visudo

And add this line:

    johnsnow ALL=(ALL:ALL) ALL

### SSH hardening

Edit the related configuration file. I always use `nano` but you can use `vi` or any other editor of your choice.

    nano /etc/ssh/sshd_config

Add or change these lines:

    Port xxxxx
    Protocol 2
    PermitRootLogin no
    UseDNS no
    AllowUsers johnsnow
    DebianBanner no

`Port`: you can choose any unused port from 1025 up to 65536, this is the port you'll use to log in via ssh, instead of the standard port 22. 
This makes it harder for someone to try to log in on your server.

`Protocol 2`: tells ssh to use SSHv2 instead of SSHv1, see [SSHv1 vs SSHv2](http://www.snailbook.com/faq/ssh-1-vs-2.auto.html) for a list of the differences.

`PermitRootLogin no`: disables the login via ssh using **root**.

`UseDNS no`: this option is probably the least important one, check it out here 
[What is the Point of sshd UseDNS Option](http://unix.stackexchange.com/questions/56941/what-is-the-point-of-sshd-usedns-option).

`AllowUsers johnsnow`: makes **johnsnow** the only user allowed to log in via ssh.

`DebianBanner no`: prevents ssh from broadcasting the distribution information, more information here 
[Disable Debian Banner Suffix on ssh Server](https://scottlinux.com/2011/06/14/disable-debian-banner-suffix-on-ssh-server/).

Then you just have to `reload` ssh:

    reload ssh

That's it, you're done, you can no longer log in using **root**. Now, you can do it using your user like so:

    ssh johnsnow@your_server_ip_address -p xxxxx

`xxxxx` being the port you chose earlier in the configuration.

### Firewall setup

Next we are going to set up a firewall that's called UFW as in **Uncomplicated Firewall**.

    apt-get install ufw

And we are going to allow the port we use for **ssh**, **http** and **https** protocols.

    ufw allow xxxxx
    ufw allow http
    ufw allow https

`xxxxx` still being our ssh port.

Now we just have to enable the firewall with:

    ufw allow enable

Now open a second shell and try logging in again via ssh to be sure you didn't break anything with the firewall rules.

You can check everything with:

    ufw status verbose

### Shared memory

To quote the tutorial I followed: *"shared memory can be used in an attack against a running service"*. To secure it, just edit the **fstab** file.

    nano /etc/fstab

And add this line:

    tmpfs     /dev/shm     tmpfs     defaults,noexec,nosuid     0     0

This works on Ubuntu 12.04, for later versions, replace `/dev/shm` by `/run/shm`. Just save and reboot.

### Network security

Edit the **sysctl** file:

    nano /etc/sysctl.conf

And add or change the following lines:

    # IP Spoofing protection
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.conf.default.rp_filter = 1

    # Ignore ICMP broadcast requests
    net.ipv4.icmp_echo_ignore_broadcasts = 1

    # Disable source packet routing
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv6.conf.all.accept_source_route = 0 
    net.ipv4.conf.default.accept_source_route = 0
    net.ipv6.conf.default.accept_source_route = 0

    # Ignore send redirects
    net.ipv4.conf.all.send_redirects = 0
    net.ipv4.conf.default.send_redirects = 0

    # Block SYN attacks
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_max_syn_backlog = 2048
    net.ipv4.tcp_synack_retries = 2
    net.ipv4.tcp_syn_retries = 5

    # Log Martians
    net.ipv4.conf.all.log_martians = 1
    net.ipv4.icmp_ignore_bogus_error_responses = 1

    # Ignore ICMP redirects
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv6.conf.all.accept_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0 
    net.ipv6.conf.default.accept_redirects = 0

    # Ignore Directed pings
    net.ipv4.icmp_echo_ignore_all = 1

To reload the configuration, just issue:

    sysctl -p

### IP Spoofing

To prevent this, edit the **host** file:

    nano /etc/host.conf

And add these lines:

    order bind,hosts
    nospoof on

### DenyHosts

To quote the tutorial I followed once again: *"denyHosts is a python program that automatically blocks SSH attacks by adding entries to /etc/hosts.deny. 
DenyHosts will also inform Linux administrators about offending hosts, attacked users and suspicious logins"*.

Install the program:

    apt-get install denyhosts

If you have to edit email settings or other options, you can edit the following file:

    nano /etc/denyhosts.conf

### Fail2Ban

Fail2Ban listens for failed attemps of connections, exploits etc.. and blocks corresponding IP addresses by updating the firewall rules dynamically.  
To install it, just use the usual:

    apt-get install fail2ban

Edit the configuration file and change the ssh port to the one you are using in the "ssh" section of the file.

    nano /etc/fail2ban/jail.conf

Change `port = ssh` to `port = xxxxx`.

And finally, restart Fail2Ban with:

    service fail2ban restart

### Rootkits

To finish with security we are going to install programs that check the server for rootkits. A rootkit is a software that has been built to hide processes or
programs and grant them privileged access.

Install two programs:

    apt-get install rkhunter chkrootkit

Update and run the first one:

    rkhunter --update
    rkhunter --propupd
    rkhunter --check

Just run the second one:

    chkrootkit

### AppArmor

I'll let Wikipedia describe what it does: [http://en.wikipedia.org/wiki/AppArmor](http://en.wikipedia.org/wiki/AppArmor).

    apt-get install apparmor apparmor-profiles

## SWAP

If you have a limited RAM, say 512M, you might want to add some swap just in case.  
First check that you do not have swap already in use:

    swapon -s

If this results in an empty list, then you're good.  
You can now create the swapfile and setup a swap area with it:

    dd if=/dev/zero of=/swapfile bs=1024 count=512k
    mkswap /swapfile

Now you can enable the file:

    swapon /swapfile

Run the first command to see what changed:

    swapon -s

You should see a new line in the list.  
Now we have to edit **/etc/fstab** file so that these changes still work after a reboot:

    nano /etc/fstab

And add this line:

    /swapfile       none    swap    sw      0       0

Now we have to choose when the system should use this swap space instead of using RAM. For this we have to choose a value between 0 and 100 that's going to be the
left percentage of RAM before using swap. For instance, if we have 512M RAM and we set the value to 10, it means the system will start using swap when we have
less than 10% of RAM available, or if you prefere less than 50M available. Here we set the value to **0**, this means the system will swap only in case of emergency
at the last moment. I personnaly had to set it to **20** for the assets precompilation to work, otherwise I was running into an error (the only indication was 
"*killed*").

    echo 0 | sudo tee /proc/sys/vm/swappiness
    echo vm.swappiness = 0 | sudo tee -a /etc/sysctl.conf

Just set permissions on the file and you're done:

    sudo chown root:root /swapfile 
    sudo chmod 0600 /swapfile

## Nginx and Unicorn

### RVM, MongoDB, Redis and other stuff

Before actually installing and configuring Nginx and Unicorn, we're going to install **rvm**, **git**, and **nodejs**.  
First, let's install **curl**:

    apt-get install curl

And **rvm**:

    curl -L get.rvm.io | bash -s stable

Then make it available in your current shell:

    source ~/.rvm/scripts/rvm

And install the requirements:

    rvm requirements

Install the ruby version your are using in your application, for instance 2.0.0, and make it the default for the system:

    rvm install 2.0.0
    rvm use 2.0.0 --default

Update **rubygems** just in case:

    rvm rubygems current

Install git:

    apt-get install git-core

I usually install **NodeJS** for assets precompilation:

    apt-get install nodejs

Let's install **MongoDB**:

    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
    echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list
    apt-get update
    apt-get install mongodb-10gen

We have to create the default **/data/db** directory and set the owner and permissions accordingly:

    mkdir -p /data/db
    chmod 0755 /data/db
    chown mongodb /data/db

Let's install **Redis** now (for sidekiq):

    apt-get install redis-server

And we're going to need **bundler** too:

    gem install bundler

Finally, at some point, chances are that you're going to install some gem requiring **Nokogiri** which requires additional packages, so let's install those
packages beforehand:

    apt-get install libxslt-dev libxml2-dev

### Nginx

It's now time to install **Nginx** and run it:

    apt-get install nginx
    service nginx start

Now if you navigate to your_server_ip_address in a browser, you should see the default Nginx page.

Let's configure it. Add this in **/etc/nginx/nginx.conf**, in the **http { ... }** block:

    upstream unicorn {
      server unix:/tmp/unicorn.your_project.sock fail_timeout=0;
    }

    server {
      listen 80 default_server deferred;
      # server_name example.com;
      root /home/johnsnow/apps/your_project/current/public;

      location ^~ /assets/ {
        gzip_static on;
        expires max;
        add_header Cache-Control public;
      }

      try_files $uri/index.html $uri @unicorn;
      location @unicorn {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://unicorn;
      }

      error_page 500 502 503 504 /500.html;
      client_max_body_size 20M;
      keepalive_timeout 10;
    }

Let's break it down:  

The `upstream` block will be used to proxy requests to a Unicorn unix socket (a socket used by Unicorn to listen and process requests). Right now there is no
socket yet but we'll be configuring this later on. Then we use this in the `server` block with the line `proxy_pass http://unicorn`. 

The `root` line is used to point to the **public** directory of the Rails app. This path structure is the one set up by **Capistrano** when deploying the
application.

The `client_max_body_size` directive is used to set the maximum body size of client requests. Check it out here: 
[http://wiki.nginx.org/HttpCoreModule#client_max_body_size](http://wiki.nginx.org/HttpCoreModule#client_max_body_size).

You can look the internet up for the rest of these options, you'll find way more information than what I could write here.

### Unicorn

Add the **unicorn** gem to your project: in the Gemfile, add

``` ruby Gemfile
# I am currently using version 4.8.0
gem 'unicorn', '~> 4.8.0'
```

We're going to create the configuration file for **Unicorn**: in your application, create a file **config/unicorn.rb**:

``` ruby config/unicorn.rb
root = "/home/johnsnow/apps/your_project/current"
working_directory root
pid "#{root}/tmp/pids/unicorn.pid"
stderr_path "#{root}/log/unicorn.log"
stdout_path "#{root}/log/unicorn.log"

listen "/tmp/unicorn.your_project.sock"
worker_processes 2
timeout 30

# Force the bundler gemfile environment variable to
# reference the capistrano "current" symlink
before_exec do |_|
  ENV["BUNDLE_GEMFILE"] = File.join(root, 'Gemfile')
end
```

And an initialization script for **Unicorn** as well in **config/unicorn_init.sh**, this file will be symlinked later in **/etc/init.d/**:

``` bash config/unicorn_init.sh
#!/bin/sh
### BEGIN INIT INFO
# Provides:          unicorn
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Manage unicorn server
# Description:       Start, stop, restart unicorn server for a specific application.
### END INIT INFO
set -e

# Feel free to change any of the following variables for your app:
TIMEOUT=${TIMEOUT-60}
APP_ROOT=/home/johnsnow/apps/your_project/current
PID=$APP_ROOT/tmp/pids/unicorn.pid
CMD="cd $APP_ROOT; bundle exec unicorn -D -c $APP_ROOT/config/unicorn.rb -E production"
AS_USER=johnsnow
set -u

OLD_PIN="$PID.oldbin"

sig () {
  test -s "$PID" && kill -$1 `cat $PID`
}

oldsig () {
  test -s $OLD_PIN && kill -$1 `cat $OLD_PIN`
}

run () {
  if [ "$(id -un)" = "$AS_USER" ]; then
    eval $1
  else
    su -c "$1" - $AS_USER
  fi
}

case "$1" in
start)
  sig 0 && echo >&2 "Already running" && exit 0
  run "$CMD"
  ;;
stop)
  sig QUIT && exit 0
  echo >&2 "Not running"
  ;;
force-stop)
  sig TERM && exit 0
  echo >&2 "Not running"
  ;;
restart|reload)
  sig HUP && echo reloaded OK && exit 0
  echo >&2 "Couldn't reload, starting '$CMD' instead"
  run "$CMD"
  ;;
upgrade)
  if sig USR2 && sleep 2 && sig 0 && oldsig QUIT
  then
    n=$TIMEOUT
    while test -s $OLD_PIN && test $n -ge 0
    do
      printf '.' && sleep 1 && n=$(( $n - 1 ))
    done
    echo

    if test $n -lt 0 && test -s $OLD_PIN
    then
      echo >&2 "$OLD_PIN still exists after $TIMEOUT seconds"
      exit 1
    fi
    exit 0
  fi
  echo >&2 "Couldn't upgrade, starting '$CMD' instead"
  run "$CMD"
  ;;
reopen-logs)
  sig USR1
  ;;
*)
  echo >&2 "Usage: $0 <start|stop|restart|upgrade|force-stop|reopen-logs>"
  exit 1
  ;;
esac
```

The file is pretty much straightforward so I suggest you read it and see what it does. Basically, it is used to manage the unicorn process.

### Capistrano

Add the required gems to your Gemfile:

``` ruby Gemfile
gem 'capistrano', '~> 2.15.4'
gem 'rvm-capistrano', '~> 1.5.1'
```

Now we create two files for Capistrano: **Capfile** and **config/deploy.rb** with the following command ran from the root directory of your app:

    capify .

As a side note, before we go further, I'm supposing that you already have the mongoid gem or whatever ORM you're using and the sidekiq gem if you are using 
sidekiq as well.

Let's write some instructions in the **config/deploy.rb** file:

``` ruby config/deploy.rb
require "bundler/capistrano"
require "rvm/capistrano"
require 'sidekiq/capistrano'

server "your_server_ip_address", :web, :app, :db, primary: true

set :application, "your_project"
set :user, "johnsnow"
set :port, xxxxx #your ssh port
set :deploy_to, "/home/#{user}/apps/#{application}"
set :deploy_via, :remote_cache
set :use_sudo, false

set :scm, "git"
set :repository, "your_project.git" #your application repo (for instance git@github.com:user/application.git)
set :branch, "master"


default_run_options[:pty] = true
ssh_options[:forward_agent] = true

after "deploy", "deploy:cleanup" # keep only the last 5 releases

namespace :deploy do
  %w[start stop restart].each do |command|
    desc "#{command} unicorn server"
    task command, roles: :app, except: { no_release: true } do
      run "/etc/init.d/unicorn_#{application} #{command}"
    end
  end

  task :setup_config, roles: :app do
    # symlink the unicorn init file in /etc/init.d/
    sudo "ln -nfs #{current_path}/config/unicorn_init.sh /etc/init.d/unicorn_#{application}"
    # create a shared directory to keep files that are not in git and that are used for the application
    run "mkdir -p #{shared_path}/config"
    # if you're using mongoid, create a mongoid.template.yml file and fill it with your production configuration
    # and add your mongoid.yml file to .gitignore
    put File.read("config/mongoid.template.yml"), "#{shared_path}/config/mongoid.yml"
    puts "Now edit the config files in #{shared_path}."
  end
  after "deploy:setup", "deploy:setup_config"

  task :symlink_config, roles: :app do
    # symlink the shared mongoid config file in the current release
    run "ln -nfs #{shared_path}/config/mongoid.yml #{release_path}/config/mongoid.yml"
  end
  after "deploy:finalize_update", "deploy:symlink_config"

  desc "Create MongoDB indexes"
  task :mongoid_indexes do
    run "cd #{current_path} && RAILS_ENV=production bundle exec rake db:mongoid:create_indexes", once: true
  end
  after "deploy:update", "deploy:mongoid_indexes"

  desc "Make sure local git is in sync with remote."
  task :check_revision, roles: :web do
    unless `git rev-parse HEAD` == `git rev-parse origin/master`
      puts "WARNING: HEAD is not the same as origin/master"
      puts "Run `git push` to sync changes."
      exit
    end
  end
  before "deploy", "deploy:check_revision"
end
```

I've added some comments but feel free to read the rest of the file, it is straightforward as well.

Your **Capfile** should look like this:

``` ruby Capfile
load 'deploy'
load 'deploy/assets'
load 'config/deploy'
```

Add your ssh key to the authorized keys on your server:

    # replace the port, the user and your ip address accordingly
    cat ~/.ssh/id_rsa.pub | ssh -p xxxxx johnsnow@your_server_ip_address 'mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys'

Make sure you pushed all your changes to your repository (on Github or Bitbucket for instance).

Now we have to tell capistrano to create the initial directory structure on the server as described in the recipe we just created:

    cap deploy:setup

You can now go to your server and edit the **mongoid.yml** file in **/home/johnsnow/apps/your_project/shared/config/mongoid.yml**.

Then you'll have to run:

    cap deploy:cold

From the documentation, this will deploy the code, run any pending migrations (not used here because it's MongoDB), and then instead of invoking 
`deploy:restart`, it will invoke `deploy:start` to fire up the application servers. Check it out here 
[http://capitate.rubyforge.org/recipes/deploy.html](http://capitate.rubyforge.org/recipes/deploy.html).

Delete the default Nginx server block:

    rm /etc/nginx/sites-enabled/default

You can restart the **nginx** service to make sure the changes are taken into account:

    service nginx restart

Make sure **Unicorn** is restarted when rebooting the server:

    update-rc.d -f unicorn_your_project defaults

Push your changes (locally) one last time and deploy:

    git push
    cap deploy

That's it, your app should be up and running when typing your server's ip address in your browser!

## Sidekiq

We already required sidekiq in the capistrano recipe so everything should be fine but what if we restart the server? Let's create a script to make sure all the
processes properly restart when rebooting. This script is made to work with **Upstart**.

Create a file **/etc/init/sidekiq.conf**:

``` bash /etc/init/sidekiq.conf
# /etc/init/sidekiq.conf - Sidekiq config

# This example config should work with Ubuntu 12.04+.  It
# allows you to manage multiple Sidekiq instances with
# Upstart, Ubuntu's native service management tool.
#
# Save this config as /etc/init/sidekiq.conf then manage sidekiq with:
#   sudo start sidekiq index=0
#   sudo stop sidekiq index=0
#   sudo status sidekiq index=0
#
# or use the service command:
#   sudo service sidekiq {start,stop,restart,status}
#

description "Sidekiq Background Worker"

# no "start on", we don't want to automatically start
stop on (stopping workers or runlevel [06])

# change to match your deployment user
setuid johnsnow
setgid johnsnow

respawn
respawn limit 3 30

# TERM and USR1 are sent by sidekiqctl when stopping sidekiq.  Without declaring these as normal exit codes, it just respawns.
normal exit 0 TERM USR1

# instance $index

script
# this script runs in /bin/sh by default
# respawn as bash so we can source in rbenv
exec /bin/bash <<EOT
  # use syslog for logging
  # exec &> /dev/kmsg

  # pull in system rbenv
  # export HOME=/home/deploy
  # source /etc/profile.d/rbenv.sh

  cd /home/johnsnow/apps/your_project/current
  nohup bundle exec sidekiq -e production -C config/sidekiq.yml -i 0 -P tmp/pids/sidekiq.pid >> log/sidekiq.log 2>&1 &
EOT
end script
```

Here are the options for the line starting with `nohup`:  
`-e`: the environment  
`-C`: a config file eventhough we do not have one  
`-i`: sidekiq process index, from 0 to whatever you want, you'll have to loop and increment the value if you want to create multiple processes  
`-P`: the file holding sidekiq's process id

`nohup` is used to run sidekiq in the background and therefore keeping it alive event after logging off.

I took the line from this file, check it out: 
[https://github.com/mperham/sidekiq/blob/master/lib/sidekiq/capistrano2.rb](https://github.com/mperham/sidekiq/blob/master/lib/sidekiq/capistrano2.rb).

## Setup daily MongoDB backups

You'll often want to setup backups so here we go: just create a directory to store these backups.

    mkdir /home/johnsnow/dumps

Then create a script to actually make a dump:

``` bash /home/johnsnow/mongodump
#!/bin/bash
DUMPPATH=/home/johnsnow/dumps # dumps directory we just created
MONGODBNAME=your_project_production # your MongoDB database name
DAY=`/bin/date +%Y%m%d` # today's datetime

mongodump --db $MONGODBNAME --out $DUMPPATH/mongo_$DAY # run the command we those variables set
cd $DUMPPATH/mongo_$DAY # navigate to the folder where the dump was saved
tar -cvzf "$DUMPPATH/mongo_$DAY.tar" $MONGODBNAME # create an archive out of that dump

cd $DUMPPATH
rm -rf $DUMPPATH/mongo_$DAY # remove the dump because we only keep the archive version
```

Next make it executable:

    chmod +x mongodump

And create a cronjob:

    sudo crontab -e

    # add the next line in the file
    @daily /home/raindal/mongodump

`@daily` means the script will be executed everyday at midnight.

That's it for backups!

## The end

I hope it helps! If I forgot something, be sure to leave a comment below and I'll try to answer as quickly as I can.

Just in case you run in the same problems I did, I had to run another `update-rc.d -f unicorn_your_project defaults` at the end in order to have unicorn restarting 
on server reboot. The second error was hapenning when I was deploying the application with `cap deploy` and the only indication was "*killed*" just in the 
middle of the assets precompilation. To fix this I just had to set up some swap space because I was lacking memory.




