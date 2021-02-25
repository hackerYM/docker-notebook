# Docker Notebook

A container is a standard unit of software that packages up code and all its dependencies so the application
runs quickly and reliably from one computing environment to another.


## Tutorial

- [Docker Practice](https://yeasy.gitbooks.io/docker_practice/content/)

- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)


## Knowledge

- Official

    - [Documents & Guides](https://docs.docker.com/get-started/)

    - [Docker command line](https://docs.docker.com/engine/reference/commandline/cli/)
    
- Basic Concept

    - [10 張圖深入理解 Docker 容器和鏡像](http://dockone.io/article/783)

    - [Docker 基本教學 - 從無到有 Docker-Beginners-Guide](https://github.com/twtrubiks/docker-tutorial)

- Install on Ubuntu 18.04

    - [docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)

    - [docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04)
    
    - [uninstall docker](https://askubuntu.com/questions/935569/how-to-completely-uninstall-docker)
    
- MongoDB container

    - [remote connections from mongo docker container](https://stackoverflow.com/questions/37450871/how-to-allow-remote-connections-from-mongo-docker-container)

    - [enable authentication on MongoDB through Docker](https://stackoverflow.com/questions/34559557/how-to-enable-authentication-on-mongodb-through-docker)

- Use Dockerfile build docker image

    - [Python Flask](https://blog.igevin.info/posts/how-to-deploy-flask-apps/)

    - [Java Tomcat](https://steffan.cn/2017/02/10/how-to-build-a-Tomcat-image-with-Dockerfile-and-deploy-war/)

- CI / CD with Docker

    - [Docker Cloud - Build Docker Image](https://ithelp.ithome.com.tw/articles/10191508)
    
    - [Bitbucket pipelines - Build and push a Docker Image](https://confluence.atlassian.com/bitbucket/build-and-push-a-docker-image-to-a-container-registry-884351903.html)
    
    - [Watchtower - Automatic Updates for Docker Containers](https://www.ctl.io/developers/blog/post/watchtower-automatic-updates-for-docker-containers/)

- Docker SDK and API

    - [Docker Engine API (v1.32)](https://docs.docker.com/engine/api/v1.32/)
    
    - [Docker SDK for Python](https://docker-py.readthedocs.io/en/stable/index.html)

    - [docker stats 命令源碼分析結果](https://my.oschina.net/jxcdwangtao/blog/828648)


## Open Sources

- [5 Docker Utilities You Should Know](https://blog.xebialabs.com/2017/05/18/5-docker-utilities-you-should-know/)

### ctop

Top-like interface for Docker Container metrics

```shell=
docker run --rm -ti -v /var/run/docker.sock:/var/run/docker.sock \
    --name ctop quay.io/vektorlab/ctop:latest
```

### glances

A cross-platform curses-based system monitoring tool written in Python

1. terminal

    ```shell=
    docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock:ro \
        --pid host --network host --name glances-terminal nicolargo/glances:stable
    ```

2. web

    ```shell=
    docker run -d --restart="always" -p 8112:61208 -e GLANCES_OPT="-w" \
        -v /var/run/docker.sock:/var/run/docker.sock:ro --pid host \
        --name eyeOn-system nicolargo/glances:stable
    ```

### docker-gc

Docker garbage collection of containers and images

- 環境變數 ```DRY_RUN=1``` ，在實際刪除之前，查詢可被刪除的容器和映象

- 環境變數 ```GRACE_PERIOD_SECONDS=0```（單位：秒），決定刪除多久時間沒被使用的容器。

1. 先查詢會被刪除對象

    ```shell=
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        -e DRY_RUN=1 -e GRACE_PERIOD_SECONDS=0 --name docker-gc spotify/docker-gc
    ```

2. 實際刪除容器和映象

    ```shell=
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        -e GRACE_PERIOD_SECONDS=0 --name docker-gc spotify/docker-gc
    ```

### docker-stress

:bangbang: 小心不要把機器測試到掛掉，參數一次不要條的太高

| Options | description |
| ------- | ----------- |
| - -t, --timeout N   | timeout after N seconds |
| - -c, --cpu N       | spawn N workers spinning on sqrt() |
| - -i, --io N        | spawn N workers spinning on sync() |
| - -m, --vm N        | spawn N workers spinning on malloc()/free() |
| - -m, --vm-bytes B  | malloc B bytes per vm worker (default is 256MB) |
| - -d, --hdd N       | spawn N workers spinning on write()/unlink() |

```shell=
docker run --rm -it --name stess-test progrium/stress \
    --cpu 4 --vm 4 --vm-bytes 512MB --io 8 --hdd 8 --timeout 60s
```

### Portainer

A management UI for managing your Docker hosts or Docker Swarm clusters

- [Official Document](https://portainer.readthedocs.io/en/stable/index.html)

- [Portainer Tutorials](https://blog.ssdnodes.com/blog/tutorial-easier-docker-management-with-portainer/)

1. Generate a certificate and a key

    ```shell=
    ## secure Portainer’s web interface using SSL
   
    ## create volumes file
    mkdir portainer
    cd portainer
    mkdir local-certs
    mkdir local-data
    
    ## create SSL keys to use https
    cd local-certs
    openssl genrsa -out portainer.key 2048
    openssl ecparam -genkey -name secp384r1 -out portainer.key
    openssl req -new -x509 -sha256 -key portainer.key -out portainer.crt -days 3650
    ```

2. Open VM's port 8090, 8091

3. Creating a new overlay network in your Swarm cluster

    ```shell=
    docker network create --driver overlay --attachable portainer_agent_network
    ```

4. Deploying the Agent as a global service in your cluster

    ```shell=
    docker service create \
        --name portainer_agent \
        --limit-memory 128m \
        --network portainer_agent_network \
        -e AGENT_CLUSTER_ADDR=tasks.portainer_agent \
        -e AGENT_PORT=8091 \
        --mode global \
        --constraint 'node.platform.os == linux' \
        --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
        --mount type=bind,src=//var/lib/docker/volumes,dst=/var/lib/docker/volumes \
        portainer/agent
    ```

5. Deploying the Portainer instance as a service

    ```shell=
    docker service create \
        --name portainer \
        --network portainer_agent_network \
        --publish 8090:9000 \
        --replicas=1 \
        --constraint 'node.role == manager' \
        --mount type=bind,src=//home/eyeon-user/portainer/local-certs,dst=/certs \
        --mount type=bind,src=//home/eyeon-user/portainer/local-data,dst=/data \
        portainer/portainer --ssl --sslcert /certs/portainer.crt --sslkey /certs/portainer.key \
        -H "tcp://tasks.portainer_agent:8091" --tlsskipverify
    ```

6. Connect ```https://host-ip:8090``` and set admin account and password

### Watchtower

Watching your Docker containers and automatically restarting them whenever their base image is refreshed.

If pulling images from private Docker registries, supply registry authentication credentials by mounting
the host's docker config file into the container.

```shell=
docker run -d \
  --name watchtower \
  -v /home/<user>/.docker/config.json:/config.json \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower container_to_watch --debug
```


## Docker Management

### Docker image

```shell=
## Get list images
docker images

## Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
docker image tag SOURCE_IMAGE TARGET_IMAGE

## Search the Docker Hub for images
docker search IMAGE_NAME

## Remove one or more images
docker rmi IMAGE_NAME|IMAGE_NAMEs

## Build an image from a Dockerfile / -t: Name and optionally a tag
docker build -t TARGET_IMAGE [PATH]
```

### Docker container

```shell=
## Get list containers / -a: Show all containers, -s: Display total file sizes
docker ps -as

## Run a command in a new container (Important!! Options have to read docs)
## --name: Container's name, -p: Publish a container’s port(s) to the host,
## -d: Run container in background, -it: Keep STDIN open even if not attached
docker run --name CONTAINER_NAME -p HOST_IP:INNER_IP -d -it SOURCE_IMAGE

## Run a command in a running container
## -it: Keep STDIN open even if not attached and Allocate a pseudo-TTY
docker exec -it CONTAINER_NAME bash

## Display a live stream of container(s) resource usage statistics
## --no-stream: Disable streaming stats and only pull the first result
docker stats --no-stream

## Fetch the logs of a container
docker logs CONTAINER_NAME

## Control container
docker start CONTAINER_NAME
docker restart CONTAINER_NAME
docker pause CONTAINER_NAME
docker stop CONTAINER_NAME

## Remove one or more containers
## -f: Force the removal of a running container (uses SIGKILL)
docker rm -f CONTAINER_NAME|CONTAINER_NAMEs

## Return low-level information on Docker objects / detail info
docker inspect CONTAINER_NAME
```

### Docker volume

```shell=
## Get list volumes
docker volume ls

## Remove all unused local volumes. 
## Unused local volumes are those which are not referenced by any containers
docker volume prune

## Display detailed information on one or more volumes
docker volume inspect VOLUME_NAME
```

### Docker register

- [Harbor - Manage and serve container images in a secure environment](https://goharbor.io/docs/)

```shell=
## Pull an image or a repository from a registry
docker pull IMAGE_NAME[:TAG]

## Log in to a cotainer registry
docker login

## Push an image or a repository to a registry
docker push IMAGE_NAME[:TAG]
```

### Docker network

- [Docker 學習與實踐 - 網絡篇](https://blog.waterstrong.me/docker-networking/)

- [Dig Into Docker Bridge Network](https://www.hwchiu.com/netfilter-eiptables-iii.html)

```shell=
## List networks
docker network ls

## Remove all unused networks
docker network prune

## Display detailed information on one or more networks
docker network inspect NETWORK_NAME 
```

### Docker system

```shell=
## Show docker images, containers, volumes total count and disk usage
docker system df
docker system df -v

## Display system-wide information
docker system info

## Remove all unused images, containers, volumes, networks, build cache
docker system prune --all
```


## Container Orchestration

### Articles

- [Compare Kubernetes vs Docker Swarm](https://platform9.com/blog/kubernetes-docker-swarm-compared/)

- [Raft Consensus Algorithm - Understandable Distributed Consensus](http://thesecretlivesofdata.com/raft/)

### Docker Compose

**Defining and running multi-container Docker applications**

Docker Compose 是一個工具能夠執行多個 container，你可以使用 docker-compose.yml 來撰寫 Docker Compose 的服務腳本，定義完成後只要一個指令就能啟動所有服務。

三個步驟使用 Docker Compose：

1. 定義好你的 Dockerfile，使得可以重複產出 image

2. 定義好你的 docker-compose.yml，以便於快速啟動多項服務

3. 執行 docker-compose up 啟動所有服務


使用一個 Dockerfile 模板文件，可以讓用戶很方便的定義一個單獨的應用容器。然而，在日常工作中，經常會碰到需要多個容器相互配合來完成某項任務的情況。

例如，要實現一個 Web ，除了 Web 服務容器本身，往往要再加上後端的數據庫容器，甚至負載均衡容器等等。

允許用戶通過一個單獨的 docker-compose.yml 模板文件 (YAML 格式)來定義一組相關聯的應用容器為一個項目 (project)。

Compose 中有兩個重要的概念：

- 服務 (service)：一個應用的容器，實際上可以包括若干運行相同鏡像的容器實例。

- 項目 (project)：關聯的應用容器組成完整業務單元，在 docker-compose.yml 文件定義。

Compose 的默認管理對像是項目，通過子命令對項目中的一組容器進行便捷地生命週期管理。

- [Docker Compose 命令說明](https://yeasy.gitbooks.io/docker_practice/content/compose/commands.html)

- [Docker Compose 模板文件](https://yeasy.gitbooks.io/docker_practice/content/compose/compose_file.html)

```shell=
## Builds, creates, starts, and attaches to container for a service (start-up)
## -d: Run containers in the background
docker-compose -f ai-server-compose.yml up -d

## List containers
docker-compose -f ai-server-compose.yml ps

## Check container and images
docker ps -a
dcoekr images

## Restarts all stopped and running services
docker-compose -f ai-server-compose.yml restart

## Displays log output from services
docker-compose -f ai-server-compose.yml logs
docker-compose -f ai-server-compose.yml logs -f

## Stops containers and removes container, network, volume, images(shut-down)
docker-compose -f ai-server-compose.yml down -v
```

### Docker Machine

**Install Docker Engine on virtual hosts, manage the hosts with docker-machine.**

- Install on Linux

```shell=
## Install virtualbox for create Docker Machine's driver
sudo apt-get install virtualbox

## Download the Docker Machine binary and extract it to your PATH.
base=https://github.com/docker/machine/releases/download/v0.14.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
  
## Check the installation by displaying the Machine version
docker-machine version
```
- First Step on Docker Machine

```shell=
## Create machines, vm1, vm2, vm3
docker-machine create --driver virtualbox vm1
docker-machine create --driver virtualbox vm2
docker-machine create --driver virtualbox vm3

## List machines
docker-machine ls

## Control a machine
docker-machine restart <machine-name>
docker-machine start <machine-name>
docker-machine stop <machine-name>
docker-machine rm <machine-name>

## Log into or run a command on a machine with SSH
docker-machine ssh <machine-name>

## Copy files from your local host to a machine
docker-machine scp -d <file-name> <machine-name>:/home/docker/
```

### Docker Swarm

**Cluster management and orchestration features embedded in the Docker Engine**

![Sample](https://i.imgur.com/fobMffk.png)

1. [Docker Swarm 基本教學 - 從無到有 Docker-Swarm-Beginners-Guide](https://github.com/twtrubiks/docker-swarm-tutorial)

1. [Docker Swarm 深入淺出 ](https://www.bookstack.cn/read/docker-swarm-guides/README.md)

1. [Docker 管理工具 - Swarm 部署記錄](https://hk.saowen.com/a/fd2b0303c8b64a7c9ae507500758032f52ac157e89f78124010ac54916f1d564)

1. [Bootstrapping a Docker Swarm Mode Cluster](https://semaphoreci.com/community/tutorials/bootstrapping-a-docker-swarm-mode-cluster)

1. Network communication between services

    - Use [overlay](https://docs.docker.com/network/overlay/) and [bridge](https://docs.docker.com/network/bridge/) networks

    - [Use swarm mode routing mesh (負載平衡機制)](https://docs.docker.com/engine/swarm/ingress/)
    
    - [What’s the Docker Swarm “–advertise-addr” ?](https://boxboat.com/2016/08/17/whats-docker-swarm-advertise-addr/)

1. Open VM ports

    :bangbang: 在建立叢集前，請確認主機的防火牆能讓 swarm 需求的埠開放，需要開啟主機之間的埠，以下埠必須可用。
    
    - 2377：TCP埠2377，用於叢集管理通訊
    
    - 7946：TCP和UDP埠7946，用於節點之間的通訊
    
    - 4789：TCP和UDP埠4789，用於覆蓋網路流量 

1. Set up machines

    - Docker Node

    ```shell=
    ## Manager Node - initialize a swarm (default port is 2377)
    docker swarm init
    docker swarm join-token worker
    
    ## Worker Node - add a worker to this swarm
    docker swarm join --token SWMTKN-1-6152hty2ge5tl7y8j2fw8qz0kf8n62b8jv7ah2dl654w0zydkl-etrc1wp5mzybiwpuzwh6eas7k 172.30.0.186:2377
    
    ## Manager Node - display system-wide information
    docker info
    
    ## Manage Swarm nodes
    docker node ls
    docker node inspect --pretty <node-name>
    
    ## Demote nodes from manager / Promote nodes to manager in the swarm
    docker node promote <node-name>
    docker node demote <node-name>
    ```
    
    - Docker Service
    
    ```shell=
    ## Create a new service
    docker service create --name=<service-name> -p 8080:80 --replicas 2 <image>
    
    ## List services / List the tasks of services
    docker service ls
    docker service ps <service-name>
    
    ## Display detailed information on one or more services
    docker service inspect --pretty <service-name>
    
    ## Fetch the logs of a service or task
    docker service logs -f <service-name>
    docker service logs --no-task-ids --since <time> -f <service-name>
    docker service logs --raw --since <time> -f <service-name>
    
    ## Scale up or down replicated services / Update a service
    docker service scale <service-name>=<replicated-count>
    docker service update <service-name>
    
    ## Revert changes to a service's configuration (scale, update ...)
    docker service rollback <service-name>
    
    ## Remove one or more services
    docker service rm <service-name>
    ```
    
    - Visualization
    
    ```shell=
    docker service create --name=viz --publish=8080:8080/tcp \
        --constraint=node.role==manager --mount=type=bind,src=/var \
        /run/docker.sock,dst=/var/run/docker.sock dockersamples/visualizer
    ```
    
    - Deployment
    
    ```shell=
    ## Deploy a new stack or update an existing stack
    docker stack -c <file-name.yml> <stack-name>
    
    ## List stacks / List the tasks in the stack
    docker stack ls <stack-name>
    
    docker stack ps <stack-name>
    
    ## Remove one or more stacks
    docker stack rm <stack-name>
    ```

1. Configure an external load balance

    ![Sample](https://i.imgur.com/X8o6RpF.png)


## Examples

### Dockerfile

Dockerfile 是一個文本文件，其內包含了一條條的指令 (Instruction)，每一條指令構建一層，因此每一條指令的內容，就是描述該層應當如何構建。

1. Python Flask

    ```Dockerfile=
    FROM python:3.7-slim
    WORKDIR home/ming
    
    # update and install the system packages
    
    RUN apt-get update && \
        apt-get install -y curl gcc libcairo2-dev libmagic-dev && \
        rm -rf /var/lib/apt/lists/*
    
    # download and install app dependencies
    
    COPY ["./pipelines/requirements.txt", "./pipelines/"]
    RUN pip install --upgrade pip && \
        pip install --no-cache-dir -r pipelines/requirements.txt
    
    # give permission to script test file
    
    COPY [".", "./"]
    RUN ["chmod", "+x", "./pipelines/app-quality.sh"]
    
    # run the backend service
    
    EXPOSE 8080
    ENTRYPOINT ["./pipelines/app-run.sh"]
    ```

2. Java Tomcat

    ```Dockerfile=
    From tomcat:9-jre8-alpine
    WORKDIR home/ming

    ENV server_war_file=cloudtalk-acs-mongo##3.05.05.45.war
    
    # Copy to images tomcat path

    ADD ${server_war_file} /usr/local/tomcat/webapps/
    CMD ["catalina.sh", "run"]
    ```

3. Java Tomcat - Https

    - [docker tomcat keytool 添加 ssl 認證](http://blog.51cto.com/mannerwang/1857447)
    
    - tomcat-key
    
        ```shell=
        ## keytool 生成 .keystore，密碼及關鍵信息請自行補足
        keytool -genkey -alias tomact -keyalg RSA -keystore ./.tomcat-key \
                -storepass raylios -validity 3650
                
        ## 有效期10年，查看證書信息，需要輸入密碼
        keytool -list -v -keystore ./.tomcat-key -storepass raylios
        ```

    - tomcat-users.xml
    
        ```xml=
        <tomcat-users>
        
          <role rolename="manager-gui"/>
          <role rolename="admin-gui"/>
          <user username="<secret>" password="<secret>" roles="manager-gui,admin-gui"/>
        
        </tomcat-users>
        ```

    - server.xml
    
        ```xml=
        <!-- Define a SSL HTTP/1.1 Connector on port 8443
             This connector uses the BIO implementation that requires the JSSE
             style configuration. When using the APR/native implementation, the
             OpenSSL style configuration is required as described in the APR/native
             documentation -->
        
        <Connector port="8081" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" 
               keystoreFile="/usr/local/tomcat/conf/.tomcat-key" keystorePass="raylios"/>
        ```

    - Dockerfile
    
        ```Dockerfile=
        FROM tomcat:7-jre8-alpine
        WORKDIR home/ming

        RUN apk add --no-cache nano
        
        COPY ["tomcat-users.xml", ".tomcat-key", "server.xml", "./conf/"]
        EXPOSE 8080 8081
        ```

### Cluster

1. [Docker Compose](./docker-compose.yml)

1. [Docker Stack](./docker-stack.yml)

### CI / CD

1. [GitLab CI](./.gitlab-ci.yml)
   
1. [Bitbucket Pipeline](./bitbucket-pipelines.yml)


## Troubleshooting

1. [Install and start rsyslog on Ubuntu Linux](https://www.rsyslog.com/tag/ubuntu/)

1. [Alpine shell can't find file in docker](https://serverfault.com/questions/883625/alpine-shell-cant-find-file-in-docker)

1. [Gevent doesn't install in the alpine-python](https://github.com/jfloff/alpine-python/issues/17)

1. [Swarm mode not load balancing](https://forums.docker.com/t/swarm-mode-not-load-balancing/24764)

1. [Python-alpine cffi package dependencies missing](https://github.com/gliderlabs/docker-alpine/issues/297)

1. [Stuck in rsyslogd process after run the image](https://github.com/nimmis/docker-alpine-micro/issues/1)
