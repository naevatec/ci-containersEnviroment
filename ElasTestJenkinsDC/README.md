docker-jenkins-image

[![License badge](https://img.shields.io/badge/license-Apache2-orange.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Docker badge](https://img.shields.io/docker/pulls/gtunon/jenkins-docker-slave.svg)](https://hub.docker.com/r/gtunon/jenkins-docker-slave/)

# Docker Jenkins Image 
----------------------

This is code for setting up a Jenkins into a docker container that can connect to the docker host daemon. 

In order to enable access to the docker host daemon it is necessary to tell the container which is the 
id of the docker group in the host. We achieve this with the export of the enviroment variable DOCKER_GROUP_ID
and declare it as an argument to the image build. 



## Dockerfile
---------------------

### Setting enviroment 

We use always the latest oficial image of Jenkins of the DockerHub repository. At the moment of this code is written
the latest image is [2.32.2](https://hub.docker.com/_/jenkins/)

``  FROM jenkins:latest ``

Defining docker group id as an argument to the Dockerfile

``  ARG dockergroupid``

We define root as the user to run the following instructions for setting the jenkins enviroment inside the container

```
    USER root 

    RUN mkdir /var/log/jenkins &&\
        mkdir /var/cache/jenkins 

    RUN chown -R jenkins:jenkins /var/log/jenkins &&\
        chown -R jenkins:jenkins /var/cache/jenkins 
```


In case you want this folders to be reachable from the host just add this folders as volumes on the *docker-compose.yml* 


### Plugin configuration

In order to obtain a list of plugins installed on a running jenkins use:
``$  curl -sSL "http://user:password@jenkins_url[:port]/pluginManager/api/xml?depth=1&xpath=/*/*/shortName|/*/*/version&wrapper=plugins" | perl -pe 's/.*?<shortName>([\w-]+).*?<version>([^<]+)()(<\/\w+>)+/\1 \2\n/g'|sed 's/ /:/' > plugins.txt``

If you don't need or don't want to preinstall any plugins just comment or delete following lines

```
   COPY plugins/install-plugins.sh install-plugins.sh
   COPY plugins/plugins.txt plugins.txt
```


### Docker configuration 

Docker volumes (sock and bin) 

```
    VOLUME /var/run/docker.sock
    VOLUME /usr/bin/docker
```

For Jenkins being able to run docker commands over the docker host it is not necessary to install a full docker on the container
but add the bin/docker of the host as a volume. And (of course) set permissions. We don't enable all conections to the docker host 
but we add the Jenkins user of the container to the docker group in the host. It maintains somewhat security on our host.

We use the argument defined for adding the same docker group of the host to the container and add the Jenkins user to this group

```
  RUN echo "**** docker_group_id= $dockergroupid *****" &&\
      groupadd -g $dockergroupid docker
      
  RUN usermod -a -G docker jenkins
```

At the end we switch user on the container to jenkins

 `` USER jenkins``

## docker-compose.yml
-----------------------

This file sets the running enviroment for Jenkins many of the configuration set here could be placed
inside the Dockerfile, but we have chose to separate it. 

We use last version of docker-compose yml definition but all the instructions here should be compatible
at least with version 2 (I haven't try)
```
  version: '3'
```
We define just one service but I guess you can use this sames service inside any other docker-compose.yml with more services 
defined (such as nginx, there are many tutorials about setting nginx and Jenkins together)

This will tell the docker daemon that this service should be started whenever the daemon is started.
```
   ...
    restart: always 
```

The build tag indicates Docker how to build the image in stead of using a ready to go image we will create this image 
in the "up" phase so the enviroment can be setted as specified in the Dockerfile.
```
    build: 
```
Whenever the structure of the deployment is as shown in this project the context of the build is .
```
      context: .
```

As specified before, host docker group id should be an argument for the Dockerfile. For defining this parameter we must set 
as an enviroment variable (on the shell) the docker group id (DOCKER_GROUP_ID) we provide a small script that when run as 
```
  $ . ./exp_docker_groupid.sh 
```
over the same console we will run the docker-compose up will set up the variable and would be reachable during the image build
```
      args:
        dockergroupid: ${DOCKER_GROUP_ID}
```

I will define a name for the image in order to be able to push it to dockerHub. If you want to publish in your own repository you 
would want to change this line.
```
    image: gtunon/docker_master:beta
```

This container name is just to make it more traceable
```
    container_name: jenkins_master
```

The ``volumes:`` defined in this part of the service are part defined as main configuration 
```
      - jenkins_home:/var/jenkins_home
      - /etc/localtime:/etc/localtime:ro
```
And particular for our Jenkins functioning. This volumes are just for sharing information with the host or other containers, you 
shoud configure your own volumes that satisfies your own needs.
```
      - /home/ubuntu/dockerImages:/home/ubuntu/dockerImages
      - built_jars:/home/ubuntu/shared/jars
```

We define the following ports as exposed on the host and maped to the container. 
```
 ports:
      - "8081:8080"
      - "50000:50000"
```
The syntaxis is host_exposed_port:container_listening_port
   
Lastly we have the definition of the volumes defined by tags.
```
volumes:
  jenkins_home: {}
  built_jars: {}
```




[![][NaevatecImage]][NaevatecPage]

[NaevatecPage]: https://www.naevatec.com/
[NaevatecImage]: https://www.naevatec.com/sites/default/files/Naevatec%20Transp%20Web2.png
