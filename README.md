# Docker
Learn docker and exercise 

<h2>Contents</h2>

<h3><a href="#intro">Introduction</a></h3>
	<h4><a href="#network">Docker Network</a></h4>
<h3><a href="#images">Images</a></h3>
<h3><a href="#containers">Containers</a></h3>
<h3><a href="#compose">Docker Compose</a></h3>
<h3><a href="#swarm">Docker Swarm</a></h3>
<h3><a href="#k8s">Kubernetes</a></h3>


<h2 id="intro">Introduction</h2>

<b>Why Docker?</b>
<ul>
<li>Developer friendly (develop faster)</li>
<li>Build faster</li>
<li>Test faster</li>
<li>Deploy faster</li>
<li>Update faster</li>
<li>Recover faster</li>
<li>Speed of business</li>
</ul>

Containers reduce complexity => without docker: The "Matrix from Hell" Breeds Complexity

Editions
* Docker CE (Community Edition): free
* Docker EE (Enterprise Edition): paid => docker.com/pricing 
* Note: Docker editions at store.docker.com

Stable vs. Edge
	Edge: beta => it comes out every month => support on old ones discontinue after new one comes out (every month)
	

Install docker on mac 
* Docker's automated script to add their repository and install all dependencies:
		curl -sSL https://get.docker.com/ | sh => latest edge release 
* store.docker.com 
		

Make sure docker is installed and working correctly
	$ docker version
	Note: it also verifies cli can talk to engine

Command: $ docker info
	most config values of engine

Starting from 2017 docker added docker management commands to organize commands
	Old way (still works): docker <command> (options)
	New format with management commands: docker <command> <sub-command> (options)

	Example:
		Old: $ docker run
		New: $ docker container run
	Note: See the full list of management commands and commands:
		$ docker

Image vs. Container
	Image: It is the application we want to run
	Container: it is an instance of that image running as a process
	Note: You can have many containers running off the same image

Note: Docker's default image "registry" is called Docker Hub (hub.docker.com)

Example: 
	$ docker container run --publish 80:80 nginx
Process:
* Downloaded image 'nginx' from Docker Hub
* Started a new container from that image
* Open the port 80 on the host IP
* Routes that traffic to the container IP, port 80

Note: to let the container run in the background => use --detach
	<code>$ docker container run --publish 80:80 --detach nginx</code>
	Note: the container ID is printed

Note: to see the list of running containers (and find the IDs and names)
	<code>$ docker container ls</code>
	or
	<code>$ docker container ps</code>
	or
	<code>$ docker ps</code>

Note: to see a list of all containers ever built and see the IDs and names
	<code>$ docker container ls -a</code>
	or
	<code>$ docker container ps -a</code>
	or
	<code>$ docker ps --all</code>

Note: to specify a custom name for the container: use --name anyName
	<code>$ docker container run --publish 80:80 --detach --name webhost nginx</code>
	Note: like a container id, name is also unique (can log, stop, or remove with container name)

Note: to see the running processes inside a container
	<code>$ docker container top <container-name></code>
	Note: top is the same command like top and htop we use to see the processes of an operating system
	Example:
		<code>$ docker container top webhost</code>

Note: to see a list of all commands you can run on a container:
	<code>$ docker container --help</code>

Remove the containers with status exited
* $ docker container ls -a  => get the first few letters of all containers you want to remove
* $ docker container rm  <id1> (id2) ... 
- Note: you cannot remove a running container with the command above=> command above will through an error for that
- Note: to remove a running container you can stop the container and then remove it or use the following command to force remove a running container
	$ docker container rm -f (id1) (id2) ...
- Note: You can remove using container's name

