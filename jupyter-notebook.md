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
$ chmod 777 /dgxdata/clteo
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
$ nvidia-docker -v /dgxdata/clteo:/data -p 9001:8888
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
$ NV_GPU=4,5 nvidia-docker run -d -it -v /dgxdata/clteo:/data -p 9001:8888 --name clteo_tf nvcr.io/nvidia/tensorflow:18.03-py3 
[container_id]
$ docker exec -it clteo_tf /bin/bash
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
#### At container, copy files to docker host directory, in docker it is /data
```
$ mv file /data/file
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

#### Questions?
Write to chenglim_teo@sutd.edu.sg or ask at Athena Helpdesk slack chat channel.
