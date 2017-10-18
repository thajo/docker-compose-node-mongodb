# Node MongoDB Docker

This repo contains a dockerized start-up solution for developing Node.js applications with mongoDB. This guide is written for the courser [1DV023](https://coursepress.lnu.se/kurs/serverbaserad-webbprogrammering/) and [1DV523](https://coursepress.lnu.se/kurs/server-based-web-programming/) at the Linnaeus University, Kalmar, Sweden.

### Disclaimer
This start-up guide is made for development and **not for production**. There may be better ways to write your Dockerfile and docker-compose.yml but feel free to make pull requests.

### The purpose
The purpose of this repo is for the student to quickly get a development environment using Docker. The solution contains a Node.js container and a link to a mongoDB container. The node.js container will have [nodemon](https://github.com/remy/nodemon) installed by default to get the ability to restart the web application on changes.

## The setup
First of all make sure you have installed Docker on your system. You can download and install the Docker Community Edition by visiting the docker site: https://www.docker.com/get-docker

This repository contains a dummy project (a simple express application) and two files that handles the Docker stuff, the Dockerfile and docker-compose.yml

### Dockerfile
This file will create the Node.js container where the application live. It will install the latest version of Node.js and copy your whole application directory over to the container (all files in and under the directory where the Dockerfile is). It will also do a `npm install` and also install the nodemon module globally in the container.

The configuration expects some things of your application.

* Your application should listen on port 8000; you will be able to visit your application from your own browser by surfing to:
http://localhost:8000
* Your application must start in a file "app.js" which must be located in the same folder as the Dockerfile

### docker-compose.yml
This file is responsible for starting up the two images (Node.js and the mongoDB) and linking them together. The Node-container will be created from the Dockerfile above and the mongoDB will be downloaded from [Docker hub](https://hub.docker.com/).

This file will create this two containers as services, one named web and one name db. These name could be used when communicate with the containers. More about that below.

### Make your setup
You will have a repository where you will have your application. You have probably started by running `npm init` and created your app.js.

1. Download the files (Dockerfile and docker-compose.yml) from this repository. You could copy the code in this repository's app.js if you want a start. Make sure to put these files in the same folder as your app.js is.
2. First we must build the Node.js image. This step will mean that Docker will download a base image from the docker hub, run the instructions defined in the Dockerfile. This will just build the image, no container has yet been started.
To do this you open our terminal in the folder containing the files and enter the following command in your terminal:

`docker-compose build`

You will now see that Docker will be pulling down stuff and extracting it and then run the instruction, some of them will take some time and you may get some red warnings (we can ignore them right now). This step we only have to do the first time.

3. We have now built the Node.js-image and will now start our containers/services; both the application/web and the mongoDB/db. To do this enter the following command in your terminal:

`docker-compose up`

If this is the first time you run this command Docker will probably pull down the mongoDB-image from docker hub and it will take some time. Then it will try to start the containers (in the right order) and link them together. It will also start nodemon and run the app.js file. If you using the same code as in this repository the end of output will probably be something like this:

![screenshot after docker-compose up](#)

This means that our application has started and are running. This terminal must always be running as long as you want your application/web server running. To shutdown the services just press `ctrl+c` in this terminal window. To restart the services just type docker-compose up again and they will start, probably much faster this time.

4. You can now start writing your code. The Docker is syncing your changes in your code to the container and nodemon will restart the application when you make changes. You should be able to open an other terminal window and do your commits to your GitHub repository from your local filesystem.

### Installing node modules in the running container
One thing that you have to think about when running this docker environment is that you must install your modules in your container which will demand to do a special command. In a separate terminal window (where the current directory is the where your Docker-files is) you write the following command:

`docker-compose exec web npm install <name of the module>`

This executes the command in the container/service named web (that is our application container). As you probably understand the last part of this command is the common "npm install"-command specifying the module you want to install. So if you want to install mongoose you write like:
`docker-compose exec web npm install mongoose`
When you write this your package.json will be updated which is important since this is file must be updated when committing your code to GitHub.

#### Check the connection to the mongoDB
If you have install the mongoose module as above you can now test your setup to see that your code on our app server can communicate with the database service.

A simple way to do this is to take some connection code from the [mongoose documentation](http://mongoosejs.com/)]:

```
let mongoose = require('mongoose')
mongoose.connect('mongodb://localhost/test', { useMongoClient: true })
mongoose.Promise = global.Promise

var Cat = mongoose.model('Cat', { name: String })

var kitty = new Cat({ name: 'Zildjian' })
kitty.save(function (err) {
  if (err) {
    console.log(err)
  } else {
    console.log('meow')
  }
})
```
This code won´t work out of the box. If we trying to connect to the mongodb://localhost/ there will be an error since this code runs in a container where we don't have the mongoDB installed. We must connect to the db-service by changing the connection-string to:
`mongoose.connect('mongodb://db/test', { useMongoClient: true })`
so we use the name of the service (db) instead of localhost. Docker will take care of the network connection.

if you run this code log in the terminal window where you see the output will hopefully write out "meow" and you know you are ready to go.

