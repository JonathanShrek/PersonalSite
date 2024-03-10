+++
title = 'Migrating a Laravel App on a Linux Server'
date = 2024-03-09T20:09:52-05:00
draft = false
+++

![Image](server.jpg)

So you are running a Laravel web application on a Linux VPS and need to migrate it to another Linux VPS. This migration would be fairly trivial if your Laravel application was running in a container of some sort, but ideal circumstances aren't always reality.

In this post I will be explaining the steps that I have personally taken to migrate a Laravel application from our production Linux VPS to a new Ubuntu Linux VPS.

In my example, I will be using NGINX as the web server. If you would like to use another web server feel free to supplement where needed.

## Versions

To begin the migration, start by getting the current versions of PHP, Node, and NPM on the current production server.

It is very important to make sure that the PHP versions match pretty close to the same. Node and NPM can vary slightly, it just depends upon the libraries used within your project and what their requirements are.

Version numbers can be checked by running the following commands:

```bash
node -v
npm -v
php -v
nginx -v
```

You can install these on Ubuntu using the following commands:

```bash
sudo apt update
sudo apt install nodejs phpX.X-fpm -y
```

If you are on Ubuntu 22.04 then you will likely need a newer version of NGINX than what the Ubuntu repositories have by default. This can be achieved by adding the NGINX ppa and then install NGINX from there.

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/nginx
sudo apt update
sudo apt install nginx
```

After these are installed you should ensure that the services are started correctly.

```bash
sudo systemctl status nginx.service
sudo systemctl status phpX.X-fpm.service
```

If the services are not running you can start and enable them with the following commands.

```bash
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
sudo systemctl start phpX.X-fpm.service
sudo systemctl enable phpX.X-fpm.service
```

If the NGINX service will not start and is throwing errors, you likely have something wrong with your nginx.conf, sites-enabled configurations, or sites-available configurations. These files are located in */etc/nginx/* and running the following command allows you to test your configuration files.

```bash
sudo nginx -t
```

## SSH Keys

Take note of any SSH keys on the current production server that will need to be on the new production server.

## Hosts Files

If your production server utilizes the hosts.allow and hosts.deny files (great idea for security), then make note that these will need to be copied over to the new server as well.

## Shell

On the new server, make sure that the default shell is set to the shell of your choice. In this case, ensure that the default shell is set to bash.

If the server you are moving away from also used a bash shell as its default, then take note of the *~/.bashrc* and copy this over to the new server if need be.

If you copied over or created your own *~/.bashrc* then make sure to run the following command in order to have the changes take effect in your shell.

```bash
source ~/.bashrc
```

## Code

Now we can pull in the code for the project and begin to try and build the project. In this example I will use *git* to clone the codebase to the new server.

```bash
git clone {link to project repo}
cd {project name}
composer install
npm install
npm run production
npm prune --production
```

You may experience some errors during the build process due to permissions or versions. Work through the errors to get the site building and running correctly.
