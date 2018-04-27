### How to launch Jupyter-notebook on Athena

#### Choose an docker images (I want to use python3/tensorflow/nvidia certified docker images)
```
$ docker images 
$ docker images | grep nvcr
nvcr.io/nvidia/tensorflow      18.03-py3                                 c0f4402aa77b        8 weeks ago         2.83GB
```

#### Looks like I found the images I want at "nvcr.io/nvidia/tensorflow:18.03-py3"
#### Let me also have a look on nvcr release note and find out if nvidia gave me all i wanted
https://docs.nvidia.com/deeplearning/dgx/tensorflow-release-notes/rel_18.03.html#rel_18.03

#### Let me check the disk space
```
$ df -h /
$ df -h /dgxdata
```

#### It seems like /dgxdata has a lot more space
#### I don't want to flood the root directory and stop myself from being able to use the system.
#### Let me create my username directory for clean file management, later I know all my data is here
```
$ cd /dgxdata
$ mkdir clteo
$ ls /dgxdata/clteo
$ chmod 6777 /dgxdata/clteo
```

#### Now test drive the container and I need to know I can start/stop it
#### After it is started, I must able to go in
#### When I exit the container, it shouldn't die unless I told it to quit
```
$ nvidia-docker run -d -it --name clteo-tf nvcr.io/nvidia/tensorflow:18.03-py3
[docker_id]
$ nvidia-docker exec -it [docker_id] /bin/bash
```

#### Now I want to have a folder that I saved back to the docker host, I can use "-v [docker host]:[docker container]"
#### and I want to expose my container port at docker host "-p [docker host]:[docker container]"
```
$ nvidia-docker -v /dgxdata/clteo:/data -p 9008:8888
```

#### Hm.. Let me also check who is using which GPU
#### Run this command at docker host
```
$ nvidia-smi
```

#### It seems like GPU id 4,5 is not in use.
#### LEt me contraint myself to user GPU id 4,5
```
NV_GPU=4,5 nvidia-docker run -ti nvidia/cuda nvidia-smi
```

#### Everything looks good, let me spin up my very own container
#### Using GPU "4,5"
#### Using Nvidia docker image "nvcr.io/nvidia/tensorflow:18.03-py3"
#### using -d so that when I exit it won't die, when I need, I will "exec in
#### using docker volume map docker host directory "/dgxdata/clteo" to container directory "/data"
#### I would use my username "clteo" prefix, "tf" so that later I can find my own container, and i can also refer it as name

```
$ NV_GPU=4,5 nvidia-docker run -d -it -v /dgxdata/clteo:/data -p 9008:8888 --name clteo_tf nvcr.io/nvidia/tensorflow:18.03-py3 
[container_id]
$ docker exec -it clteo_tf /bin/bash
```

#### Which port is in used at container host?
```
$ netstat -ntlp
```

#### Now I inside my container but jupyter notebook isn't found
#### But never mind, since I am the root (GOD)
```
$ apt update -y
$ apt upgrade -y
$ apt install python3-pip -y
$ python --version
$ python3 --version
$ apt install ipython3 -y
$ pip3 install jupyter -y
$ pip3 install --upgrade pip

$ jupyter notebook --ip=0.0.0.0 --allow-root
[W 10:29:54.516 NotebookApp] No web browser found: could not locate runnable browser.
[C 10:29:54.516 NotebookApp]

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://0.0.0.0:8888/?token=390502b15f04b3e307dada26df4176f4146a3dc66700562c
```

#### Everything looks good.
#### Now I will use MobaXTerm portal personal edition (On Windows)
```
$ firefox &
```

#### Access my own jupyter notebook with modification
#### The address in container is http://0.0.0.0:8888/?token=390502b15f04b3e307dada26df4176f4146a3dc66700562c
#### Remember I forward the port to 9008?
#### I will key in this address in firefox http://0.0.0.0:9008/?token=390502b15f04b3e307dada26df4176f4146a3dc66700562c

#### When I finished
#### In container, copy all files to docker host directory, in docker container it is /data
```
$ mv file /data/file
$ mv * /data/
$ exit
```

#### At docker host
```
$ docker stop clteo_tf
$ docker rm clteo_tf
```

#### Check my file is still with host
```
$ ls -l /dgxdata/clteo
```

#### Maybe a better way is to have uid/gid mapping between container and host
#### chmod 6777 /dgxdata/clteo
#### when launch container, you can also map your current user id to container root id "--user 0" 

```
chenglim@brainlab:/dgxdata/clteo$ id
uid=1030(chenglim) gid=1030(chenglim) groups=1030(chenglim),27(sudo),999(docker)
chenglim@brainlab:/dgxdata/clteo$ ls -ld ../clteo
drwsrwsrwx 2 chenglim chenglim 5 Apr 27 14:39 ../clteo
chenglim@brainlab:/dgxdata/clteo$ chmod 6777 ../clteo
chenglim@brainlab:/dgxdata/clteo$ NV_GPU=7 nvidia-docker run -d --user 0 -it -v /dgxdata/clteo:/data -p 9008:8888 --name clteo_tf nvcr.io/nvidia/tensorflow:18.03-py3 6b266d97e755123d1cedab624519b6759e21fd7ed1878f4bec186373bb768e6f
chenglim@brainlab:/dgxdata/clteo$ docker exec -it clteo_tf /bin/bash
root@6b266d97e755:/workspace# cd /data
root@6b266d97e755:/data# ls
a  b  c
root@6b266d97e755:/data# touch e
root@6b266d97e755:/data# touch f
root@6b266d97e755:/data# touch d
root@6b266d97e755:/data# ls -l
total 24
-rw-r--r-- 1 nobody 1030 0 Apr 27 06:39 a
-rw-r--r-- 1 nobody 1030 0 Apr 27 06:39 b
-rw-r--r-- 1 nobody 1030 0 Apr 27 06:39 c
-rw-r--r-- 1 nobody 1030 0 Apr 27 06:48 d
-rw-r--r-- 1 nobody 1030 0 Apr 27 06:48 e
-rw-r--r-- 1 nobody 1030 0 Apr 27 06:48 f
root@6b266d97e755:/data# id
uid=0(root) gid=0(root) groups=0(root)
root@6b266d97e755:/data# exit
exit
chenglim@brainlab:/dgxdata/clteo$ ls -l
total 24
-rw-r--r-- 1 nobody chenglim 0 Apr 27 14:39 a
-rw-r--r-- 1 nobody chenglim 0 Apr 27 14:39 b
-rw-r--r-- 1 nobody chenglim 0 Apr 27 14:39 c
-rw-r--r-- 1 nobody chenglim 0 Apr 27 14:48 d
-rw-r--r-- 1 nobody chenglim 0 Apr 27 14:48 e
-rw-r--r-- 1 nobody chenglim 0 Apr 27 14:48 f
chenglim@brainlab:/dgxdata/clteo$ docker stop clteo_tf && docker rm clteo_tf
clteo_tf
clteo_tf
chenglim@brainlab:/dgxdata/clteo$
```

#### Questions?
Write to chenglim_teo@sutd.edu.sg or ask at Athena Helpdesk slack chat channel.
