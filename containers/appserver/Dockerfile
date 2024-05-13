FROM ubuntu:latest
USER root
RUN apt-get update && apt-get install -y openjdk-17-jre-headless
RUN apt-get update && apt-get install -y openssh-server
RUN apt-get update && apt-get install -y sshpass
RUN apt-get update && apt-get install -y ansible-core
RUN mkdir /var/run/sshd
EXPOSE 22
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
RUN useradd -ms /bin/bash jenkins -d /var/jenkins_home
RUN usermod -aG sudo jenkins
RUN echo "<username>:<password>" | chpasswd
RUN touch /var/jenkins_home/download.yml
RUN chown jenkins:jenkins /var/jenkins_home/download.yml
RUN chmod 700 /var/jenkins_home/download.yml
RUN echo '\
---\n\
- name: Download jar file\n\
  hosts: localhost\n\
  tasks:\n\
    - name: Download from nexus\n\
      ansible.builtin.get_url:\n\
        url: http://<IP>:8081/repository/Local_maven/com/example/demo/0.0.1/demo-0.0.1.jar\n\
        dest: /tmp/demo.jar\n\
        mode: '0755'\n\
    - name: Define java and run\n\
      shell: "USEDJAVA=$(which java);$USEDJAVA -jar /tmp/demo.jar"\n\
...\n\
'\
>>  /var/jenkins_home/download.yml 
CMD ["/usr/sbin/sshd", "-D"]