What does 'docker container run' do?
1. Looks for the image locally in image cache. If finds any go to 4, else go to 2
2. Looks in remote image repository (defaults to Docker Hub). If finds any go to 3, else throw an error
3. Downloads the latest (or specified) version
4. Creates new container based on that image and prepares to start
5. Gives it a virtual IP on a private network inside docker engine
6. Opens up port 80 on host and forwards to port 80 in container (each image has it's own default port. 80 is for nginx)
7. Starts container by using the CMD in the image Dockerfile (will learn about it later)
- Note: take a closer look at the run command:
	$ docker container run --publish 8080:80 --name webhost -d nginx:1.11 nginx -T
    - In the command above you can change:
        * Host listening port: 8080 (should be unique to the container)
        * Container port: 80
        * Version of image: 1.11
        * CMD run on start nginx -T
    - Note: -d is the same as --detach
    - Note: you ca use -p instead of --publish
 
Note: Docker containers are processes on Linux and processes running on mini VMs running Linux on Windows and Mac
Note: From 2017 windows 10 has its own containers running windows 

Exercise:
* Run a nginx, a mysql, and a httpd (apache) server
* Run the all with -d and name them with --name
* nginx should listen to 80:80, httpd on 8080:80, mysql on 3306:3306 
* When running mysql, use the --env option (or -e) to pass in MYSQL_RANDOM_ROOT_PASSWORD = yes (pass an environmental variable)
* Use 'docker container logs' on mysql to find the random password it created on startup => docker container logs mysql 
* Clean it up with docker container stop and docker container rm (both can accept multiple names or ID's)
* Use docker container ls (-a) to make sure everything is cleaned up
- Note: you can use the command $ curl localhost:port to test if the service (container) is working

CLI process monitoring
* List of processes running inside a container
	$ docker container top <container-name-or-id>
	Example: $ docker container top nginx

* How a container is configured when it started and all its metadata config 
	$ docker container inspect <container-name-or-id>
	Example: $ docer container inspect nginx
	Note: the old way: docker inspect <container-name-or-id>
	Note: the command above returns a json with all the data how the container was started

* Check the live status of containers
	$ docker container stats 
	Note: command above will show live stats of all running containers
	$ docker container stats <container-neme-or-id>
	Note: command above shows live status of a specific container

Start a new container interactively
- Use -it 
	Example:	 docker container run -it --name proxy nginx bash
	Note: the command above allows you to run bash commands inside the container
	Note: You can use sh instead of bash
	Note: to exit the interactive mode: $ exit
	Note: the exit command will stop the container
	Note: -it is equivalent to -i -t. -i is for connecting to stdin of the container and -t is for a nicely formatted output (stdout)  

	Example: 
		$ docker container run -it --name ubuntu ubuntu
		Note: the default CMD command for ubuntu image is bash. So you don't need to specify it
	Note: If you install softwares in the ubuntu container and exit, it will keep a new image with all changes
	Note: to start the old exited image in interactive mode:
		docker container start -ai <container-name-or-id>
		Example: docker container start -ai ubuntu


Run additional command in existing container
	The container is running already and we just want to execute some command inside it
	Example: $ docker container exec -it  mysql  bash
			 $ ps aux  => to list the processes (also shows the bash process we are currently sitting in)
	Note: The ps command is no longer included in the mysql image by default
	Note: You can install once you are in the container with: apt-get update && apt-get install -y procps 
	Note: if you exit the bash into the container, it does not stop the root process of the container. It would only stop the bash process to execute commands in the container

Alpine Linux
- A small security-focused distribution
- Alpine is designed to be very small. Only about 5MB in size
* To get the alpine image without running a container of it:
	$ docker pull alpine
* To see a list of images on your local machine:
	$ docker image ls => see the size of the alpine image
* $ docker container run -it alpine bash => error
- alpine is so small that it doesn't even have bash in it
*  $ docker container run -it alpine sh => works
- alpine package manager is apk. You can use apk to install bash if you need

<h3 id="network">Docker Networks</h3>

- Each container connected to a private virtual network "bridge"
- Note: Two containers connected to the same virtual network can communicate with each other by default and we don't need to open ports for their communications
- Example: -p 8080:80 => 8080 is the port in the actual network of the host (router) and 80 is the virtual network port
- Note: on the host level (and each virtual network), you can only use a port to forward to one direction
- Example: 8080:80 and 80:80 for two different containers are valid but 80:80 and 80:8080 is not
* Show virtual networks: $ docker network ls
    - Note: --network bridge => bridge is default docker virtual network which is NAT'ed behind the host IP
    - Note: --network host => it gains performance by skipping virtual networks but sacrifices security of container model
    - Note: --network none => removes eth0 and only leaves you with localhost interface in container
* Inspect a virtual network: $ docker network inspect
    - Note: see the list of container attached to a network (bridge for example): $ docker network inspect bridge
* Create a virtual network: docker network create --driver
    - Example: $ docker network create my_network
    - Note: use 'docker network ls' => check the driver => default driver is bridge
    - Note: network driver: Built-in or 3rd party extensions that give you virtual network features
    - Note: check other options of network creation: $ docker network create --help 
* Attach a virtual network to a container: docker network the connect
    - Attach a network when starting a container. Example:
		$ docker container run -d -p 80:80 --name webhost --network my_network nginx:alpine
    - $ docker network connect => dynamically creates a NIC in a container on an existing virtual network 
    - Example: docker network connect <virtual-network-ID> <container_ID>
    - Note: use 'docker container inspect <container-ID>' to see the networks a container is attached to 
	Note: Use 'docker network inspect my_network' to see attached containers  
* Detach a virtual network from container: docker network disconnect 
    - Docker network disconnect <VN-ID> <container-ID>
    - Note: It dynamically removes a NIC from a container on a specific virtual network
    - Note: Use 'docker container inspect <container-id>' to check the detached network
- Recommendations:
    1. Create your apps so frontend-backend sit on same docker network
    2. Their inter-communication never leaves host
    3. All externally exposed ports closed by default
    4. You must manually expose via -p which is better default security

DNS & Networking
- Forget IP's: Static IP's and using IP's for talking to containers is an anti-pattern. Do your best to avoid it.
- Containers might go away or fail and docker might bring them up somewhere else (with a different IP)
- Docker DNS: docker daemon has a built-in DNS server that containers use by default
- Containers on the same virtual network can talk to each other regardless of their IP addresses with their container names
- Note: Docker defaults the hostname to the container's name, but you can also set aliases
- Example: check if DNS resolution using default container names works:
    * $ docker network create my_network
    * $ docker container -d --name first-nginx nginx:alpine
    * $ docker container -d --name second-nginx nginx:alpine
    * $ docker network my_network inspect => see containers
    * $ docker container first-nginx ping second-nginx => first-nginx listens to the second-nginx
    * $ docker container second-nginx ping first-nginx => second-nginx listens to the first-nginx
    * As we can see we use container's name as DNS instead of its IP addresses
- Note: The bridge network does not have the DNS server built-in to it by default
    - Use --link in the bridge network t manually link containers => $ docker container create -help => link
    - Creating a new network is easier that linking containers in the bridge network
    - Use docker container (will learn about it later)

Assignment
* Use different Linux distributions containers to check curl cli tool version
* Use two different terminal windows to start bash in both centos:7 and ubuntu:14.04, using -it
* Learn the docker container --rm option so you can save cleanup
* Ensure curl is installed and on latest version for that distribution
    * Ubuntu: apt-get update && apt-get install curl
    * Centos: yum update curl
* Check curl --version
- Note: use --rm when running a container to make sure it will be removed once it stops running
- Note: In ubuntu: apt-get install -y curl => -y will install it if not exist already

<h2 id="images">Images</h2>

Official definition: "An image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime"

Image is made of:
* App binaries and dependencies
* Metadata about the image data and how to run the image
    * Get the meta data: $ docker image inspect <image-name-ID>

Inside the image there is not complete OS, no kernel, no kernel modules (e.g. drivers). The host provides the kernel 
- Note: That is one of the differences of containers and virtual machines => with containers we do not booting up a full OS. Its really staring an application

Size of an image could be
* Small as one file (your app binary) like a gaoling static binary
* Big as a Ubuntu bistro with apt, Apache, PHP, and more installed

Ducker Hub
- Signup in hub.docker.com and search for nginx => tons of different nginx images => only one official
- Select the official nginx => check the versions
- Download an image without running off a container from it: $ docker pull nginx => it would download the latest
- Versions: 
    * Equivalent versions (at the time of writing this): 1.19.6  -  mainline  -  1  -  1.19  -  latest => you can use any one of them to pull the latest version
        * Example: $ docker run nginx:latest   or   $ docker run nginx:1.19.6   or $ docker run nginx
    * Note: if you pull one version and pull an equivalent version again, it would use the cached image the second time
    * Alpine: is the simplest and smallest version of Linux => nginx:alpine is the simplest and smallest version of nginx
    * Equivalent versions of nginx alpine: 1.19.6-alpine, mainline-alpine, 1-alpine, 1.11-alpine, alpine
    * Compare nginx versions sizes:
        * $ docker pull nginx
        * $ docker pull nginx: alpine
        * $ docker image ls => 180MB vs 55MB 
    * Note: if you pull two equivalent images of same version and then => $ docker image ls => two images with different tags and same version and size
        - They are in fact two tags pointing to same image (downloaded only once)
- You can get other nginx images instead of official ones. Each might have some custom packages installed on them. 

Image Layers
	$ docker image history <image-name-ID>
* Command above gives us different layers of a specific image
* Each layer is cached and will not be downloaded twice 
* If a container needs some specific layer of an image it will use that layer instead of downloading it again
* More information in Dockerfile

Image Tagging & Pushing
* We normally cannot remember all image IDs
* $ docker image ls => see repository and tag
* Refer to an image: <user>/<repo>:<tag> 
* Default tag is latest
* User is sometimes the organization name
    * Example: search mysql in docker hub => find mysql/mysql-server
* User is usually your docker username
* Latest tag does not actually mean latest version. It's the latest version at the time you pulling an image
* How to retag an image
    * $ docker image tag source_image_tag target_image_tag
        * Example: $ docker image tag nginx:latest hesam/nginx:11.9
    * $ docker image ls => two images (source and target) with the same image ID
    * Tags are just pointers to image IDs
* How to push your custom image into docker hub?
    * $ docker image push <image-tag>
        * Example: $ docker image push hesam/nginx:11.9 => access deny => login to docker
        * $ docker --help => login => $ docker login
    * See the authentication key: 
        * Mac: cd /Users/<username>
        * cat .docker/config.json => auth 
* You can logout from docker with: $ docker logout
* If you create a new tag with the source: hesam/nginx:11.9 and push it:
    * You would see all the layers that exist already
    * It wouldn't upload any layer => just create same tag on docker hub

Private Repository On Docker Hub
* Create a repo and choose visibility to be private

Build A Custom Image
* Command: $ docker build -t tagName .
* The dot at the end means current directory
* The build command builds a new image based on the instruction given in the 'Dockerfile' in the current directory 
*  The default docker file that build expects is 'Dockerfile'. In case of using other filename, it should be specified with -f option: 
    * Example: $ docker build -f NodeDockerfile -t tagName .
* Dockerfile Instructions:
    * FROM: required => example: FROM debian:jessie
        * 'FROM' is used to mention the base image we use to build our own custom image
        * If you want to start with an empty container, use 'FROM scratch'
    * RUN => optional command that we want to run at shel inside container at build time
        * Example: RUN npm install && npm  cache clean --force
        * Note: the && will run the command at the right if the command at the left runs successfully
        * Example: RUN npm install; npm cache clean --force
        * Note: the ; will run command at the right after the one at the left regardless of its successful of failure run
    * EXPOSE: optional  => example: EXPOSE 80 443 3000
        * 'EXPOSE' opens the internal (VN) ports
    * Note: To forward request and error logs to docker log collector:
        * RUN ln -sf /dev/stdout /var/log/nginx/access.log \
			&& ln -sf /dev/stderr /var/log/nginx/error.log
    * CMD: Required: it will run this command when container is launched.
        * CMD would usually goes at the bottom of the Dockerfile
        * Only one CMD instruction should be used. If multiple, the last one wins.
    * WORKDIR: optional => example: WORKDIR /usr/src/app
        * The default directory inside the image or container where we copy files into
    * COPY: optional => example: COPY package.json package.json  
            * Note: Copy package.json file from current directory to the WORKDIR of the image 
        * Example: COPY . .  => copy all everything from current dir to working dir of the image. 
    * ENV: optional => example: ENV NGINX_VERSION 1.11.10-1-jessie
        * Optional environmental variable
        * Example: ENV MYSQL_RANDOM_ROOT_PASSWORD yes
        * Note: the main way we set keys and values for container building and for running containers 
*  Important note: ordering the lines in the Dockerfile
    * Keep the things that change less at the top of the Dockerfile
    * Keep the things that change more at the bottom. Of the Dockerfile

 















