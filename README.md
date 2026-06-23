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

запускаем
```
```

проверяем страницу
<img width="583" height="295" alt="image" src="https://github.com/user-attachments/assets/3ea9bfcf-0151-4ef9-a61c-7f3ebafd93b4" />






