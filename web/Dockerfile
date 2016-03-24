FROM openmicroscopy/omero-ssh-systemd

MAINTAINER OME

ENV JENKINS_SWARM_VERSION 2.0
ENV LANG en_US.UTF-8
ARG JAVAVER=openjdk18


# To avoid error: sudo: sorry, you must have a tty to run sudo
RUN sed -i -e "s/Defaults    requiretty.*/ #Defaults    requiretty/g" /etc/sudoers

# Download and run omero-install. 
ENV OMERO_INSTALL /tmp/omero-install/linux

RUN yum install -y git && \
    git clone -b java-parameters https://github.com/jburel/omero-install.git /tmp/omero-install

ADD ./settings.env $OMERO_INSTALL/settings.env
RUN chmod +x $OMERO_INSTALL/settings.env

# Customize: run particular scripts from omero-install
RUN rm -f $OMERO_INSTALL/setup_centos_selinux.sh
RUN touch $OMERO_INSTALL/setup_centos_selinux.sh
RUN rm -f $OMERO_INSTALL/setup_omero_db.sh
RUN touch $OMERO_INSTALL/setup_omero_db.sh
# do not create user
RUN rm -f $OMERO_INSTALL/step02_all_setup.sh
RUN touch $OMERO_INSTALL/step02_all_setup.sh
# omeroweb will be isntalled by the job
RUN rm -f $OMERO_INSTALL/step04_all_omero.sh
RUN touch $OMERO_INSTALL/step04_all_omero.sh
# not need nginx
RUN mv $OMERO_INSTALL/step05_centos7_nginx.sh $OMERO_INSTALL/_step05_centos7_nginx.sh
RUN touch $OMERO_INSTALL/step05_centos7_nginx.sh

RUN sed -i.bak -re 's/(.*omero.service.*)/#\1/' $OMERO_INSTALL/step06_centos7_daemon.sh
RUN sed -i.bak -re 's/(systemctl daemon-reload)/#\1/' $OMERO_INSTALL/step06_centos7_daemon.sh


WORKDIR /tmp/omero-install/linux

RUN JAVAVER=$JAVAVER PGVER=nopg bash install_centos7_nginx.sh

RUN yum install -y http://download-aws.ej-technologies.com/exe4j/exe4j_linux_5_0_1.rpm

ADD ./jenkins-slave.sh /tmp/jenkins-slave.sh
ADD ./jenkins.service /etc/systemd/system
RUN systemctl enable jenkins.service

USER omero
WORKDIR /tmp
RUN curl --create-dirs -sSLo /tmp/swarm-client-$JENKINS_SWARM_VERSION-jar-with-dependencies.jar http://maven.jenkins-ci.org/content/repositories/releases/org/jenkins-ci/plugins/swarm-client/$JENKINS_SWARM_VERSION/swarm-client-$JENKINS_SWARM_VERSION-jar-with-dependencies.jar


ENTRYPOINT ["/tmp/jenkins-slave.sh"]