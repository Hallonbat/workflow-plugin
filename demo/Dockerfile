FROM dockerfile/java
RUN curl -o /tmp/maven.tgz http://mirror.cogentco.com/pub/apache/maven/maven-3/3.2.2/binaries/apache-maven-3.2.2-bin.tar.gz
RUN cd /usr/local; tar xvzf /tmp/maven.tgz; ln -s apache-maven-3.2.2 maven; cd /usr/bin; ln -s /usr/local/maven/bin/mvn .
RUN mkdir -p /var/lib/jenkins/plugins /var/lib/jenkins/jobs/cd

# install Jetty
WORKDIR /opt
ENV VERSION 9.2.2.v20140723
RUN curl "http://mirrors.ibiblio.org/eclipse/jetty/$VERSION/dist/jetty-distribution-$VERSION.tar.gz" | tar xvfz -
RUN ln -sv jetty-distribution-$VERSION jetty
RUN cd /tmp; ln -s /opt/jetty/webapps

# install OpenSSH and allow root to login with password 'root'
RUN apt-get install -y openssh-server
RUN echo 'root:root' | chpasswd

# make a local checkout of the demo workspace
WORKDIR /var/lib/jenkins
RUN git clone https://github.com/jenkinsci/workflow-plugin-pipeline-demo && cd workflow-plugin-pipeline-demo
WORKDIR /var/lib/jenkins/workflow-plugin-pipeline-demo
# Specify a particular master revision in the Dockerfile so we can track changes (and so Docker cache works correctly):
RUN git reset --hard 58677d8a2230c7b8e655c2674a42d2cab655c265
# Prepare local Maven repo:
RUN mvn install && mvn -f sometests -Dmaven.test.failure.ignore clean install

RUN git config --global user.email "demo@jenkins-ci.org" && git config --global user.name "Workflow Demo"

WORKDIR /var/lib/jenkins
# preseed data
ADD jenkins.war /var/lib/jenkins/jenkins.war
ADD JENKINS_HOME /var/lib/jenkins/
ADD ws/plugins/ /var/lib/jenkins/plugins
RUN find /var/lib/jenkins/plugins/* -type d | xargs rm -rf

ADD run.sh /var/lib/jenkins/run.sh
ADD sshd_config /etc/ssh/sshd_config

CMD /var/lib/jenkins/run.sh
EXPOSE 8080 8081 22
