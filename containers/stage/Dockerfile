FROM ubuntu:latest
USER root
RUN apt-get update && apt-get install -y openjdk-17-jre-headless
RUN apt-get update && apt-get install -y openssh-server
RUN apt-get update && apt-get install -y sshpass
RUN apt-get update && apt-get install -y git
RUN mkdir /var/run/sshd
EXPOSE 22
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
RUN useradd -ms /bin/bash jenkins -d /var/jenkins_home
RUN usermod -aG sudo jenkins
RUN echo "<username>:<password>" | chpasswd
CMD ["/usr/sbin/sshd", "-D"]