# cavemen-to-kuberenetes
A presentation covering an introduction and demo for Kubernetes.

## Prerequisites

Must have installed:
- [Minikube](https://github.com/kubernetes/minikube)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

To check if `minikube` is installed:

```sh
$ minkube version
minikube version: v0.30.0
```

To check if 'kubectl` is installed:

```sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.7", GitCommit:"dd5e1a2978fd0b97d9b78e1564398aeea7e7fe92", GitTreeState:"clean", BuildDate:"2018-04-19T00:05:56Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

## Minikube
Minikube starts a virtual machine, or node, and installs kubernetes.

Start minikube. This may take a couple of minutes.

```sh
$ minkube start
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

You can view details about the cluster:

```sh
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
CoreDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

You can view the status of the node(s):

```sh
$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    13d       v1.10.0
```

The `kubectl` CLI tool uses the k8s API to interact with the cluster.
We can access the API directly through a proxy.

```sh
 $ kubectl proxy
 Starting to serve on 127.0.0.1:8001
 ```

 Now we can use the API with HTTP requests to interact with our cluster.

 ```sh
 $ curl localhost:8001/version
 {
  "major": "1",
  "minor": "10",
  "gitVersion": "v1.10.0",
  "gitCommit": "fc32d2f3698e36b93322a3465f63a14e9f0eaead",
  "gitTreeState": "clean",
  "buildDate": "2018-03-26T16:44:10Z",
  "goVersion": "go1.9.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```


Another feature is a dashboard available from the API as a web GUI.
While the proxy is runnin, navigate your browser to
<http://localhost:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy>

## Create a Namespace
Namespaces are logical separations within the cluster.
They can be used to divide teams, environments, or tenants.
Resource names are unique within namespaces.
Create a Namespace:

```sh
kubectl create namespace dev
namespace "dev" created
```

You can specify the namespace for any command when executing it. For example:

```sh
kubectl -n dev get pods
```

## Create a Deployment
Deployments are typically used for stateless applications.
Stateful applications can use a resource called 'StatefulSet'.

There are a number of ways to create Kuberenetes resources.
One way is by declaring the specifications in a resource manifest.
The deployment has been pre-defined in a Kubernetes manifest at
<./kubernetes/hello-k8s-deploy.yml>.
To create it:

```sh
$ kubectl apply -f ./kubernetes/hello-k8s-deploy.yml
deployment "hello-k8s-deploy" created
```

The Deployment will manage the state and replicas of our pods.
Deployments also allow us to perform rollouts and rollbacks.
We can view the status of Deployments with:

```sh
$ kubectl get deploy
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-k8s-deploy   1         1         1            1           7m
```

We can get more detailed information with:
```sh
$ kubectl describe deploy
Name:                   hello-k8s-deploy
Namespace:              default
CreationTimestamp:      Fri, 26 Oct 2018 02:19:03 -0500
Labels:                 app=hello-k8s-deploy
...
```

You could also pass this command the specific name of a Deployment:

```sh
$  kubectl describe deploy hello-k8s-deploy
Name:                   hello-k8s-deploy
Namespace:              default
CreationTimestamp:      Fri, 26 Oct 2018 02:19:03 -0500
Labels:                 app=hello-k8s-deploy
...
```

Sometimes you may see the use of a forward slash, but it means the same thing:

sh```
$ kubectl describe deploy hello-k8s-deploy
Name:                   hello-k8s-deploy
Namespace:              default
CreationTimestamp:      Fri, 26 Oct 2018 02:19:03 -0500
Labels:                 app=hello-k8s-deploy
...

There are similar commands for every resource, including Pods:

```sh
$ ks get pods
NAME                               READY     STATUS    RESTARTS   AGE
hello-k8s-deploy-87cf7f884-c9r76   1/1       Running   0          12m
```

Again, you can also try:

```sh
kubectl get pods <POD-NAME>
```

We can view the logs of a Pod:

```sh
kubectl logs -f <POD-NAME>
```

And also execute commands or get shell access:

```sh
kubectl exec <POD-NAME> /bin/bash
```

## Exposing our Deployment
We expose Pods with with abstract logical groupings called Services.
A Service delivers traffic to Pods with labels matching its selectors.

```sh
kubectl apply -f ./kubernetes/hello-k8s-svc.yml
```

There are three types of Services:
1. ClusterIP - Creates IP address within the cluster (Pod - Pod). Cannot access
   externally.
2. NodePort - Creates a static port on EVERY node in the cluster. Can acccess at
   <http://NODE-IP:NODE-PORT> for any node.
3. LoadBalancer - Will assign an external IP address to a LoadBalancer if one is
   available.

For types NodePort and LoadBalancer, `minikube` has a nice command to open a
browser to the service.

```sh
$ minikube service hello-k8s-svc
Opening kubernetes service default/hello-k8s-svc in default browser...
```

If you know the name of a Pod, you can always access it using port-forwarding
with the `kubectl` CLI tool:

```sh
$ kubectl port-forward POD-NAME HOST-PORT:TARGET-PORT
Forwarding from 127.0.0.1:HOST-PORT -> TARGET-PORT
```

## Next
- Scaling deployments
- Rolling back deployments
- StatefulSets
- Persistent Storage (static and dynamic)
- Ingress Resources and Controllers
- Nice tools
    - [kubectx](https://github.com/ahmetb/kubectx)
    - [kube-ps1](https://github.com/jonmosco/kube-ps1)
    - [stern](https://github.com/wercker/stern)
- Resource requests/limits
- Readiness/liveness probes
- Node Autoscaling
- [Helm](https://github.com/helm/helm)

This demo used resources from:
- https://kubernetes.io/docs/tutorials/kubernetes-basics/
- https://github.com/paulbouwer/hello-kubernetes
