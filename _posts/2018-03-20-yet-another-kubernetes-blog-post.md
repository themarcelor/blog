---
layout: post
author: Marcelo Costa
---
Automating deployment, scaling, and management of containerized applications

What is Kubernetes? Here’s the description from the official doc:

“Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.”

So, let me take a step at describing it in a more verbose way (touching some low-level details):

It’s a Golang-based solution that interacts with an underlying container engine (usually, Docker) to spin up groups of containers, known as “pods”. This is done through agents (kubelets) deployed on multiple servers or VMs (nodes) that are managed through a centralized Kube Controller (i.e. `​kube-apiserver`). The state of the Kubernetes “Cluster” is stored on “etcd” (distributed key/value database) and the communication between pods, containers and nodes is performed through a virtual SDN (Software Defined Network).

Now, as always, diagrams for visual learners (with some AWS details we will talk about later in another blog post):

cacoo_kube_aws

Also known as “k8s”, it offers many features, such as:

Processing the heartbeat from kubelets along with a Quality Of Service (qos) management module that helps the control plane with its heuristics while scheduling pods against a given set of nodes (i.e., the Kubernetes scheduling process evaluates which nodes have “room” to accommodate new pods and it calculates CPU and Memory accordingly before starting any set of Docker containers). Also, through the instrumentation of the underlying Docker daemon, the Kubelet also knows when pods get “evicted” if, for whatever reason, a container crashes (it will also perform other operations depending on the Public Cloud provider that is serving the underlying infrastructure, e.g., While operating on AWS, it describes ec2 instances to process metadata information, etc.).
Working with a virtual network layer that facilitates the communication with pods and their respective containers. This is commonly achieved through the combination of technologies like Calico (Network Policies. Allows/Blocks communication between pods) and Flannel (Overlay network that encapsulates packages during the communication between nodes and their containers). Here is a diagram that summarizes this:
flannel

It provides mechanisms to easily deploy dockerized applications and/or micro-services with its respective configuration (ConfigMaps), sensitive data (secrets), storage requirements (Persistent Volumes), replication & load balancing (Replica Sets & Services) and, last but not least, nice CLI capabilities to scale the number of replicas up and down, including ZDT (Zero Down Time) deployment through rolling upgrades.
So, as you can see, k8s just rocks!

To illustrate a more tangible example of Kubernetes’ powers, I’m sharing below all the artifacts required for a small Proof Of Concept that involves Kubernetes. This is based on a project I’ve worked on some time ago, the source files can be found in Github repos:

https://github.com/themarcelor/MyWebSocketsApp
https://github.com/themarcelor/NginxTLSTerminationInK8SPOC
The POC was conducted with:

Minikube. This is a tiny Kubernetes engine running within a single VirtualBox VM. This is all you need to start playing with Kubernetes running your own experiments.
Custom Docker images (created based on `centos:7` and `tomcat` images) hosted in my personal DockerHub space (e.g., https://github.com/themarcelor/NginxTLSTerminationInK8SPOC).
Self Signed certificates (https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7).
The objective was to introduce a side-car container to the main User Interface (UI) pod, whose container hosts the front-end layer of the overall system, and perform TLS termination (i.e., take the inbound encrypted HTTPS request and forward it to the underlying back-end app in unencrypted HTTP format). To make things more interesting, there was also an extra requirement to reinforce Web Sockets support.

The idea was to load the “index.html” page hosted in the UI app and let the JS try to establish the Web Sockets communication through the WSS protocol:

var ws = new WebSocket("wss://"+window.location.host+"/mywebsocketsapp/echo");
The HTTPS communication hops through the Kubernetes overlay network and its “Service” forwards the request to the target pod. The entry point is the Nginx “side-car” container that mounts the k8s secrets containing the self-signed certificates required to initiate and terminate the TLS communication. The HTTPS request is then converted to HTTP and it finally arrives on the UI container (containing a Tomcat-hosted Java Web App).

If everything is correctly assembled, the following flow is reproduced:

poc_flow.png

 

To sum it up: Kubernetes will definitely make your life easier if you are trying to deploy a cloud-oriented solution. There are many gotchas and occult tips & tricks, but I will have to find some time to write about them. Hopefully in the next blog post.

Cheers!
