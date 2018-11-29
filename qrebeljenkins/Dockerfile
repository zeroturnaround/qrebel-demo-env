FROM jenkins/jenkins:lts
USER root


RUN apt-get update
RUN apt-get install -y sudo

RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers

# Docker compose to startup the spring-petclinic-microservices app
RUN curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose

#Docker CE
RUN apt-get update && \
apt-get -y install apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common && \
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable" && \
apt-get update && \
apt-get -y install docker-ce

RUN usermod -a -G docker jenkins


# Minimal Jenkins Plugins
COPY --chown=jenkins:jenkins sources/qrebel-jenkins.hpi /var/jenkins_home/plugins/qrebel-jenkins.hpi
RUN /usr/local/bin/install-plugins.sh git matrix-auth workflow-aggregator docker-workflow credentials-binding ws-cleanup greenballs

# Skip initial setup
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

# Copy config
COPY --chown=jenkins:jenkins sources/jobs/spring-petclinic-microservices/config.xml /var/jenkins_home/jobs/spring-petclinic-microservices/config.xml
COPY --chown=jenkins:jenkins sources/config.xml /var/jenkins_home/jobs/QRebel-demo/config.xml
COPY --chown=jenkins:jenkins sources/config-plugin.xml /var/jenkins_home/jobs/QRebel-plugin/config.xml

# Copy build history
COPY --chown=jenkins:jenkins sources/jobs/spring-petclinic-microservices/lastStable /var/jenkins_home/jobs/spring-petclinic-microservices/lastStable
COPY --chown=jenkins:jenkins sources/jobs/spring-petclinic-microservices/nextBuildNumber /var/jenkins_home/jobs/spring-petclinic-microservices/nextBuildNumber
COPY --chown=jenkins:jenkins sources/jobs/spring-petclinic-microservices/lastSuccessful /var/jenkins_home/jobs/spring-petclinic-microservices/lastSuccessful
COPY --chown=jenkins:jenkins sources/jobs/spring-petclinic-microservices/builds /var/jenkins_home/jobs/spring-petclinic-microservices/builds/


RUN apt-get update && apt-get install -y maven
RUN mkdir -p /home/jenkins
COPY sources/qrebel-agent.jar /home/jenkins/qrebel-agent.jar

#Install Wait-for-it so we know when spring petclinic is up and running so we can start the test
RUN apt-get install wait-for-it
ENV JENKINS_OPTS --httpPort=8088
EXPOSE 8080 50000 7000 443 8088

# Install Chrome (https://qxf2.com/blog/preparing-a-docker-image-for-running-selenium-tests/)
RUN curl https://dl-ssl.google.com/linux/linux_signing_key.pub -o /tmp/google.pub \
    && cat /tmp/google.pub | apt-key add -; rm /tmp/google.pub \
    && echo 'deb http://dl.google.com/linux/chrome/deb/ stable main' > /etc/apt/sources.list.d/google.list \
    && mkdir -p /usr/share/desktop-directories \
    && apt-get -y update && apt-get install -y google-chrome-stable
# Disable the SUID sandbox so that chrome can launch wgit cithout being in a privileged container
RUN dpkg-divert --add --rename --divert /opt/google/chrome/google-chrome.real /opt/google/chrome/google-chrome \
    && echo "#!/bin/bash\nexec /opt/google/chrome/google-chrome.real --no-sandbox --disable-setuid-sandbox \"\$@\"" > /opt/google/chrome/google-chrome \
    && chmod 755 /opt/google/chrome/google-chrome

# Essential tools and xvfb
RUN apt-get update && apt-get install -y \
    software-properties-common \
    unzip \
    curl \
    xvfb

#Additional plugins for Jenkins
COPY --chown=jenkins:jenkins plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/plugins.txt

USER jenkins

