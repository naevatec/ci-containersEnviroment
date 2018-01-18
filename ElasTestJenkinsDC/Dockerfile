# Docker file for mounting Naevatec Jenkins server

# in order to set DOCKER_GROUP_ID variable the exp_docker_groupid.sh should be
# executed before over the same console with ". ./exp_docker_groupid.sh" and 
# passed as a parameter of the build (dockergroupid)

FROM jenkins/jenkins:2.101 
#latest on 18/01/2018

##########################
#Arguments
##########################
# docker group of the host default 
ARG dockergroupid
ARG jenkins_opts

USER root 

RUN apt-get update && apt-get install -y python3-pip \
    && pip3 install awscli

RUN mkdir /var/log/jenkins &&\
    mkdir /var/cache/jenkins 


RUN chown -R jenkins:jenkins /var/log/jenkins &&\
    chown -R jenkins:jenkins /var/cache/jenkins

# for host access to the jenkins logs and paths configure properly
# in docker-compose.yml with volumes

#TODO: Back-up configuration

##############################
# Plugin configuration
##############################
# in order to obtain a list of plugins installed on a running jenkins use:
# $  curl -sSL "http://user:password@jenkins_url[:port]/pluginManager/api/xml?depth=1&xpath=/*/*/shortName|/*/*/version&wrapper=plugins" | perl -pe 's/.*?<shortName>([\w-]+).*?<version>([^<]+)()(<\/\w+>)+/\1 \2\n/g'|sed 's/ /:/' > plugins.txt

COPY plugins/install-plugins.sh install-plugins.sh
COPY plugins/plugins.txt plugins.txt

ENV REF_DIR .
ENV JENKINS_OPTS ${jenkins_opts}

###############################
#Docker configuration 
##############################
#remember to set the proper volumes on docker-compose.yml

#add docker group to container execute to set env $DOCKER_GROUP_ID
RUN echo "**** docker_group_id= $dockergroupid *****" &&\
    groupadd -g $dockergroupid docker

#add jenkins to docker group
RUN usermod -a -G docker jenkins

USER jenkins

