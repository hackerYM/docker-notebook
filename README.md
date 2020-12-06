# Docker - Containers as a Service 

###### tags `container` `2019`

**Build, Manage and Secure Your Apps Anywhere. Your Way.**


## Tutorial

- [Docker GitBook - 從入門到實踐](https://yeasy.gitbooks.io/docker_practice/content/)

- [Docker 問答錄 (100 問)](https://blog.lab99.org/post/docker-2016-07-14-faq.html)

- [dockone.io 技術論壇](http://dockone.io/)

- [awesome-docker](https://github.com/veggiemonk/awesome-docker)


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


## Useful Open Source

- [5 Docker Utilities You Should Know](https://blog.xebialabs.com/2017/05/18/5-docker-utilities-you-should-know/)

### ctop

**Top-like interface for Docker Container metrics**

```shell=
docker run --rm -ti -v /var/run/docker.sock:/var/run/docker.sock \
    --name ctop quay.io/vektorlab/ctop:latest
```

### glances

**A cross-platform curses-based system monitoring tool written in Python**

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

**Docker garbage collection of containers and images**

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

:bangbang: 小心不要把 VM 測試到掛掉，參數一次不要條太高。

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

### portainer

**A management UI for managing your Docker hosts or Docker Swarm clusters**

- [Official Document](https://portainer.readthedocs.io/en/stable/index.html)

- [Easier Docker Management with Portainer](https://blog.ssdnodes.com/blog/tutorial-easier-docker-management-with-portainer/)

1. Generate a certificate and a key

    **Secure Portainer’s web interface using SSL**

    ```shell=
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

### watchtower

**Watching your Docker containers and automatically restarting them whenever their base image is refreshed.**

If pulling images from private Docker registries, supply registry authentication credentials with the environment variables REPO_USER and REPO_PASS or by mounting the host's docker config file into the container.

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
## example
docker run --name acs -p 8080:8080 -d -it acs:latest 

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

## Log in to a Docker registry & Log out from a Docker registry
docker login
docker logout

## Push an image or a repository to a registry -> have to login Docker
docker push IMAGE_NAME[:TAG]
```

### Docker network

- [Docker 學習與實踐 - 網絡篇](https://blog.waterstrong.me/docker-networking/)

- [Dig Into Docker Bridge Network By iptables/ebtables](https://www.hwchiu.com/netfilter-eiptables-iii.html)

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


## Cluster & Orchestration

### Article

1. [Compare Kubernetes vs Docker Swarm](https://platform9.com/blog/kubernetes-docker-swarm-compared/)

2. [巔峰對決之 Swarm、Kubernetes、Mesos](https://itw01.com/G6DIEY6.html)

3. [Docker 编排工具最佳選擇是 swarm/kubernetes/Mesos ?](https://www.zhihu.com/question/55391506)

4. [世界是 container 的，也是 microservice 的，但最終還是 serverless](https://hk.saowen.com/a/8c4a7cbf77b2d791cac127d2c7f7e65dd60362c9f5c01ee7e67fa6f0e142018d)

5. [Raft Consensus Algorithm - Understandable Distributed Consensus](http://thesecretlivesofdata.com/raft/)

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

**install Docker Engine on virtual hosts, manage the hosts with docker-machine.**

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

![](https://i.imgur.com/fobMffk.png)

1. [Docker Swarm 基本教學 - 從無到有 Docker-Swarm-Beginners-Guide](https://github.com/twtrubiks/docker-swarm-tutorial)

2. [Docker Swarm 深入淺出 ](https://www.bookstack.cn/read/docker-swarm-guides/README.md)

3. [Docker 管理工具 - Swarm 部署記錄](https://hk.saowen.com/a/fd2b0303c8b64a7c9ae507500758032f52ac157e89f78124010ac54916f1d564)

4. [Bootstrapping a Docker Swarm Mode Cluster](https://semaphoreci.com/community/tutorials/bootstrapping-a-docker-swarm-mode-cluster)

5. Swarm - service communication

    - [Use swarm mode routing mesh (負載平衡機制)](https://docs.docker.com/engine/swarm/ingress/)

    - [Use overlay networks](https://docs.docker.com/network/overlay/) / [Use bridge networks](https://docs.docker.com/network/bridge/)
    
    - [What’s the Docker Swarm “–advertise-addr” ?](https://boxboat.com/2016/08/17/whats-docker-swarm-advertise-addr/)

- First Step on Docker Swarm

    :bangbang:  在建立叢集前，請確認主機的防火牆能讓 swarm 需求的埠開放，需要開啟主機之間的埠，以下埠必須可用。
    
    - 2377：TCP埠2377，用於叢集管理通訊
    
    - 7946：TCP和UDP埠7946，用於節點之間的通訊
    
    - 4789：TCP和UDP埠4789，用於覆蓋網路流量 


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

- Docker Swarm Visualizer

```shell=
docker service create --name=viz --publish=8080:8080/tcp \
    --constraint=node.role==manager --mount=type=bind,src=/var \
    /run/docker.sock,dst=/var/run/docker.sock dockersamples/visualizer
```

- Docker Stack

```shell=
## Deploy a new stack or update an existing stack
docker stack -c <file-name.yml> <stack-name>

## List stacks / List the tasks in the stack
docker stack ls <stack-name>

docker stack ps <stack-name>

## Remove one or more stacks
docker stack rm <stack-name>
```

- Configure an external load balance and using the routing mesh

![](https://i.imgur.com/X8o6RpF.png)


## Example

### Dockerfile

Dockerfile 是一個文本文件，其內包含了一條條的指令 (Instruction)，每一條指令構建一層，因此每一條指令的內容，就是描述該層應當如何構建。

[Dockerfile 指令詳解](https://yeasy.gitbooks.io/docker_practice/content/image/build.html)

1. Python Flask

    ```Dockerfile=
    # 使用官方的 Python 執行環境作為基本的 Docker 影像
    FROM python:alpine3.7
    
    # 維護者： docker_user <docker_user at email.com> (@docker_user)
    MAINTAINER yenming yenming@raylios.com
    
    # 建立檔案工作區
    RUN mkdir -p /usr/src/app  && \
        mkdir -p /var/log/gunicorn
    
    # 設定工作目錄為 /usr/src/app
    WORKDIR /usr/src/app
    
    # 複製目前目錄下的內容, 放進 Docker 容器中的 /usr/src/app
    COPY . /usr/src/app
    
    # 安裝 requirements.txt 中所列的必要套件, 並延長安裝時間
    RUN pip install --default-timeout=100 --no-cache-dir gunicorn && \
        pip install --default-timeout=100 --no-cache-dir -r requirements.txt
    
    # 讓 80 連接埠可以從 Docker 容器外部存取
    ENV PORT 80
    EXPOSE 80
    
    # 當 Docker 容器啟動時, 自動執行 ai-raylios.py
    # Gunicorn 是一個 Python WSGI Unix - http web service, 開啟 2 個 worker
    CMD ["/usr/local/bin/gunicorn", "-w", "2", "-b", ":80", "ai-raylios:app"]
    
    ```

2. Java Tomcat

    ```Dockerfile=
    # Pull base image
    From tomcat:9-jre8-alpine
    
    # Maintainer
    MAINTAINER "ming yenming@raylios.com"
    
    # set Env parameter
    ENV server_war_file=cloudtalk-acs-mongo##3.05.05.45.war
    
    # Copy to images tomcat path
    ADD ${server_war_file} /usr/local/tomcat/webapps/
    
    CMD ["catalina.sh", "run"]
    ```

3. Java Tomcat - Https

    [docker tomcat keytool 添加 ssl 認證](http://blog.51cto.com/mannerwang/1857447)
    
    - tomcat-key
    
        ```shell=
        ## keytool 生成 .keystore，密碼及關鍵信息請自行補足
        keytool -genkey -alias tomact -keyalg RSA -keystore ./.tomcat-key \
                -storepass raylios -validity 3650
                
        ## 有效期10年，查看證書信息，需要輸入密碼
        keytool -list -v -keystore ./.tomcat-key -storepass raylios
        ```

    - tomcat-users.xml
    
        **修改部份的文件**
    
        ```xml=
        <tomcat-users>
        
          <role rolename="manager-gui"/>
          <role rolename="admin-gui"/>
          <user username="<secret>" password="<secret>" roles="manager-gui,admin-gui"/>
        
        </tomcat-users>
        ```

    - server.xml
    
        **修改部份的文件**
    
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
        
        RUN apk add --no-cache nano
        
        COPY ["tomcat-users.xml", ".tomcat-key", "server.xml", "./conf/"]
        
        EXPOSE 8080 8081
        ```

4. Go server

    - Dockerfile
    
        ```Dockerfile=
        #FROM ubuntu:18.04
        FROM alpine:3.8
        
        MAINTAINER yenming yenming@raylios.com
        
        RUN mkdir -p /home/eyeon-user
        COPY . /home/eyeon-user
        
        WORKDIR /home/eyeon-user/servers
        
        #RUN apt-get update \
        # && apt-get install -y curl \
        # && apt-get install -y rsyslog
        
        RUN apk add --no-cache libc6-compat rsyslog curl \
         && rm -rf /var/cache/apk/*
        
        CMD ["./cserver"]
        ```

    - cserver
    
        ```sequence=
        #!/bin/sh
        # Auto Start Servers:
        # DVR(mediaGW), File servers need to connect to Azure Blob
        
        pubip=$(curl v4.ifconfig.co)
        LOCAL_IP=$pubip
        PACKAGE_PATH=/home/eyeon-user/servers/
        
        MASTER_URL="http://23.100.89.3:8082"
        
        MEDIAGW_URL="http://$LOCAL_IP:8804"
        MEDIAGW_PORT=8804
        MEDIAGW_KEY_PATH=/home/eyeon-user/servers/keys/key_mg1
        
        # start syslog
        /usr/sbin/rsyslogd
        
        # run MediaGW
        cd /home/eyeon-user/servers/mediaGW
        commandStr="./mediaGateway -k $MEDIAGW_KEY_PATH -m $MASTER_URL -p :$MEDIAGW_PORT -u $MEDIAGW_URL; exec bash"
        sh -c "$commandStr"
        
        exit
        ```

5. Smaller Python Image

    ```Dockerfile=
    FROM alpine:3.8
    
    RUN apk add --no-cache python3 && \
        python3 -m ensurepip && \
        rm -r /usr/lib/python*/ensurepip && \
        pip3 install --upgrade pip setuptools && \
        if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi && \
        if [[ ! -e /usr/bin/python ]]; then ln -sf /usr/bin/python3 /usr/bin/python; fi && \
        rm -r /root/.cache
    ```

### Docker Compose

1. Python server

    ```yaml=
    version: "3"
    
    services:
      
      # API server 
      ai-agent:
        image: raylios.azurecr.io/ai-agent:test-env
        container_name: ai-agent
        ports:
          - "8080:8080"
        environment:
          - username=<secert>
          - password=<secert>
          - config=test
        networks:
          - ai-net
    
      # Web service
      ai-web:
        image: raylios.azurecr.io/ai-web:test-env
        container_name: ai-web
        ports:
          - "8081:8081"
        environment:
          - config=test
        networks:
          - ai-net
            
      # CRM interface
      crm-port:
        image: raylios.azurecr.io/crm-port:test-env
        container_name: crm-port
        ports:
          - "8084:8084"
        environment:
          - config=test
        networks:
          - ai-net
    
      # Auto deployment container - remember to change volumes - home/<user-name>/...
      watchtower:
        image: v2tec/watchtower
        container_name: ai-server-watch
        restart: always
        networks:
          - ai-net
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
          - /home/eyeon-user/.docker/config.json:/config.json
        command: --interval 300 --cleanup ai-agent ai-web
    
    networks:
      ai-net:
        driver: bridge
    ```

2. go-server

    ```yaml=
    version: "2"
    
    services:
    
      file-blob:
        image: raylios/file-blob
        container_name: file-blob
        restart: always
        build:
          context: fileBlob/
        networks:
          - go-server
        ports:
          - "8805:8805"
    
      media-gw:
        image: raylios/media-gw
        container_name: media-gw
        restart: always
        build:
          context: mediaGateway/
        networks:
          - go-server
        ports:
          - "8804:8804"
    
    networks:
      go-server:
        driver: bridge
    ```

### Docker Stack

1. Python server

    ```yaml=
    version: "3.2"
    
    services:
      
      # API server 
      dash-api:
        image: raylios.azurecr.io/dash-api:test-env
        ports:
          - "8082:8082"
        environment:
          - username=<secret>
          - password=<secret>
          - config=test
        networks:
          - dash-net
        deploy:
          mode: replicated 
          replicas: 1
          update_config:
            parallelism: 1
            delay: 5s
          restart_policy:
            condition: on-failure
    
      # Web service
      dash-web:
        image: raylios.azurecr.io/dash-web:test-env
        ports:
          - "8083:8083"
        environment:
          - config=test
        networks:
          - dash-net
        deploy:
          mode: replicated
          replicas: 1
          update_config:
            parallelism: 1
            delay: 5s
          restart_policy:
            condition: on-failure
    
    networks:
      dash-net:
        driver: overlay
    ```

## CI / CD in Docker
**Continuous Integration & Continuous Delivery**

### Article

1. [架構師觀點: 你需要什麼樣的 CI / CD](https://columns.chicken-house.net/2017/08/05/what-cicd-do-you-need/)

2. [微服務架構 #1, WHY Microservices?](https://columns.chicken-house.net/2016/09/15/microservice-case-study-01/)


### Bitbucket Pipeline

```yaml=
# Set PIPELINES Environment variables: $ACR_LOGIN_SERVER, $ACR_USERNAME, $ACR_PASSWORD
# Auto unit test app server, and build docker image. Finally, push docker image to registry.

# Maintainer: Raylios Ming
# Date: 2019-01-09

# Registry: Azure Container Registry
# Environment: Test & Demo Env

options:
  docker: true

definitions:
  steps:
    - step: &Unit-test-step
        name: Server Unit Test
        script:
          - export username=$SERVER_USERNAME
          - export password=$SERVER_PASSWORD
          - export config=test

          - apk add --no-cache py-gevent
          - pip install --default-timeout=100 --no-cache-dir -r requirements.txt
          - pytest -v --durations=0 --cov-config=./tests/.coverage --cov=./app --cov-report term-missing

pipelines:
  branches:
    master:
      - step: *Unit-test-step
      - step:
          name: Deploy Demo Env
          deployment: staging
          trigger: manual
          services:
            - docker
          script:
            - export SERVER_NAME="dash-api"
            - export TAG_ANME="demo-env"
            - export IMAGE_NAME=$ACR_LOGIN_SERVER/$SERVER_NAME:$TAG_ANME

            - docker build -t $IMAGE_NAME .
            - docker login --username $ACR_USERNAME --password $ACR_PASSWORD $ACR_LOGIN_SERVER
            - docker push $IMAGE_NAME
            - docker logout $ACR_LOGIN_SERVER

    feature/*:
      - step: *Unit-test-step
      - step:
          name: Deploy Test Env
          deployment: test
          services:
            - docker
          script:
            - export SERVER_NAME="dash-api"
            - export TAG_ANME="test-env"
            - export IMAGE_NAME=$ACR_LOGIN_SERVER/$SERVER_NAME:$TAG_ANME

            - docker build -t $IMAGE_NAME .
            - docker login --username $ACR_USERNAME --password $ACR_PASSWORD $ACR_LOGIN_SERVER
            - docker push $IMAGE_NAME
            - docker logout $ACR_LOGIN_SERVER
```

### TravisCI


## Troubleshooting

1. [Install and start rsyslog on Ubuntu Linux](https://www.rsyslog.com/tag/ubuntu/)

2. [Alpine shell can't find file in docker](https://serverfault.com/questions/883625/alpine-shell-cant-find-file-in-docker)

3. [Docker Containers can not be stopped or removed - permission denied Error]()

4. [Gevent doesn't install in the alpine-python](https://github.com/jfloff/alpine-python/issues/17)

5. [Swarm mode not load balancing](https://forums.docker.com/t/swarm-mode-not-load-balancing/24764)

6. [Python-alpine cffi package dependencies missing](https://github.com/gliderlabs/docker-alpine/issues/297)

7. [Stuck in rsyslogd process after run the image](https://github.com/nimmis/docker-alpine-micro/issues/1)

