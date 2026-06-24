Docker основы работы с контейнеризацией 

### Задание
- Установите Docker на хост машину https://docs.docker.com/engine/install/ubuntu/
- Установите Docker Compose - как плагин, или как отдельное приложение
- Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)
- Определите разницу между контейнером и образом
- Вывод опишите в домашнем задании.
- Ответьте на вопрос: Можно ли в контейнере собрать ядро?

### Решение

задание выполняем на сервере vs-ubn4  

ставим Docker из репозитория , делаем по инструкции https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

```
# настроим репозиторий
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

```
Устанавливаем пакеты, здесь же кстати ставим и docker-compose-plugin
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

делаем проверку, что docker установился
```
sadmin@lp-ubn4:~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-06-22 15:31:21 UTC; 1min 19s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 40184 (dockerd)
      Tasks: 10
     Memory: 30.3M (peak: 31.1M)
        CPU: 382ms
     CGroup: /system.slice/docker.service
             └─40184 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jun 22 15:31:19 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:19.660817967Z" level=info msg="Restoring containers: start."
Jun 22 15:31:20 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:20.033365339Z" level=info msg="Deleting nftables IPv4 rules" error="running n>
Jun 22 15:31:20 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:20.047649372Z" level=info msg="Deleting nftables IPv6 rules" error="running n>
Jun 22 15:31:21 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:21.135268031Z" level=info msg="Loading containers: done."
Jun 22 15:31:21 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:21.140927350Z" level=info msg="Docker daemon" commit=70eaf5e containerd-snaps>
Jun 22 15:31:21 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:21.141039508Z" level=info msg="Initializing buildkit"
Jun 22 15:31:21 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:21.901913715Z" level=info msg="Completed buildkit initialization"
Jun 22 15:31:21 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:21.905771687Z" level=info msg="Daemon has completed initialization"
Jun 22 15:31:21 lp-ubn4 dockerd[40184]: time="2026-06-22T15:31:21.905821182Z" level=info msg="API listen on /run/docker.sock"
Jun 22 15:31:21 lp-ubn4 systemd[1]: Started docker.service - Docker Application Container Engine.
lines 1-22/22 (END)
sadmin@lp-ubn4:~$
sadmin@lp-ubn4:~$
sadmin@lp-ubn4:~$
sadmin@lp-ubn4:~$
sadmin@lp-ubn4:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
Digest: sha256:96498ffd522e70807ab6384a5c0485a79b9c7c08ca79ba08623edcad1054e62d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

потренируемся, создадим простейший dockerfile с nginx и python

```
# syntax=docker/dockerfile:1
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y nginx python3 python3-pip

# Запускаем Nginx
CMD ["nginx", "-g", "daemon off;"]
```

соберем имидж
```
 sudo docker build -t test-image-nginx.latest .

```

проверяем, имидж появился

```
sadmin@lp-ubn4:~$ sudo docker images
[sudo] password for sadmin:
                                                                                                                                                                                          i Info →   U  In Use
IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest               96498ffd522e       25.9kB         9.49kB    U
test-image-nginx.latest:latest   e19da8959b74        809MB          226MB
sadmin@lp-ubn4:~$
```

запускаем и проверяем страницу  

<img width="583" height="295" alt="image" src="https://github.com/user-attachments/assets/3ea9bfcf-0151-4ef9-a61c-7f3ebafd93b4" />


меняем докер файл на

```
# syntax=docker/dockerfile:1
FROM alpine:3.18
RUN apk add --no-cache nginx python3
# Запускаем Nginx
CMD ["nginx", "-g", "daemon off;"]
```

собираем новый имидж 
```
sadmin@lp-ubn4:~$ sudo docker build -t  nginx-alpine3.18:3.18 .
[+] Building 2.1s (8/8) FINISHED                                                                                                 docker:default
 => [internal] load build definition from Dockerfile                                                                                       0.0s
 => => transferring dockerfile: 184B                                                                                                       0.0s
 => resolve image config for docker-image://docker.io/docker/dockerfile:1                                                                  0.8s
 => CACHED docker-image://docker.io/docker/dockerfile:1@sha256:87999aa3d42bdc6bea60565083ee17e86d1f3339802f543c0d03998580f9cb89            0.0s
 => => resolve docker.io/docker/dockerfile:1@sha256:87999aa3d42bdc6bea60565083ee17e86d1f3339802f543c0d03998580f9cb89                       0.0s
 => [internal] load metadata for docker.io/library/alpine:3.18                                                                             0.7s
 => [internal] load .dockerignore                                                                                                          0.0s
 => => transferring context: 2B                                                                                                            0.0s
 => [1/2] FROM docker.io/library/alpine:3.18@sha256:de0eb0b3f2a47ba1eb89389859a9bd88b28e82f5826b6969ad604979713c2d4f                       0.0s
 => => resolve docker.io/library/alpine:3.18@sha256:de0eb0b3f2a47ba1eb89389859a9bd88b28e82f5826b6969ad604979713c2d4f                       0.0s
 => CACHED [2/2] RUN apk add --no-cache nginx python3                                                                                      0.0s
 => exporting to image                                                                                                                     0.1s
 => => exporting layers                                                                                                                    0.0s
 => => exporting manifest sha256:ead92b222f5405ba97cc65ad130a5240af09e943e4bb4db6e5964f36667b1507                                          0.0s
 => => exporting config sha256:cce6ef711514bba33f6776d385e56b41b1ab69658105fef546ed5eb821c11a3c                                            0.0s
 => => exporting attestation manifest sha256:4ff82ff2af0e6737c1e363a852be29224853e3fba01db44e77b6780d21f9f985                              0.1s
 => => exporting manifest list sha256:ee9f2e655bb3ce83937c78c56155d5b80e1d92544d349d03fa676f5fe95c7658                                     0.0s
 => => naming to docker.io/library/nginx-alpine3.18:3.18                                                                                   0.0s
 => => unpacking to docker.io/library/nginx-alpine3.18:3.18
```

запускаем 
```
sadmin@lp-ubn4:~$ sudo docker run -d -p 8082:80 --name nginx-alpine  nginx-alpine3.18
dfe5b0a3f3a63d64448c7958cb9dca7501c16974d193dca4c2750da9b6e1c299 nginx-alpine
```
смотрим появился запущеный контейнер. И он появился 
```
sadmin@lp-ubn4:~$ sudo docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED              STATUS              PORTS                                     NAMES
dfe5b0a3f3a6   nginx-alpine3.18          "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:8082->80/tcp, [::]:8082->80/tcp   nginx-alpine
66aa97b136fa   nginx-alpine3.18          "nginx -g 'daemon of…"   19 hours ago         Up 19 hours         0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   naughty_wu
d87e4752bed9   test-image-nginx.latest   "nginx -g 'daemon of…"   21 hours ago         Up 21 hours         0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   nostalgic_mcnulty
```









