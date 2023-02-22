上传MongoDB、Spark、Flume、Kafka、JDK到Centos7的/opt/docker_build中



编写Dockerfile配置文件

vim /opt/docker_build/Dockerfile

```dockerfile
FROM centos:7
MAINTAINER ming
# 安装openssh-server和sudo软件包，并且将sshd的UsePAM参数设置成no  
RUN yum install -y openssh-server sudo
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config
# 安装openssh-clients
RUN yum  install -y openssh-clients
RUN echo "root:root" | chpasswd
RUN echo "root   ALL=(ALL)       ALL" >> /etc/sudoers
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
# 启动sshd服务并且暴露22端口  
RUN mkdir /var/run/sshd
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]

# 拷贝并解压jdk                                        
ADD jdk-8u212-linux-x64.tar.gz /usr/local/
RUN mv /usr/local/jdk1.8.0_212 /usr/local/jdk
ENV JAVA_HOME /usr/local/jdk
ENV PATH $JAVA_HOME/bin:$PATH

# 拷贝并解压zookeeper
ADD apache-zookeeper-3.5.7-bin.tar.gz /usr/local
RUN mv /usr/local/apache-zookeeper-3.5.7-bin /usr/local/zookeeper
ENV ZK_HOME /usr/local/zookeeper
ENV PATH $ZK_HOME/bin:$PATH

# 拷贝并解压kafka
ADD kafka_2.12-3.0.0.tgz /usr/local
RUN mv /usr/local/kafka_2.12-3.0.0 /usr/local/kafka
ENV KAFKA_HOME /usr/local/kafka
ENV PATH $KAFKA_HOME/bin:$PATH

# 拷贝并解压flume
ADD apache-flume-1.9.0-bin.tar.gz /usr/local
RUN mv /usr/local/apache-flume-1.9.0-bin /usr/local/flume
ENV FLUME_HOME /usr/local/flume
ENV PATH $FLUME_HOME/bin:$PATH

# 拷贝并解压MongoDB
ADD mongodb-linux-x86_64-rhel70-5.0.14.tgz /usr/local
RUN mv /usr/local/mongodb-linux-x86_64-rhel70-5.0.14 /usr/local/mongodb
ENV MONGODB_HOME /usr/local/mongodb
ENV PATH $MONGODB_HOME/bin:$PATH
```

在/opt/docker_build中构建docker镜像：`docker build -t graduation  .`



创建docker容器

```shell
sudo docker network create --subnet=192.168.120.0/24 docker-br0

docker run --name master --privileged=true -v /opt/cluster/opt:/opt/transfer --net docker-br0 --ip 192.168.120.10 --hostname master -d -P -p 9870:9870 -p 8088:8088 -p 22222:22 graduation
```

