## Overview

A short lab used to get people started with minikube.

The goals are

 * Get comfortable creating objects with `kubectl apply`

 * Get comfortable using `kubectl get` for pod and service types

 * Learn that `kubectl describe` provides more information than `kubectl get`

 * Be able to get logs from a container via `kubectl logs`

## Installation

These installation instructions are specific to mac users and a subset of what is detailed in the MiniKube documentation https://github.com/kubernetes/minikube.
Instructions for installing on windows and linux are available in the MiniKube documentation.

To use MiniKube you need MiniKube itself, kubectl (the kubernetes cli) and a virtualization driver.

```
brew cask install minikube
brew install kubernetes-cli
brew install docker-machine-driver-xhyve
```

## Start MiniKube

Now let's make sure we can get MiniKube started by running the following.  If you are using a different vm-driver you'll need to change the flag to match your installation.

```
$minikube start --vm-driver=xhyve                                                                                                                                                                                                                       ⌆ 1   master 
Starting local Kubernetes v1.6.0 cluster...
Starting VM...
Downloading Minikube ISO
 89.51 MB / 89.51 MB [==============================================] 100.00% 0s
 SSH-ing files into VM...
 Setting up certs...
 Starting cluster components...
 Connecting to cluster...
 Setting up kubeconfig...
 Kubectl is now configured to use the cluster.
```

Verify everything is ok.

```
$minikube status                                                                                                                                                                                                                                        ⌆ 1   master 
minikubeVM: Running
localkube: Running
```

```
$kubectl cluster-info                                                                                                                                                                                                                                   ⌆ 1   master 
Kubernetes master is running at https://192.168.64.4:8443
```

## Hello MiniKube
Let's run the first example from the documentation to make sure that you can access a pod when it's been deployed to MiniKube

Create the deployment

```
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
```


Verify a pod has been created
```
kubectl get pod
```

Create the service

```
kubectl expose deployment hello-minikube --type=NodePort
```

Verify the svc exists

```
kubectl get svc
```

Send an http request to the service which will pass the request to the pod
```
curl $(minikube service hello-minikube --url)
```

Launch the UI to make it easier to visualize the cluster and the objects we created above.

```
$minikube dashboard
Opening kubernetes dashboard in default browser...
```

Cleanup the hello minikube objects

```
kubectl delete deployment hello-minikube
kubectl delete svc hello-minikube
```

## Pod
The basic building block in kubernetes is a pod.  Let's create a pod and use kubectl to inspect the pod.

```
kubectl create -f pod-nginx.yaml
```

Check the state of the pod

```
kubectl  get pod
```

Describe the pod object

```
kubectl describe pod nginx
```

Delete the pod

You can do either of the following

```
kubectl delete pod nginx
```

or

```
kubectl delete -f pod-nginx.yaml
```

## Deployment
A deployment is a higher level abstraction within kubernetes that describes the pods you want to deploy and the count of them.  Normally you will find yourself working with deployments instead of pods.  For example if you wanted to have 3 pods running nginx you would create 1 deployment which contains the pod information and specify that you want 3 replicas.

Apply the deployment

```
kubectl apply -f deployment-nginx.yaml
```

Get the pod list

```
kubectl get pod
```

Get the deployment list

```
kubectl get deployment
```

Delete one of the pods from the output of `kubectl get pod`

```
kubectl delete pod <POD_NAME>
for example
kubectl delete pod nginx-240716091-2sk6g
````

Get the pod list again ( you should know how to do this now )

Delete the deployment

You can either do

```
kubectl delete deployment nginx
```

or

```
kubectl delete -f deployment-nginx.yaml
```

## Service
Because pods can be very short lived we need some way to present a persistent name or ipaddress that can be used even when the pods are changing.  A service is a way to map requests for a given service to one of the pods running the application.  In this case we will use our nginx as our appliation that we want to expose.

Create the service

```
kubectl create -f service-nginx.yaml
```

Get the svc

```
kubectl get svc
```

Describe the service to see how many pods are available to fill requests.  You are looking for the `Endpoints` line.  No endpoints should be available now.

```
kubectl describe svc nginx
```

Let's create the nginx deployment again ( you can do it! )

Describe the nginx service again.  You may need to do this a few times until the `Endpoints` line lists some available pods.

Now send a request to the service

```
curl $(minikube service nginx --url)
```

Let's find which pod got the request.  Get a list of the nginx pods and for each one run the following

```
kubectl logs <POD_NAME>
```

You should see something like the following from the pod that got the request

```
172.17.0.1 - - [24/May/2017:21:39:05 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.43.0" "-"
```

## What now?

I would suggest going through the excellent guest book example which will go deeper into important concepts like `label` and `selector` https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook

## Other Resources

### Docker Specific

http://training.play-with-docker.com/ Has a list of self paced (free) courses.

I would recomend these courses
http://training.play-with-docker.com/helloworld/
http://training.play-with-docker.com/docker-images/
http://training.play-with-docker.com/docker-containers/

### Kubernetes Specific
https://kubernetes.io/docs/tutorials/
https://kubernetes.io/docs/concepts/
