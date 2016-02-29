---
layout: post
title: Let's do development into Docker!
---

## Docker with Rails - right development environment
***
Working as a team, you are constantly faced with the problem of support the project infrastructure on developers mashines. When there is a small team (2-4 people), this problem is solved quickly.    

The question is what to do when the number of developers increases, when the product grows or when there are remote developers? The solution is very simple and tech - to use the Docker for local development.  

With the local development we want: to work quickly and efficiently with the project, with a minimum of time to dev ops, use your favorite code editor (for me it’s VIM), do not worry about installing of additional tools and packages. All these gives us Docker with its simplicity and convenience.

Let’s consider a simple project (`Rails application`), which one we want to develop on the local machine, but do not want to set: `ruby, gems, nginx, postrgesql`.

First we need to set up a `virtual box, docker, docker-composer, docker-machine`:

* `brew install virtualbox`
* `brew install docker`
* `brew install docker-compose`
* `brew install docker-machine`
* `brew install dnsmasq`

The next step is to synchronize the configuration of our virtual machine and a local folder for code:

* `sudo echo  “\"/Users\" 192.168.99.100 -alldirs -mapall=501:20” > /etc/exports`
* `sudo echo “nameserver 192.168.99.100” > /etc/resolver/dev`
* `sudo nfsd restart`

Now let's create the docker machine (call it `lab`):

* `docker-machine create --driver virtualbox --virtualbox-memory 4096 --virtualbox-disk-size 20480 --virtualbox-cpu-count 4 lab`
* `docker-machine start lab`
* `docker-machine env lab`
* `eval $(docker-machine env lab)`
* Copy now your project in the folder `~ /`
* Create `Dockerfile` in the project and `docker-compose.yml`

Example files: [Dockerfile](https://github.com/OpakAlex/rails-docker-nginx-example/blob/master/Dockerfile),
[compose](https://github.com/OpakAlex/rails-docker-nginx-example/blob/master/docker-compose.yml)

Let’s try to start:

* ``` docker-compose build```
* `docker-compose run web bundle install`
* `docker-compose run web rake db:create`
* `docker-compose up`

And the last magic is to open the browser: `testapp.dev` and see our app!

It works virtually, so it is very easy to add support of any service, as well as to migrate from one database to another, the team will not have problems with the installation of local packages and their administration. Make life of your team easier with Docker.

If you have problems running nginx, you need to make a few commands:

* https://github.com/OpakAlex/rails-docker-nginx-example/blob/master/bin/install.rb

Working Example: https://github.com/OpakAlex/rails-docker-nginx-example
