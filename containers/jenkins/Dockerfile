FROM jenkins/jenkins:latest
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN apt-get update && apt-get install -y maven
RUN apt-get update && apt-get install -y ansible
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
RUN echo "jenkins:<password>" | chpasswd
USER jenkins
RUN jenkins-plugin-cli --plugins pipeline-utility-steps
RUN jenkins-plugin-cli --plugins nexus-artifact-uploader
RUN jenkins-plugin-cli --plugins ansible