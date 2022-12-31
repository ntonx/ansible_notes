FROM ubuntu:latest
RUN apt update && apt install  openssh-server sudo -y
RUN useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 test 
RUN  echo 'test:test' | chpasswd
#User is declared as test and password as test , you can modify executing the last 2 lines
RUN service ssh start
EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]
