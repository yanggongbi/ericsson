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

 * verify https://hub.docker.com/repository/docker/yanggongbi/go-web-hello-world is available

### Task 8: document the procedure in a MarkDown file

### Task 9: install a single node Kubernetes cluster using kubeadm
## Check in the admin.conf file into the gitlab repo

 * ensure legacy binaries are installed

```
apt-get install -y iptables arptables ebtables
```

 * Installing kubeadm, kubelet and kubectl

```
apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

 * Initializing control-plane node

```
export KUBECONFIG=/etc/kubernetes/admin.conf
kubeadm init
```

 * Installing a Pod network add-on

```
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

 * Control plane node isolation

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

 * verify if all the components are ready

```
root@ubuntu-1604:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5b644bc49c-4hggv   1/1     Running   0          10m
kube-system   calico-node-jf6fm                          1/1     Running   0          10m
kube-system   coredns-6955765f44-45jtv                   1/1     Running   0          41m
kube-system   coredns-6955765f44-l7mcw                   1/1     Running   0          41m
kube-system   etcd-ubuntu-1604                           1/1     Running   0          42m
kube-system   kube-apiserver-ubuntu-1604                 1/1     Running   0          42m
kube-system   kube-controller-manager-ubuntu-1604        1/1     Running   0          42m
kube-system   kube-proxy-z5vk5                           1/1     Running   0          41m
kube-system   kube-scheduler-ubuntu-1604                 1/1     Running   0          42m
```

 * mkdir `go-web-hello-world/k8s` and adding `go-web-hello-world/k8s/admin.conf` to the remote repository

### Task 10: deploy the hello world container

## in the kubernetes above and expose the service to nodePort 31080

* touch `go-web-hello-world/k8s/go-web-hello-world-deployment.yaml` with the following content

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-hello-world
  template:
    metadata:
      labels:
        app: go-web-hello-world
    spec:
      containers:
      - image: yanggongbi/go-web-hello-world:v0.1
        name: go-web-hello-world
        ports:
          - containerPort: 8081
```

 * create deployment

```
kubectl apply -f go-web-hello-world-deployment.yaml
```

 * do verification

```
root@ubuntu-1604:~/go-web-hello-world/k8s# kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/go-web-hello-world-5cb7449d9c-xl8d5   1/1     Running   0          14s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   128m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-web-hello-world   1/1     1            1           14s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/go-web-hello-world-5cb7449d9c   1         1         1       15s
```

```
root@ubuntu-1604:~/go-web-hello-world/k8s# kubectl get pods -l app=go-web-hello-world
NAME                                  READY   STATUS    RESTARTS   AGE
go-web-hello-world-5cb7449d9c-xl8d5   1/1     Running   0          2m49s
```

 * touch `go-web-hello-world/k8s/go-web-hello-world-svc.yaml` with the following content

```
apiVersion: v1
kind: Service
metadata:
  name: go-web-hello-world
  labels:
    run: go-web-hello-world
spec:
  type: NodePort
  ports:
    - port: 31080
      protocol: TCP
      targetPort: 8081
  selector:
    app: go-web-hello-world
```

* apply the svc yaml and verify

```
kubectl apply -f go-web-hello-world-svc.yaml
```

```
root@ubuntu-1604:~/go-web-hello-world/k8s# kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/go-web-hello-world-5cb7449d9c-jns9x   1/1     Running   0          117m

NAME                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE
service/go-web-hello-world   NodePort    10.103.30.30   <none>        31080:30275/TCP   83m
service/kubernetes           ClusterIP   10.96.0.1      <none>        443/TCP           118m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-web-hello-world   1/1     1            1           117m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/go-web-hello-world-5cb7449d9c   1         1         1       117m
```

 * tried to use *LoadBalancer* type svc with *EXTERNAL-IP* but no luck with it, so using `port-forward` as a workaround

```
root@ubuntu-1604:~/go-web-hello-world/k8s# kubectl port-forward service/go-web-hello-world 31080:31080 >/dev/null 2>&1 &
[1] 24567
```

 * curl verify

```
root@ubuntu-1604:~# curl 127.0.0.1:31080
Go Web Hello World!
```

## Check in the deployment yaml file or the command line into the gitlab repo

* adding both of `k8s/go-web-hello-world-deployment.yaml` and `k8s/go-web-hello-world-svc.yaml` to the remote repository
