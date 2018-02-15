---
layout: post
title: "docker and ec2"
date: 2018-02-13 19:53:07 -0800
comments: true
categories:
---
# A Docker And EC2 Instance Guide

-

**This is a bare bones guide to setting up a project on an EC2 instance**

Things this guide will cover and hopefully answer:

1. What is docker?
2. What is an EC2 Instance?
3. How do I tie them together with my app that I am developing locally?
4. How do I set it up?

-
**Disclaimer:**
This is a naive method. It does not use ECS, load balancing, clusters all that stuff. You can look that up (and its very confusing tutorials which I thoroughly browsed) if you wish.

If you have a two databases and one node app (i.e redis, mongodb, nodejs files) you should end up with 3 ec2 instances. (1 minimum dockerized container or 3 depending on how you do it).

There is an even more naive way to do it which is to just use docker-compose on all 3 and put them in one ec2-instance (if you got docker-compose to work on your local machine you are basically doing the same thing on your ec2 instance).

**I suggest skimming through the guide to get a good understanding of how this works before blindly following it (although I believe I covered most of the essentials). YOU MIGHT HAVE TO GOOGLE SOME STUFF in addition.**

-
#Create an EC2 Instance.

1. Follow this guide
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html
- Sign up for AWS (make an account/pw), create an IAM user, create a key pair.
- **Save your key pair to your local machine. You will use it. (It should be a .pem file) Mine will be called app.pem for reference.**
- MAKE SURE YOU ARE IN THE **CORRECT REGION** YOU WANT TO BE IN
- (select in top-right us-west-1 N. California).
  ![ss](https://puu.sh/znR7s/f6c593d6fd.png)
- You do not have to create a VPC or security group.

2. In you AWS console, click the top left Services button and then EC2 (make sure correct region always)
3. Click Launch Instance.
4. Select type of machine image you want to launch. I suggest one of these.
   They require different commands depending on the one you choose.
	- Amazon Linux AMI 2017.09.1 (HVM), SSD Volume Type
	- Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-07585467
5. Select type of instance you want.
	- t2.micro is free tier eligible
	- Other ones you have to pay more depending on how long they are running for.
	- Bigger ones give more space or are 'stronger' cpus.
6. Select default for everything for simplicity sake
   (you can specifiy and customize as you understand/research more).
7. Step 4 (Add storage), 5 (Add tags) leave default.
8. Step 6 - Configure security group
	- Make sure you have something like this for your security group.
	![ss](https://puu.sh/znQGT/affbd06e61.png)
	- This is super important because you need to have the inbound port 22 allowed
	  so you can access it from the outside world (which is what SSH uses).
	- Another example: You want to access your service from the outside world on
	  port 80. So what you would do is add another inbound rule allowing port 80
	  of a specific ip (or anywhere).
	- The figure below allows access on port 80, 22, 6379, 7474, 3000, 7687 on anywhere.
	![ss](https://puu.sh/znQZ2/8851b026d4.png)
	- Add the ports you need depending on the database your port exposes to.

9. Review and Launch and wait for it to set up.
10. You should see something like this now in Services -> Instances.
  ![ss](https://puu.sh/znRam/44bfb0821a.png)
11. Click it. At the bottom you should see a public IPv4. This is the IP you can access your instance from.
12. We now need to ssh into it.

On your terminal (omit the $ from all your commands you c/p).

Find your pem file (mine is app.pem) and is located on desktop. You can move it to your
~/.ssh folder if you wish.
Do 1 of these.

```
example: $ chmod 400 <PATH TO pem file>

$ chmod 400 ~/desktop/app.pem        <- (if it located in desktop)
$ chmod 400 ~/.ssh/app.pem           <- (if it is located in .ssh)
```

Now we ssh in using this template

```
example: $ ssh -i <PATH TO pem file> <type of machine image>@<public ip of YOUR ec2 instance>

$ ssh -i ~/desktop/app.pem ec2-user@57.213.217.136     <- use this if you created a linux machine image from step 4.
$ ssh -i ~/desktop/app.pem ubuntu@57.213.217.136       <- use this if you created an ubuntu machine image from step 4.

```
#### Success your ec2 instance is now created!


## This part below is optional. Only do this if you DON'T want to use docker. Skip to docker otherwise.

### Installing a Database/application to your ec2 instance.

Now you should have access to your ec2 instance. You can run commands on it and do whatever you wish. The best way to think about your ec2-instance is that it is now your new 'local computer'. You can install any type of database you wish i.e. mongodb, redis, neo4j, cassandra, postgres, mysql, etc. How do I do this?

```
Google this: "How to install < DB NAME > on < MachineImageUsedType > EC2 Instance"
```

i.e. "How to install redis on linux ec2 instance" links me to this [article](
https://medium.com/@andrewcbass/install-redis-v3-2-on-aws-ec2-instance-93259d40a3ce).

Follow the steps! Some of the links will lead you to the official manual to set you up.
Each database has its own specific quirk to installing it on Ubuntu or Linux just like own OSX or Windows. You have to figure that part out.


Note: installation of certain databases (like redis) is super simple.

Installation of certain databases (like cassandra) is really annoying because you have to configure yml files and whatnot. Look at the section below if you want to skip this pain.

**Assuming you decided to not use docker and you installed your specific database.**

Ideally when you are sshed into your EC2 Instance you should be able to run something like if you have your db properly installed.
```
$ redis-cli
$ cypher-shell
$ MYSQL -u root
$ cqlsh
```

**If you properly installed it on your EC2 instance, move onto the exposing port to real world**

Hopefully it worked!

Final note for this section: You can also deploy your regular applications this way too! Say your MVP project. You could git clone your files from the ec2 instance, spin it up, install a package that has node run detached, expose port 3000 (or whatever port you exposed in express), and you now have access to that app from the real world!

-
#Docker

There is an issue with installing databases and applications on an EC2 instance.
For example, installing cassandra on OSX is a pain in the ass. You have to make sure you have the specific Java version (8 with a specific tag)... You have to install all these different dependencies. Some of these dependencies change according to the machine you have. Imagine trying to install it on some new interface (Linux,Ubuntu) you've never dealt with.

**I wish I could do something like a npm install cassandra on this EC2 instance and magically make it for me.**

**Here is where docker comes in.**

You can skip all the previous installation steps in the section above and trying to figure out how to install and get the correct packages on your linux/ubuntu instance and just do this.

```
1. Install docker
2. Pull the docker database image
3. Run the docker image (creating a new container) on a specific port
4. Expose the port via the inbound security group
5. You're done!
```

Docker is an OS virtualization within a computer which provides a layer of abstraction. This means all of the apps you want to build will always build equally across different types of machines. Imagine if you were building an application on OSX, but randomly decided to change operating systems to Windows. Instead of reinstalling everything on Windows, you can just install docker, create a docker image of your app, pull it to your windows, then continue to work off that. It enables **consistency across environments.**

-

# Using Docker

###If you are using a Linux machine image from step 4.

```
YourTerminal-$ ssh -i ~/desktop/app.pem ec2-user@57.213.217.136

You should be sshed into your ec2-instance now (type yes).

ec2-linux-$ sudo yum update                           <- updates linux
ec2-linux-$ sudo yum install -y docker                <- installs docker
ec2-linux-$ sudo service docker start                 <- starts docker
ec2-linux-$ sudo usermod -a -G docker ec2-user        <- no longer need sudo for docker commands
ec2-linux-$ docker info                               <- docker info
```

###If you are using an Ubuntu machine image from step 4.
Commands are slightly different.

Referenced from https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

```
YourTerminal-$ ssh -i ~/desktop/app.pem ubuntu@57.213.217.136
ec2-ubuntu-$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
ec2-ubuntu-$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
ec2-ubuntu-$ sudo apt-get update
ec2-ubuntu-$ apt-cache policy docker-ce
ec2-ubuntu-$ sudo apt-get install -y docker-ce
ec2-ubuntu-$ sudo systemctl status docker
```

Great. We now have docker installed on our ec2 instance. What's next? It's very simple. All we have to do is figure out what image from dockerhub to pull down depending on our database.

Go here -> [dockerhub](https://hub.docker.com/explore/)

You can see all types of pre built official images like node, redis, mongo, mysql, etc.

Let's say we want to spin up a redis container.

```
Make sure docker is running.
ec2-instance-$ docker run --name new-redis-instance -d -p 6379:6379 redis

```
If you have enough memory on your ec2 instance and it can support the database, it should work.

The run command looks for an image name and pulls from dockerhub if one is not stored locally. It uses the image to spin up a container of the application you specified (the image).

The -d flag creates the container in detached mode. Meaning you can just exit out of your ec2 instance and it will still be running. The first port is the one it exposes to the outside, and the second port is the one for the inside world.

The --name flag is to specify the name.

The last 'redis' is just the image name. So if I wanted to use the mongo image I'd use 'mongo' instead. If I wanted to specify a specific version, I could add a tag like so.

```
ec2-instance-$ docker run --name new-redis-instance -d -p 6379:6379 redis:alpine
```
Useful docker commands

```
$ docker ps -a                       <- shows all containers running/stopped
$ docker stop 'container name/id'       <- specify the container you wish to stop.
$ docker rm 'container name/id'         <- remove the container you wish (this destroys data unless you specified a volume)
$ docker images                      <- shows all images you've downloaded locally (cached)
$ docker rmi 'image name/id'         <- deletes the image
$ docker network ls                  <- shows all docker networks
$ docker inspect 'specific network i.e bridge'    <- shows all docker containers on that network (useful for docker-compose)
$ docker exec -it 'container name' sh  <- ssh into docker container shell.

```
To make sure it is working... do this next! If your container is immediately crashing it might be a memory issue (your service isn't capable enough to handle it). But for mongo, redis, a micro should be enough (until you upload millions of data).

```
ec2-instance-$ docker ps -a     <- make sure your docker container is working (for the db!)
ec2-instance-$ docker exec -it 'container name' sh        <- sshs you into the shell.
```
You should now be in your docker container and be able to browse the contents. Essentially you are now inside of a computer inside of a computer... pretty cool!
Try running the cli for your specific database. So, if you created a redis db you should be able to do:

```
docker-container-$ redis-cli
```
And it should work. This means your db inside your docker is up and running and if you ran the specific port, it is exposing it outside to ec2 (via 6379) when you did docker run -p 6379:6379.

The last step is to now expose your 6379 port to the outside world. In otherwords, have other services be able to talk to it. Follow the steps in "Exposing port to the real world for EC2 Instance"

-
# Exposing port to the real world for EC2 Instances.

These next few steps apply to whether you are using docker or no docker (only for AWS).
The next step is to make sure you have the port you can access your db to expose. So if you use redis, the default port it exposes the database to something like 6379 i.e localhost:6379.

So for example if I created an EC2 Instance and I wanted to access the redis db on the ec2 instance from locally (your computer).
```
I should be able to access my EC2 Instance from Public ip: 54.241.150.234:6379 right?
```
![ss](https://puu.sh/znVzL/d79d0c5f0c.png)
Wrong. Remember the security group port thing I was talking about all the way back in step 8 when creating the EC2 Instance. Well now is where you actually specify which port to expose to the real world (to machines like our local ones).

1. Go to Services -> EC2 -> Instances.
2. You should see the screen like the picture above.
3. Click the security groups link.
![ss](https://puu.sh/znVSA/614f86c808.png)
4. Scroll down to the bottom and click the inbound tab -> Edit.
5. So if you want to expose 6379 to the real world, you need to add the port number in, and choose the source. The source is who you want to allow to access it. If you want everyone in the world to access your database, just click anywhere. (This is bad practice!) You only really want to have your database be accessed by the service that interacts with it. In other words, like an EC2 Instance that has the node server... or perhaps your local machine. Once you click save, you should now be able to send requests to PUBLIC IP:DBPORTNUMBER via postman, curls, insomnia. (You'd change the localhost to EC2 instance IP/port on your NodeJS files).

You should now be able to connect from anywhere if you specify the correct ip/port from any client! (If it doesn't it's because you need to edit the yml files for certain databases like cassandra/neo4j)... but for redis/mongo it should be straightforward...

Repeat the steps above for all the databases you need to create if you want to 'split them apart into different instances'

-
# Okay I got my database deployed, but how do I deploy my app?

The best way to do is to create a docker image of your app (your node files).

```
0. Install docker on your computer.
1. Go to your root project directory
2. $ touch dockerfile
3. copy this into the docker file
    FROM node:carbon

    RUN mkdir -p /src/app

    WORKDIR /src/app

    COPY . /src/app

    RUN npm install

    EXPOSE 3000

    CMD [ "npm", "start" ]
4. $ docker build -t nameofproject/tag .       <- make sure you include the . it builds everything inside the current directory using the dockerfile into an image.
5. $ docker images                             <- you should see your image now!
6. Follow this guide for pushing your image to your dockerhub
   https://docs.docker.com/docker-cloud/builds/push-images/
7. Create a new EC2 Instance (do the steps above)
8. Install docker on the ec2 instance
9. Pull your docker image of your app
10. Run the docker container and your app should now be running.
11. Lastly, add the security group to expose whatever port you are using...
12. Your app is now available as a service to other services by accessing the public IP!
    This means if you have an endpoint to "/api/getUsers",  you can have a service
    do fetch('ec2-publicip:3000/api/getUsers'). In turn, your service will call your db
    which is set up to a different ec2 instance (you set up earlier) and if you have data,
    return results!
```
-
# Adding data to my database on docker/ec2
How do I upload my data if I have .csv or .json files locally?
You can ssh them into your instances/containers.

```
local-machine-$ scp -i ~/pathtomypem -r machine-image@public-ipv4:~/dst src
ex: local-machine-$ scp -i ~/.ssh/app.pem -r ec2-user@54.159.147.19:~/. ~/.neo4j/data/import

then ssh into your ec2-instance

ex: local-machine-$ ssh -i ~/.ssh/app.pem ec2-user@54.159.147.19

your data will be wherever you specified the dst path to.

ec2-instance-$ sudo docker cp src/files image:/dist
ex: ec2-instance-$ sudo docker cp Desktop/file da9230faeb38:/

Then import your data via cli or however you did it

```

-

#Conclusion
There's all this stuff about docker-compose I've been reading about and ECS, and load balancing, and auto balancing, and clusters, and task definitions, and whatnot. These are all correct things to do and things to add on if you want to make your application 'dynamic'.

A cluster is a managing system to bundle certain ec2 instances together. You can specify which specific task each service should run. Each specific task would equate to a docker container. Each service would equate to a docker image. This means you can have multiple databases running (multiple tasks) of the same database (one service). If you think about this means you can split up the work your databases do.

If you had two databases, you can have one specifically write, and the other read. You could create a queue that splits up the work each database does. If you have many concurrent users, you don't want them to read from the same database. You can create copies of your database and have certain users go to certain databases to get their info.

The even cooler part of AWS is the load balancing/auto balancing part. Say your ec2 instance dies, you can't have that happen. AWS will just create a new one based on the minimum tasks that need to be running (so no downtime).

The flaw of the tutorial above is that you are hardcoding a specific ec2-instance into your exxpress server. In the event, it crashes... well a new ip is assigned when you create an ec2-instance. There are ways to handle that (elastic ip addresses) or creating subnets via VPC, but I didn't get a chance to explore it too deeply. For the sake of our project, I don't think people will be creating more than 1-3 ec2 instances as well (for financial purposes).

This is a very **NAIVE** but hopefully a good way to shed light on how EC2 and docker work.

LASTLY, I want to say that you can be even more naive!

```
1. Push up your app image to dockerhub
2. Create a docker-compose file of all your services.
3. Make an EC2 Instance
4. Install docker
5. SSH/git clone that file into your EC2 Instance
6. Run docker-compose file.

Now ALL your containers are running on one instance (which resource-wise, very inefficient).

```

