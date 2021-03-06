Nginx pre-baked config
======================

_Note: this project was created two years ago and hasn't been updated since. It may be out of date. Use with caution._

For those who, like me, are just getting started with [nginx](http://nginx.org/en/) as an alternative webserver: 
here are some config files that you can use. I've tested them myself, and have gotten them working.
Don't worry: they don't overwrite anything, so if you want, you can just remove them again later.

The idea, taken from [the Wordpress nginx instructions](http://codex.wordpress.org/Nginx), 
is to specify settings for any kind of server package in a separate file, so that when you write a new website config,
you can simply include the config for functionality you need.

How to get started
------------------

Here are instructions for using these config files on Ubuntu.

Before installing nginx, checkout this repo at `/etc/nginx/`.

```bash
sudo git clone https://github.com/RomkeVdMeulen/nginx-config.git /etc/nginx/
```

Then install nginx:

```bash
sudo apt-get install nginx
```

That should do it! Now start adding websites (see below).

Using PHP
---------

Apache has it's own plugin for PHP, but nginx doesn't care how PHP works, just that it can call it.

To get PHP working with nginx, install the PHP5 Fast Process Manager:

```bash
sudo apt-get install php5-fpm
```

By default, the FPM will listen on port 9000 on your server, but I think it's more correct to have it listen on a socket.
To enable this, just edit the FPM default pool settings:

```bash
sudo nano /etc/php5/fpm/pool.d/www.conf
```

Now replace `listen = 127.0.0.1:9000` with `listen = /var/run/php-fpm.sock`. Afterwards, restart the FPM:

```bash
sudo service php5-fpm restart
```

If you can't get this to work, you can settle for using port 9000, but you have to tell nginx about it. 
To do this, edit the PHP config:

```bash
sudo nano /etc/nginx/conf.d/php5.conf
```

Comment the line telling nginx to write to the socket, and uncomment the line telling it to write to the port.

Adding a website
----------------

Add a server config file to `/etc/nginx/sites-available` (e.g. `/etc/nginx/sites-available/com.example.www`). 
To enable the site, link to the config file in `/etc/nginx/sites-enabled`. If these directories don't exist yet,
create them and add this to `/etc/nginx/nginx.conf` (after `include /etc/nginx/conf.d/*.conf`):

```conf
    include /etc/nginx/sites-enabled/*;
```

Use the config files defined in `/etc/nginx/global` to add extra functions to your website. 
But when you do, be sure to read the config file you're including: there may be additional instructions in the comments.
E.g.: when including `global/php-fcgi.conf` you may need to change a setting in `/etc/php5/fpm/php.ini`.

Example site config:
```conf
server {
  # Listen on the default HTTP port (80)
  listen 80;

  # Replace this with your website's URL (minus the http://)
  server_name hostname.com www.hostname.com;

  # Where are the files to use for the website located?
  root /path/to/server/root/;
  
  # Use separated logfiles for every website
  access_log  /var/log/nginx/yourhost_access.log;
  error_log   /var/log/nginx/yourhost_error.log;

  # Now choose which extra features to include:
  
  # This does some basic security and optimization
  include global/restrictions.conf;
  
  # Use this if your website uses PHP
  include global/php-fcgi.conf;
  
  # Use this if your site is a single site wordpress blog
  # Note that you also have to include PHP as this doesn't happen automatically
  include global/wordpress.conf;
}
```

Once you're done, don't forget to have nginx load the new configuration:
```bash
sudo service nginx reload
```

Securing a website with a password
----------------------------------

One of the simplest ways to restrict access to one of your websites is to use basic HTTP authentication.
To do this, you must first generate a htpasswd file containing the usernames and passwords of people who have access to the website.
This file will contain the usernames in plain text and the password encrypted with Crypt.

Using Ruby, we can generate a new htpasswd file like this:
```bash
ruby -le 'print "your-username: #{%Q{your password}.crypt(%Q{salt})}"' > /path/to/your/website/root/.htpasswd
```

Once your htpasswd file is set up, add the necessary config to your website's nginx config file:
```
auth_basic            "This website is restricted!";
auth_basic_user_file  /path/to/your/website/root/.htpasswd;
```
