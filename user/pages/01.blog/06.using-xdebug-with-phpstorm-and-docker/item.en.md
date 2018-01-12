---
title: 'Using xdebug with PhpStorm and docker'
media_order: 'pexels-photo-276452.jpeg,New project dialog.png,Select project path dialog.png,Open Project dialog.png,Edit config menu.png,run debug config dialog.png,add new config dialog.png,setup remote debug 01.png,empty servers proof.png,server settings.png,simultaneous connections.png,index php breakpoint.png,enable debug dialog.png,debug proof breakpoint.png'
published: true
date: '08-01-2018 20:42'
taxonomy:
    category:
        - blog
    tag:
        - docker
        - development
        - setup
        - php
        - symfony
        - container
        - phpstorm
        - xdebug
---

This article is related to [this](/blog/docker-based-php-7-symfony-4-development-environment) one and is build on top of the setup described there. You can also use a similar setup for generic xdebug remote debugging with PhpStorm. The main difference is here, that I recommend to SSH remote forward the debug port 9000. I'm sure I will describe this in a later article in more detail.

If you look at our `docker/app/Dockerfile`, you already see that our app container has [Xdebug](https://xdebug.org/) configured and installed, so we can
directly start to configure it under [PhpStorm](https://www.jetbrains.com/phpstorm/).

## 1. Open our brand new project
The first step is to open our project in PhpStorm under `File > New Project from Existing Files`. In the following window select `Sources files are in a local directory, no Web server is yet configured.` and press `Next`.

![new project dialog](New%20project%20dialog.png)

In the next window we select the path where our project is stored and we click on `Finish`.

![select project path dialog](Select%20project%20path%20dialog.png)

In the next dialog you can choose what you want. We open it here in the current window.

![open project dialog](Open%20Project%20dialog.png)

We should now see our opened project and can go on with configuring xdebug.

## 2. Debug configuration
On the top of our main menu we click on `run` and should see this:

![edit config dialog](Edit%20config%20menu.png)

`Edit Configurations` is our next step. We see now this:

![run debug config dialog](run%20debug%20config%20dialog.png)

With the '**+**' symbol in the top of the left corner, we open this menu:

![add new config dialog](add%20new%20config%20dialog.png)

After clicking on `PHP Remote Debug`, we get a next dialog where we for now enter the following marked details. We can find the ide session key in our [app.env.dist](https://github.com/rkl-/docker-skeleton-dev-environment/blob/master/app.env.dist) file.

![setup remote debug](setup%20remote%20debug%2001.png)

As we can see in the lower error message *"**Error:** Server is not selected"*, we can assume that we need to configure a server as next. For doing this, we click on the *"…"* in the server line, where we currently see *"<no server\>"*. In the dialog coming next we see, that we really have no server configured yet. So let's click on the **"+"** in the above left corner again.

In the next screen we enter the following:

![server settings](server%20settings.png)

The name `app_host` is important here, because this is the name we configured also in our [app.env.dist](https://github.com/rkl-/docker-skeleton-dev-environment/blob/master/app.env.dist) file with `PHP_IDE_CONFIG=serverName=app_host`. This setting allows us to easily debug CLI commands from the console of our app container.

The second important thing here is to setup the path mappings, for this we need to enable `Use path mappings (select if the server is remote or symlinks are used)`. We only need to set the mapping of our local `app` folder here. Based on our [docker-compose.yml](https://github.com/rkl-/docker-skeleton-dev-environment/blob/master/docker-compose.yml) file under the app service, this folder is mounted under `/var/www/html` inside our app container.

We are now done with our basic xdebug and server setup on the PhpStorm side.

The only thing I recommend to do, is setting the number of maximum simultaneous debug connection to a higher value (default is 1). The problem is, if we don't do this, we are only able to debug  just one thread at once and we need more than one. For example, if we trigger a CLI command which triggers something else which then establish a new request. A new request means a new thread in the case of PHP. To prevent this, we simply increase this number. For this we open `File > Settings…` and going to `Languages & Frameworks > PHP > Debug`. Let's change now the default `1` to `5` in our example.

![simultaneous connections](simultaneous%20connections.png)

Click on `OK` and we are done here.

## 3. Test our configuration
We can simply test our debug configuration by opening the `app/public/index.php` file and place a breakpoint at line 19 for example.

![index php breakpoint](index%20php%20breakpoint.png)

Ignore the red underlined `??` here, this is because we have currently the wrong PHP version configured.

Now, in our main menu, we click again on `Run` and enable debug listening.

![enable debug dialog](enable%20debug%20dialog.png)

If we open now [http://localhost](http://localhost) we should see that PhpStorm is pausing at our breakpoint.

![debug proof breakpoint](debug%20proof%20breakpoint.png)

Congratulations, our debug environment is working!