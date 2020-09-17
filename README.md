# Continuous Delivery for SIGL-2021

## Objective

- Implement continuous delivery for your project ARLAIDE

## Build your continuous delivery pipeline

### Step 1: Create your git repository on github

- Create one github repository for your group, with the following format for the
  name: `arla-group-XX` (e.g. `arla-group-1`, `arla-group-10`...)
- Make it private
- Add your team members to the project: Settings > Manage Access
- Add your teachers to your project:
  - Lucas: LucasBoisserie
  - Florent: ffauchille
- Push your `index.html` file from the previous workshop 

### Step 2: Dockerize your application

- Reuse your index.html file 
- Create a Dockerfile at the root of your repository
  - As a base image, use latest stable image of nginx from docker hub: https://hub.docker.com/_/nginx
  - Copy your index.html to the location of the default index.html file of
    nginx: /usr/share/nginx/html/index.html
- Try to build and run your page on your local host:
  - `docker build -t arla-group-XX .`
  - `docker run -p 8080:80 arla-group-XX`
  - You should see your application running on http://localhost:8080 from your
    browser

### Step 3: Deploy your image to your virtual machine (VM)

We made some changes to your VM. We installed Traefik as a web server / reverse
proxy.
This means, you will NOT need to install any web server on your VM.

> Note: Traefik is a web server that fits perfectly with containerized applications. For
> our use case, we have already installed and configure Traefik on your VM.
> 
> Traefik inspects all running container from the same docker network for docker
> labels. Based on those labels, it will configure for you the reverse proxy to
> route traefik to your application, based on the host name of the incomming HTTP request.
> Further reading here: https://docs.traefik.io/?__hstc=265350736.b8372f9d02e9354a7959f956d9ae00cd.1600363912360.1600363912360.1600363912360.1&__hssc=265350736.1.1600363912360&__hsfp=3215949073

From your local machine:
- Push your docker image using github registry
- Connect to your VM using ssh

From your VM (ssh):
- Pull your docker image using `docker pull` command
- Run your container specifying Traefik labels for your context:
  - "traefik.enable=true"
  - "traefik.docker.network=web"
  - "traefik.frontend.rule=Host:groupe-XX.arla-sigl.fr"
  - "traefik.frontend.port=80"

> Note 2: The version of Traefik installed is 1.7.
> Unfortunately, we are not using version 2 of Traefik since there is quite some
> breaking changes that we didn't yet integrated into this course.

### Step 4: Automate the deployment process using Github actions

This is the fun part of the workshop, where we automate the whole process!

What we want to achieve: 
- on every commit to master branch, I want my application
to be deploy on the VM.

What we know sofar:
- build our application using docker
- publish our application using docker and github registry
- deploy our application using docker and ssh

#### Step 4.1: Create our first Github workflow

Github action is a CD tool that is fully integrated to github.
To integrate Github Actions to your project, you just have to create a new `yml`
file in your project under `.github/workflows/` directory.

Fortunately, Github provides an easy way to create a template file to avoid any
lost of time on typos.

From your Github project page: 
 - Select the `Action` tab > click `set up a workflow yourself`
 - Commit the file clicking the `Commit` green button on master, leaving all
   defaults values

From your local machine: 
- pull changes: `git pull origin master`
- You should see the file `.github/workflows/main.yml`
- Edit the file by changing the first `name: CI` line by `name: CD`
- commit / push changes to master

From your Github project page:
- Select the Action tab
- Make sure your workflow is executing

We just reach our first victory, triggering a pipeline on every commit to
master!

#### Step 4.2: Configuring our workflow

Before we start, make sure you understood the previous step. Read again this
`main.yml` file to understand the structure.

Basically, we have a workflow composed of:
- a name (to identify your workflow in the `Actions` tab on Github), specified
  by the `name: ...` line
- a job trigger (here, on every commit to master), specified by the `on: ...` lines
- `jobs` with a set of `steps`:
  - 1 job is composed of 1 or more steps. A step could be a `shell` command or
    a github action. This is the atomic element of your workflow.
  - 1 job `runs-on` a machine with a specific system. We will use
    `ubuntu-latest`

Let's configure our workflow.

We have 3 steps to deploy:
- build the docker image
- publish the docker image
- run our docker image (on our production VM)

##### Build step

From our local machine, we use `docker build` command.

Fortunately, the `ubuntu-latest` system on which our `steps` are running do have
docker installed and git by default.

But the machine doesn't have our project installed on it.
So we will leave the first step that uses `actions/checkout@v1`. This checks out
the project in this VM machine on ubuntu.

Just add the build step:  

##### Publish step

##### Deploy step



