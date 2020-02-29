# demo project required by SRE role

### Task 0: Install a ubuntu 16.04 server 64-bit

 * spin up a digitalocean instance with 4 GB Memory / 25 GB Disk / Ubuntu 16.04.6 (LTS) x64
 * upload my SSH public key to the account
 * verify access

```
gyang@gyang-macOS ~ $ ssh root@178.128.50.80
The authenticity of host '178.128.50.80 (178.128.50.80)' can't be established.
ECDSA key fingerprint is SHA256:LBjYOXIN2JlN25gpJaC4QJbinvZ6LfgSkw4OpVx0Lfw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '178.128.50.80' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-169-generic x86_64)


 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


0 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.


Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


root@ubuntu-1604:~#
```

### Task 1: Update system

```
apt update
apt upgrade
```

### Task 2: install gitlab-ce version in the host

 * install dependencies and gitlab packages

```
apt-get install -y curl openssh-server ca-certificates
apt-get install -y postfix
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
apt-get install gitlab-ce
```

 * update `external_url` in `/etc/gitlab/gitlab.rb`
 * restart gitlab after set `LC_ALL` & `LC_CTYPE`

```
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
gitlab-ctl reconfigure
```

### Task 3: create a demo group/project in gitlab 

## use golang to build a hello world web app (listen to 8081 port) and check-in the code to mainline.

 * update `root` password then create *group* `demo` & *project* `go-web-hello-world` in gitlab
 * run `ssh-keygen` to generate SSH key pair and upload the pub key to the gitlab
 * clone the git repo to local

```
root@ubuntu-1604:~# git clone git@127.0.0.1:demo/go-web-hello-world.git
Cloning into 'go-web-hello-world'...
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:LBjYOXIN2JlN25gpJaC4QJbinvZ6LfgSkw4OpVx0Lfw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '127.0.0.1' (ECDSA) to the list of known hosts.
warning: You appear to have cloned an empty repository.
Checking connectivity... done.
```

 * cd to `go-web-hello-world` and mkdir `docker`
 * cd to `docker` and touch `hello-world.go` with the following demo code

```
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Go Web Hello World!\n")
    })

    http.ListenAndServe(":8081", nil)
}
```

 * adding `docker/hello-world.go` to the remote repository

### Task 4: build the app and expose ($ go run) the service to 8081 port

 * install golang

```
add-apt-repository ppa:longsleep/golang-backports
apt update
apt install golang-1.11-go
```

 * build the following demo code and get the expected output

```
root@ubuntu-1604:~# go run hello-world.go

root@ubuntu-1604:~# curl http://127.0.0.1:8081
Go Web Hello World!
```

### Task 5: install docker

```
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
```

### Task 6: run the app in container

## build a docker image ($ docker build) for the web app and run that in a container ($ docker run), expose the service to 8082 (-p)

 * cd to `go-web-hello-world/docker` and touch the `Dockerfile` with following content

```
FROM golang:latest

LABEL maintainer="Gary Yang <ygb_1023@sina.com>"

RUN mkdir /app
ADD hello-world.go /app/
WORKDIR /app
RUN go build -o hello-world hello-world.go

EXPOSE 8081
CMD ["./hello-world"]
```

 * build and start the container

```
docker build . -t go-hello-world
docker run -dt -p 8082:8081 go-hello-world
```

 * test and get the expected output

```
root@ubuntu-1604:~# curl http://127.0.0.1:8082
Go Web Hello World!
```

### Task 7: push image to dockerhub

## tag the docker image using your_dockerhub_id/go-web-hello-world:v0.1 and push it to docker hub (https://hub.docker.com/)

 * sign up the hub.docker.com with dockerhub id *yanggongbi*
 * tag the image

```
docker tag 09c29cfd2002 yanggongbi/go-web-hello-world:v0.1
```

 * docker login and push the image

```
docker login --username yanggongbi --password my_dockerhub_passwd
docker push yanggongbi/go-web-hello-world:v0.1
```
