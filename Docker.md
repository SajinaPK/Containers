Docker originally used **LinuX Containers** ([LXC](https://linuxcontainers.org/lxc/)), but later switched to **runC** (formerly known as **libcontainer**), which runs in the same operating system as its host. This allows it to share a lot of the host operating system resources. Also, it uses a layered filesystem (**AuFS**) and manages networking.

**Containers** are just a process (or a group of processes) running in isolation, which is achieved with Linux namespaces and control groups.   
Linux namespaces and control groups are features that are built into the Linux kernel. Other than the Linux kernel itself, there is nothing special about containers.  

What makes containers useful is the tooling that surrounds them. **Docker** has been the understood standard tool for using containers to build applications. Docker provides developers and operators with a friendly interface to build, ship, and run containers on any environment.

Docker store: https://hub.docker.com/_/mongo  
Docker Hub: https://hub.docker.com/search?q=&type=image  

LinuxKit is essentially a container-native toolkit that allows organizations to build their own containerized operating systems that are secure, lean, modular and portable. It makes it easy to create a new Linux OS using LinuxKit and a YAML file in a matter of seconds or minutes.

A **docker image** is the blueprint for spinning up containers. An image is a tar of a file system, and a container is a file system plus a set of processes running in isolation.
Filesystem+metadata  
For sharing and distribution  

**Docker registry**  
Push and pull images from registry  
Default registry: Docker Hub (public and free for public images, many pre-packaged images available)  
Private registry: self host or cloud provider options available.  

Creating a docker image - with a docker build  
Create a docker file - list of instruction on how to construct the container  
`docker build -f dockerfile`  
```
cat Dockerfile
FROM ubuntu
ADD myapp/
EXPOSE 80
ENTRYPOINT /myapp
```
Each of these lines is a layer. Each layer contains only the delta, or changes from the layers before it. To put these layers together into a single running container, Docker uses the union file system to overlay layers transparently into a single view.  

Each layer of the image is read-only except for the top layer, which is created for the container. The **read/write container layer** implements "copy-on-write," which means that files that are stored in lower image layers are pulled up to the read/write container layer only when edits are being made to those files. Those changes are then stored in the container layer.  

The **"copy-on-write"** function is very fast and in almost all cases, does not have a noticeable effect on performance. You can inspect which files have been pulled up to the container level with the docker diff command. 
<img width="425" alt="Screenshot 2024-01-03 at 3 50 44 PM" src="https://github.com/SajinaPK/Containers/assets/42661832/7ebf031e-6e92-4e12-96fa-0b903297ddde">  
Because image layers are read-only, they can be shared by images and by running containers. For example, creating a new Python application with its own Dockerfile with similar base layers will share all the layers that it had in common with the first Python application.  
```
FROM python:3.6.1-alpine
RUN pip install flask
CMD ["python","app2.py"]
COPY app2.py /app2.py
```
<img width="472" alt="Screenshot 2024-01-03 at 3 53 41 PM" src="https://github.com/SajinaPK/Containers/assets/42661832/aeea6daa-03dc-40a1-9983-ffba53a73e23">  

**Docker image layers** 
- Union File system - Merge image layers into single file system for each container  
- Copy-on-write - Copies files that are edited up to top writable layer

**Container Orchestration Solutions** (Kubernetes, Docker Swarm. Apache Mesos) help with the below:  
- Automated scheduling and scaling
- Service discovery
- Zero downtime depoyments
- High availability and fault tolerance
- A/B deployments
- Implementing reconciliation
- Logging

To run containers on a **Docker Swarm**, you need to create a service. A **service** is an abstraction that represents multiple containers of the same image deployed across a distributed cluster. A **task** is another abstraction in Docker Swarm that represents the running instances of a service  

**Limits of the routing mesh**: The routing mesh can publish only one service on port 80. If you want multiple services exposed on port 80, you can use an external application load balancer outside of the swarm to accomplish this.  

Although you control the swarm directly from the node in which its running, you can control a Docker swarm remotely by connecting to the Docker Engine of the manager by using the remote API or by activating a remote host from your local Docker installation (using the **$DOCKER_HOST and $DOCKER_CERT_PATH** environment variables). This will become useful when you want to remotely control production applications, instead of using SSH to directly control production servers.  

- Docker Swarm schedules services by using a **declarative language**. You declare the state, and the swarm attempts to maintain and reconcile to make sure the actual state equals the desired state.
- Docker Swarm is composed of **manager and worker nodes**. Only managers can maintain the state of the swarm and accept commands to modify it. Workers have high scalability and are only used to run containers. By default, managers can also run containers.
- The **routing mesh** built into Docker Swarm means that any port that is published at the service level will be exposed on every node in the swarm. Requests to a published service port will be automatically routed to a container of the service that is running in the swarm.

You should have at least three manager nodes but typically no more than seven. Manager nodes implement the **raft consensus algorithm**, which requires that more than 50% of the nodes agree on the state that is being stored for the cluster. If you don't achieve more than 50% agreement, the swarm will cease to operate correctly. For this reason, note the following guidance for node failure tolerance:  

- Three manager nodes tolerate one node failure.
- Five manager nodes tolerate two node failures.
- Seven manager nodes tolerate three node failures.

It is possible to have an even number of manager nodes, but it adds no value in terms of the number of node failures. For example, four manager nodes will tolerate only one node failure, which is the same tolerance as a three-manager node cluster. However, the more manager nodes you have, the **harder it is to achieve a consensus on the state of a cluster**.  

While you typically want to limit the number of manager nodes to no more than seven, you can scale the number of worker nodes much higher than that. Worker nodes can scale up into the thousands of nodes. Worker nodes communicate by using the **gossip protocol**, which is optimized to be perform well under a lot of traffic and a large number of nodes.  
<img width="299" alt="Screenshot 2024-01-03 at 3 56 27 PM" src="https://github.com/SajinaPK/Containers/assets/42661832/5d28bbfd-6ce7-4998-a69a-b4cbd2455b88">  
*Container Runtime Interface (CRI) spec was introduced in K8s 1.5. CRI also consists of protocol buffers, gRPC API and libraries. This brought the abstraction layer, and acted as an adapter, with the help of gRPC client running in kubelet and gRPC server running in CRI Shim. This allowed a simpler way to run the various container runtimes.*  

<img width="299" alt="Screenshot 2024-01-03 at 3 58 54 PM" src="https://github.com/SajinaPK/Containers/assets/42661832/cb7621da-4036-490b-b062-73d9dce50b5c">  

*So the OCI (Open Container Initiative), came up with a clear container runtime & image specification, which helped multi-platform support (Linux, Windows, VMs etc). Runc is the default implementation of OCI, and that is the low level, of container runtime. The modern container runtimes are built on this layered architecture, where Kubelets talk to Container Runtimes through CRI-gRPC and the Container Runtimes run the containers through OCI.* 
*There are various implementations of CRI such as Dockershim, CRI-O, containerD.*  
![0barzTkbPnPpGBc85](https://github.com/SajinaPK/Containers/assets/42661832/d0e67d75-8b6a-4073-a3ee-4382b995cc02)  

————————————————————————————————————  
			**Lab**  
———————————————————————————————————  

Create file named Dockerfile:
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return “hello World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```
`docker image build -t python-hello-world .`  
*Successfully built 8776553b8745  
Successfully tagged python-hello-world:latest*

`docker image ls`  
*REPOSITORY           TAG             IMAGE ID       CREATED              SIZE  
python-hello-world   latest          8776553b8745   About a minute ago   59.5MB  
python               3.10.5-alpine   27edb73bd1fc   2 weeks ago          47.6MB*  

`docker run -p 5001:5000 -d python-hello-world`  
*bf4a59fb70abee28d4f8f9be13201ed1c03ec4798c06aa076efce8840f5a78e4*  

`docker container ls`   
CONTAINER ID&nbsp;&nbsp;&nbsp;&nbsp;IMAGE&nbsp;&nbsp;&nbsp;&nbsp;COMMAND&nbsp;&nbsp;&nbsp;&nbsp;CREATED&nbsp;&nbsp;&nbsp;&nbsp;STATUS &nbsp;&nbsp;&nbsp;&nbsp; PORTS &nbsp;&nbsp;&nbsp;&nbsp;NAMES  
bf4a59fb70ab   python-hello-world   "python app.py"   About a minute ago   Up About a minute   0.0.0.0:5001->5000/tcp   vigorous_carson

`docker container logs bf4`  

`docker login`  
*Username: sajinapk
Password:*  

`docker tag python-hello-world sajinapk/python-hello-world`  

`docker push sajinapk/python-hello-world`  
*latest: digest: sha256:7898a5e349a4b3d6fe11c306ee7b34c569bd3e69e4ea13e511ef2a2e1e4b2823 size: 1786*  

Edit app.py    

`docker image build -t sajinapk/python-hello-world .`   

*Sending build context to Docker daemon  88.06kB  
Step 1/4 : FROM python:3.10.5-alpine  
 ---> 27edb73bd1fc  
Step 2/4 : RUN pip install flask  
 ---> **Using cache**  
 ---> 2a9b3d338c78  
Step 3/4 : CMD ["python","app.py"]  
 ---> **Using cache**  
 ---> 2c064a44aa39  
Step 4/4 : COPY app.py /app.py  
 ---> f6d4c4746f23  
Successfully built f6d4c4746f23  
Successfully tagged sajinapk/python-hello-world:latest*  

`docker push sajinapk/python-hello-world`  
*Using default tag: latest  
The push refers to repository [docker.io/sajinapk/python-hello-world]  
fccbef2e9d82: Pushed   
0adfff32bcdb: Layer already exists   
87652a1ad873: Layer already exists   
9ad237c539b1: Layer already exists   
24a6c9301506: Layer already exists   
09c126bb3acd: Layer already exists   
24302eb7d908: Layer already exists   
latest: digest: sha256:036a2cd2c9653faab4391c72a32ed95f8d8652aef764116d6f783bfd31bb731d size: 1786*  

`docker image history python-hello-world`  

IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT  
8776553b8745   24 minutes ago   /bin/sh -c #(nop) COPY file:55682d4cbdac8ede…   159B        
2c064a44aa39   24 minutes ago   /bin/sh -c #(nop)  CMD ["python" "app.py"]      0B          
2a9b3d338c78   24 minutes ago   /bin/sh -c pip install flask                    11.9MB      
27edb73bd1fc   2 weeks ago      /bin/sh -c #(nop)  CMD ["python3"]              0B          
<missing>      2 weeks ago      /bin/sh -c set -eux;   wget -O get-pip.py "$…   10.2MB      
<missing>      2 weeks ago      /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_SHA256…   0B          
<missing>      2 weeks ago      /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_URL=ht…   0B          
<missing>      2 weeks ago      /bin/sh -c #(nop)  ENV PYTHON_SETUPTOOLS_VER…   0B          
<missing>      2 weeks ago      /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=22…   0B          
<missing>      2 weeks ago      /bin/sh -c set -eux;  for src in idle3 pydoc…   32B         
<missing>      2 weeks ago      /bin/sh -c set -eux;   apk add --no-cache --…   30.2MB      
<missing>      2 weeks ago      /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.10.5    0B          
<missing>      4 weeks ago      /bin/sh -c #(nop)  ENV GPG_KEY=A035C8C19219B…   0B          
<missing>      4 weeks ago      /bin/sh -c set -eux;  apk add --no-cache   c…   1.8MB       
<missing>      4 weeks ago      /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B          
<missing>      4 weeks ago      /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:/…   0B          
<missing>      4 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B          
<missing>      4 weeks ago      /bin/sh -c #(nop) ADD file:8e81116368669ed3d…   5.53MB  

**Lab: https://labs.play-with-docker.com/**  

`docker swarm init --advertise-addr eth0`  
*Swarm initialized: current node (mne6t2ewp2joakjxxnps7y85s) is now a manager.  
To add a worker to this swarm, run the following command:  
    docker swarm join --token SWMTKN-1-4cdwul33c9dkjeqz6aumyyd910pa6pjy0u9tqgy8w5z3w7k48u-dcoyqyziigk19tr91uh4y1u9m 192.168.0.13:2377  
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.*  

`$ docker swarm join --token SWMTKN-1-4cdwul33c9dkjeqz6aumyyd910pa6pjy0u9tqgy8w5z3w7k48u-dcoyqyziigk19tr91uh4y1u9m 192.168.0.13:2377`
*This node joined a swarm as a worker.*  

`docker node ls`  
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION  
mne6t2ewp2joakjxxnps7y85s *   node1      Ready     Active         Leader           20.10.0  
g6mfcuxb1kdc8toonp5lglml4     node2      Ready     Active                          20.10.0  
ipte6l9k89w9ceod5gk7j3e0t     node3      Ready     Active                          20.10.0  

`$ docker node ls`  
*Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.*  

`docker service create --detach=true --name nginx1 --publish 80:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12`  
*5zhs0jzwe886novgi8lknliy6*  

A task is another abstraction in Docker Swarm that represents the running instances of a service. In this case, there is a 1-1 mapping between a task and a container.  
`$ curl localhost:80`  
*node1*  
Curling will output the hostname where the container is running. For this example, it is running on node1, but yours might be different.  

`docker service update --replicas=5 --detach=true nginx1`  
*nginx1*  

- The state of the service is updated to 5 replicas, which is stored in the swarm's internal storage.  
- Docker Swarm recognizes that the number of replicas that is scheduled now does not match the declared state of 5.  
- Docker Swarm schedules 5 more tasks (containers) in an attempt to meet the declared state for the service.  

`$ curl localhost:80`  
*node3*  

`$ curl localhost:80` 
*node2*  

`docker service logs nginx1`  
*nginx1.3.bihw4j0aphxp@node3    | 10.0.0.2 - - [24/Jun/2022:21:15:54 +0000] "GET / HTTP/1.1" 200 6 "-" "curl/7.69.1" "-"  
nginx1.3.bihw4j0aphxp@node3    | 10.0.0.2 - - [24/Jun/2022:21:15:58 +0000] "GET / HTTP/1.1" 200 6 "-" "curl/7.69.1" "-"  
nginx1.3.bihw4j0aphxp@node3    | 10.0.0.2 - - [24/Jun/2022:21:16:01 +0000] "GET / HTTP/1.1" 200 6 "-" "curl/7.69.1" "-"  
nginx1.3.bihw4j0aphxp@node3    | 10.0.0.2 - - [24/Jun/2022:21:16:36 +0000] "GET / HTTP/1.1" 200 6 "-" "curl/7.69.1" "-"  
nginx1.3.bihw4j0aphxp@node3    | 10.0.0.2 - - [24/Jun/2022:21:16:45 +0000] "GET / HTTP/1.1" 200 6 "-" "curl/7.69.1" "-"*  

`docker service update --image nginx:1.13 --detach=true nginx1`  
*nginx1*  

--update-parallelism: specifies the number of containers to update immediately (defaults to 1).  
--update-delay: specifies the delay between finishing updating a set of containers before moving on to the next set.  

`docker service ps nginx1`  
ID             NAME           IMAGE        NODE      DESIRED STATE   CURRENT STATE                 ERROR     PORTS  
myjgn3msx0dz   nginx1.1       nginx:1.13   node1     Running         Running about a minute ago                
uwb10paim0q9    \_ nginx1.1   nginx:1.12   node1     Shutdown        Shutdown about a minute ago               
epa1v86ue4sj   nginx1.2       nginx:1.13   node3     Running         Running about a minute ago                
p3yqzxads0ww    \_ nginx1.2   nginx:1.12   node3     Shutdown        Shutdown about a minute ago               
d88a4400gbu1   nginx1.3       nginx:1.13   node2     Running         Running about a minute ago                
bihw4j0aphxp    \_ nginx1.3   nginx:1.12   node3     Shutdown        Shutdown about a minute ago               
zcfldztrqmg6   nginx1.4       nginx:1.13   node3     Running         Running about a minute ago                
e9gx7bckly8c    \_ nginx1.4   nginx:1.12   node1     Shutdown        Shutdown about a minute ago               
03rkh8uq2vpg   nginx1.5       nginx:1.13   node2     Running         Running about a minute ago                
lc2lw8n9mqq0    \_ nginx1.5   nginx:1.12   node2     Shutdown        Shutdown about a minute ago   

`docker service create --detach=true --name nginx2 --replicas=5 --publish 81:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12`  
*qn0q35od3ck61hp7jac0lgp99*  

`watch -n 1 docker service ps nginx2`  
*Every 1.0s: docker service ps nginx2          2022-06-24 21:23:25 *   

ID             NAME       IMAGE        NODE      DESIRED STATE   CURRENT STATE                ERROR     PORTS  
dhgbs4tj2roc   nginx2.1   nginx:1.12   node1     Running         Running about a minute ago               
hyzzn5u5fytv   nginx2.2   nginx:1.12   node3     Running         Running about a minute ago               
l3wnloktcsaz   nginx2.3   nginx:1.12   node1     Running         Running about a minute ago               
bdat6dsn5iv7   nginx2.4   nginx:1.12   node3     Running         Running about a minute ago               
y2zrhvnc7cu2   nginx2.5   nginx:1.12   node2     Running         Running about a minute ago   

On node 3:  
`$ docker swarm leave`  
Node left the swarm.  

On Node 1:    
*Every 1.0s: docker service ps nginx2         2022-06-24 21:24:20*  

ID             NAME           IMAGE        NODE      DESIRED STATE   CURRENT STATE           ERROR     PORTS  
dhgbs4tj2roc   nginx2.1       nginx:1.12   node1     Running         Running 2 minutes ago               
myp6y3f5u4ba   nginx2.2       nginx:1.12   node2     Running         Running 1 second ago                
hyzzn5u5fytv    \_ nginx2.2   nginx:1.12   node3     Shutdown        Running 2 minutes ago               
l3wnloktcsaz   nginx2.3       nginx:1.12   node1     Running         Running 2 minutes ago               
k7fo9f8lf3gy   nginx2.4       nginx:1.12   node1     Running         Running 1 second ago                
bdat6dsn5iv7    \_ nginx2.4   nginx:1.12   node3     Shutdown        Running 2 minutes ago               
y2zrhvnc7cu2   nginx2.5       nginx:1.12   node2     Running         Running 2 minutes ago    
