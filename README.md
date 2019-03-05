
# Docker homework
## Introduction
This is the documentation of the docker homework for VITMAC03 course at the Budapest University of Technology and Economics  
Written by: Fábián Füleki  
Neptun code: AEP0TG

## Prerequisites (tested version)

- docker (17.05.0)  
- docker-compose (1.23.2)

## Installing docker-compose and docker

`sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`sudo chmod +x /usr/local/bin/docker-compose`

`sudo apt update && sudo apt install -y --upgrade docker-engine`

## Clone repository

`git clone https://github.com/ffabi/DockerHomework`

## Purpose of each file

file [./app/requirements.txt](./app/requirements.txt)

Describes the needed python packages to be installed

```
flask==1.0.2
redis==3.2.0
```


file [./app/Dockerfile](./app/Dockerfile)

Describes the needed steps for docker to build the webserver image

```
#Use a pre-configured python image
FROM python:3.6-stretch
#Copy the requirements.txt file to the container
ADD . /code
WORKDIR /code
#Install needed python libraries
RUN pip3 install -r requirements.txt
#Set start up command
CMD ["python3", "app.py"]
```


file [./db/Dockerfile](./db/Dockerfile)

Describes the needed steps for docker to build the database image

```
#Set redis-server as base image
FROM redis:alpine
#Start the redis-server at startup
CMD [ "redis-server" ]
```


file [app.py](app.py)

The server app that gathers information from the database server and returns that to the client

```python
import time
import redis
from flask import Flask

app = Flask(__name__)
#set connection to the redis-server
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            #return and increment the hit counter
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'HAKAP class hello! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
	
```
    

file [docker-compose.yml](docker-compose.yml)

Describes how the images should be built and the containers should be started

```yaml
version: '3'
services:
  web:
    #set the location of the Dockerfile
    build: ./app/
    ports:
    #open port 5000 to the flask server
     - "5000:5000"
    volumes:
     - .:/code
     #configure to be the same as the redis-server network
    networks:
      - docker_network

  redis:
    build: ./db/
    #configure to be the same as the flask-server network
    networks:
      - docker_network
  
#create common network for the containers
networks:
  docker_network:

```
  

## Running

To run the two containers, simple run the following

`docker-compose up`
