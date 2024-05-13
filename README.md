# cicd-docker
pull code to jenkin
maven build code
deploy code to Docker

https://github.com/yankils/Simple-DevOps-Project/blob/master/Docker/Docker_Installation_Steps.MD

docker --version 

sudo vi /etc/hostname
dockerhost

sudo useradd dockeradmin
sudo passwd dockeradmin
(Benja_0983541544)

sudo usermod -aG docker dockeradmin

docker pull tomcat
docker run -d --name test-tomcat-server -p 8090:8080 tomcat:latest

change port tomcat
docker exec -it test-tomcat-server /bin/bash
cd webapps.dist
cp -R * ../webapps/
cd ../webapps

http://172.19.194.148:8090/

docker stop test-tomcat-server



Docker file
FROM
-pull centos from dockerhub
RUN
-install java 
RUN
-create /opt/tomcat directory
WORKDIR
--change work directory to /opt/tomcat
ADD/RUN
Download tomcat packages
RUN
-Extract tar.gz file
RUN
-rename to tomcat directory
EXPOSE
tell to docker that it runs on port 8081
CMD
-start tomcat services

/root
vi Dockerfile
FROM amazonlinux:latest
RUN yum install java -y
RUN mkdir /opt/tomcat
WORKDIR /opt/tomcat
ADD https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.89/bin/apache-tomcat-9.0.89.tar.gz
RUN tar -xvzf apache-tomcat-9.0.89.tar.gz
RUN mv apache-tomcat-9.0.89/* /opt/tomcat
EXPOSE 8081
CMD ["/opt/tomcat/bin/catalina.sh","run"]



อันนี้ใช้ได้
# Use Ubuntu as the base image
FROM ubuntu:latest

# Install dependencies
RUN apt-get update && \
    apt-get install -y curl tar && \
    apt-get clean

# Install Java (OpenJDK 8)
RUN apt-get install -y openjdk-8-jdk

# Set environment variables for Java
ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ENV PATH=$PATH:$JAVA_HOME/bin

# Create directory for Tomcat installation
RUN mkdir /opt/tomcat

# Download and extract Apache Tomcat
WORKDIR /opt/tomcat
RUN curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.89/bin/apache-tomcat-9.0.89.tar.gz && \
    tar -xvf apache-tomcat-9.0.89.tar.gz --strip-components=1 && \
    rm apache-tomcat-9.0.89.tar.gz

# Expose Tomcat port
EXPOSE 8080

# Start Tomcat
CMD ["bin/catalina.sh", "run"]



docker build -t tomcat-test .
docker run -d --name mytomcat-server-doc -p 8083:8080 tomcat-test
เปิดไม่ได้เพราะ ยังไม่ได้ config port ใน tomcat

vi Dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps

docker build -t demotomcat .
docker run -d --name tomcat-server-pull -p 8083:8080 demotomcat


Integrate Docker with Jenkins
create a dockeradmin user
install "Publish Over ssh" plugin
add Dockerhost to jenkins "configure systems"

/root
cat /etc/passwd
cat /etc/group
sudo useradd dockeradmin
sudo passwd dockeradmin
(Benja_0983541544)

sudo usermod -aG docker dockeradmin

add ไปแล้วตั้งแต่แรกๆ
id dockeradmin

vi /etc/ssh/sshd_config
search 
/Password
uncomment PasswordAuthentication yes

service sshd reload

Jenkins console
install plugin "Publish Over SSH"

manage jenkins
system
set
Publish over SSH
SSH Servers -> Add
Name dockerhost
Hostname 172.19.194.148 (ip-address EC2)
Username dockeradmin

Passphrase / Password
Benja_0983541544
password เดี๋ยวกันกับ dockeradmin

และกลับไปที่ EC2
dockeradmin@dockerhost: ssh-keygen

แล้วกลับไปที่หน้า Jenkins console
แล้ว test configuration
apply and save

Build item (Maven project)
Source Code Management -> Git
Poll SCM -> * * * * *
Post-build Actions -> Send build artifacts over SSH
Transfer Set
Source files -> webapp/target/*.war
Remove prefix -> webapp/target
Remote directory -> /home/dockeradmin

apply and save
Build now


back to EC2
cd home/dockeradmin
มันจะขึ้นตาม path 

** Remote directory คือ pathที่จะวางไฟล์
/root
/root/opt
mkdir docker
chown -R dockeradmin:dockeradmin docker
cd
mv Dockerfile /opt/docker/
chown -R dockeradmin:dockeradmin /opt/docker

แก้ไข Dockerfile
vi Dockerfile
FROM tomcat:latest
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
COPY ./*.war /usr/local/tomcat/webapps

docker build -t tomcat-build:v1 .
docker run -d --name tomcatv1 -p 8086:8080 tomcat-build:v1

http://172.19.194.148:8086/
http://172.19.194.148:8086/webapp/

ถ้า แก้ไข code และ push ขึ้นมา ต้องสร้าง image ขึ้นมาใหม่ และรัน container ใหม่

เลยแก้โดยการ ใส่ Exec command
cd /opt/docker;
docker build -t regapp:v1 .;
docker run -d --name registerapp -p 8087:8080 regapp:v1;


Exec command
cd /opt/docker;
docker build -t regapp:v1 .;
docker stop registerapp;
docker rm registerapp;
docker run -d --name registerapp -p 8087:8080 regapp:v1;