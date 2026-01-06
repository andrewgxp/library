# Kubectl

## What Is kubectl?

`kubectl` is a command line tool that allows you to interact with Kubernetes. `kubectl` uses the Kubernetes API to communicate with the cluster and carry out your commands.

## Basic Kubectl Commands

### kubectl get

Use `kubectl get` to list objects in the Kubernetes cluster.  

• `-o`  Set output format.  
• `--sort-by`  Sort output using a JSONPath expression.  
• `--selector` Filter results by label.

### kubectl describe

You can get detailed, human-readable information about Kubernetes objects using `kubectl describe`

Example: `kubectl describe <object type> <object name>`

### kubectl create

Use `kubectl create` to create objects. 

Supply a YAML file with -f to create an object from a YAML descriptor stored in the file.

If you attempt to create an existing object, an error will occur.

Example: `kubectl create -f <file name>`

### kubectl apply

`kubectl apply` is similar to `kubectl create`. 

However, if you use `kubectl apply` on an existing object, it will modify the existing object, if possible.

Example: `kubectl apply -f <file name>`

### kubectl delete

Use `kubectl delete` to delete objects from the cluster.

Syntax: `kubectl delete $OBJECT-TYPE $OBJECT-NAME -n $NAMESPACE`

```
cloud_user@k8s-control:~$ kubectl get service --all-namespaces
NAMESPACE       NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
beebox-mobile   beebox-auth-svc   ClusterIP   10.109.202.47   <none>        80/TCP                   22m
default         kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP                  23m
kube-system     kube-dns          ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   23m

cloud_user@k8s-control:~$ kubectl delete service beebox-auth-svc -n beebox-mobile
service "beebox-auth-svc" deleted
```

### kubectl exec

`kubectl exec` can be used to run commands inside containers. 

Syntax: `kubectl exec $POD-NAME -n $NAMESPACE -- $YOUR-COMMAND`

You must know the pod name and namespace to run a command inside a container. 

```
cloud_user@k8s-control:~$ kubectl get pods --all-namespaces
NAMESPACE       NAME                                       READY   STATUS    RESTARTS   AGE
beebox-mobile   quark                                      1/1     Running   0          16m
kube-system     calico-kube-controllers-56cdb7c587-gmc6j   1/1     Running   0          16m
kube-system     calico-node-gr8n9                          1/1     Running   0          16m
kube-system     calico-node-lhkkr                          1/1     Running   0          16m
kube-system     coredns-6d4b75cb6d-4xmhl                   1/1     Running   0          17m
kube-system     coredns-6d4b75cb6d-clqlc                   1/1     Running   0          17m
kube-system     etcd-k8s-control                           1/1     Running   0          17m
kube-system     kube-apiserver-k8s-control                 1/1     Running   0          17m
kube-system     kube-controller-manager-k8s-control        1/1     Running   0          17m
kube-system     kube-proxy-7stl7                           1/1     Running   0          17m
kube-system     kube-proxy-jncvr                           1/1     Running   0          16m
kube-system     kube-scheduler-k8s-control                 1/1     Running   0          17m

cloud_user@k8s-control:~$ kubectl exec quark -n beebox-mobile -- cat /etc/issue
Welcome to Buildroot
```
Remember that for a command to succeed, the necessary software must exist within the container to run it.

## Kubectl Quick Tips

### Declarative vs Imperative Object Management

Kubectl can be used to manage objects either declaratively or imperatively. 

Declarative: Define objects using data structures such as YAML or JSON

Imperative: Define objects using kubectl commands and flags. Some people find imperative commands faster. Experiment and see what works for you!

Example: `kubectl create deployment my-deployment -- image=nginx`

### Creating a quick sample YAML

Use the --dry-run glad to run an imperative command without creating an object. Combine it with -o yaml to quickly obtain a sample YAML file you can manipulate

`kubectl create deployment my-deployment --image=nginx --dry-run -o yaml`

### Record a Command 

You can use the --record flag to record the command that was used to make a change.

`kubectl scale deployment my-deployment replicas=5 --record`

# RBAC in Kubernetes

## What is RBAC

RBAC stands for role-based access control. In Kubernetes, this allows you to control what users are allowed to do and access within your cluster. 

For example, you can use RBAC to allow developers to read metadata and logs from your pods without making changes to your pods. 

## RBAC Objects 

`Roles` and `ClusterRoles` are Kubernetes objects that define a set of permission. These permissions determine what users can do in the cluster. 

A Role defines permissions with a particular namespace.

A ClusterRole defines cluster-wide permissions not specific to a single namespace. 

`RoleBinding` and `ClusterRoleBinding` are objects that connect Roles and ClusterRoles to users. 

A `RoleBinding` binds a `Role` to a subject, but only within a particular namespace.

You can apply RBAC rules by creating and applying a YAML file that defines RBAC as an object. 

Role Sample:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Role-binding Sample: 

```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

# Service Accounts 

## What is a Service Account 

In Kubernetes, a service account is an account used by container processes within Pods to authenticate with the Kubernetes API. 

If applications within the Pods need to communicate with the Kubernetes API, you can use service accounts to control their access. 

## Creating Service Accounts

A service account object can be created with some YAML just like any other Kubernetes object. 

They have their own `kind` value called `ServiceAccount`

Example: 

```
apiVersion: v1 
kind: ServiceAccount
metadata:
  name: my-serviceaccount
```

You can also create a service account using Imperative commands. 

Example: `kubectl create sa my-serviceaccount -n $YOUR-NAMESPACE`

The service account that you've created can easily be listed with `kubectl get sa`

### Binding Roles to Service Accounts

You can manage access control for service accounts just like any other user, using RBAC objects. 

Bind service accounts with RoleBindings or CluseterRoleBindings to provide access to Kubernetes API functionality. 

Example: 

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-pod-reader
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

# Kubernetes Pod Resource Usage

There is an addon called [kubernetes-sigs/metrics-server](https://github.com/kubernetes-sigs/metrics-server) that can be installed on a Kubernetes cluster to provide basic information about CPU and Memory usage for pods.

## Installing metrics-server

The metrics-server can be easily installed by applying a components file provided by the project. This will install all the necessary components that the metrics-server needs as Kubernetes objects within the cluster. 

Example: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

You can verify a proper installation by querying the metrics API server. 

Example: `kubectl get --raw /apis/metrics.k8s.io/`

## kubectl top 

With `kubectl top`, you can view data about resource usage in your pods and nodes. The `kubtctl top` command also supports flags like `--sort-by` and `--selector`. 

Example: `kubectl top pod --namespace $YOUR-NAMESPACE --sort-by $SORT-VALUE --selector $YOUR-SELECTOR`

```
kubectl top pod --namespace beebox-mobile --sort-by CPU --selector app=auth
NAME           CPU(cores)   MEMORY(bytes)
auth-proc      100m         6Mi
beebox-auth1   0m           0Mi
beebox-auth2   0m           0Mi
```

# Pods and Containers 

## Application Configuration 

When you are running applications in Kubernetes, you may want to pass dynamic values to your applications at runtime to control how they behave. This is known as application configuration. 

### ConfigMaps

You can store configuration data in Kubernetes using ConfigMaps. ConfigMaps store data in the form of a key-value map. ConfigMap data can be passed to your container applications. 

Example YAML: 

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: value1
  key2: value2
  key3:
    subkey:
      morekeys: data
      evermore: some more data
  key4: | 
    This is a 
    Multi-line
    Data Value 
```

### Secrets 

Secrets are similar to ConfigMaps but are designed to store sensitive data, such as passwords or API keys, more securely. They are created and used similarly to ConfigMaps. 

Your secrets must be base64 encoded before you can store them in Kubernetes.

To encode a secret, you can use the `base64` command like this: 

```
echo -n "mysecretusername" | base64

echo -n "mysecretpassword" | base64
```

Example YAML:

```
apiVersion: v1
kind: Secret
metadata: 
  name: my-secret
type: Opaque
data: 
  username: bXlzZWNyZXR1c2Vy
  password: bXlzZWNyZXRwYXNz
```

After creating your YAML file containing the secret you can create it with `kubectl create -f $YOUR-SECRET-FILE.yaml`

### Environment Variables 

You can pass ConfigMap and Secret data to your containers as environment variables. These variables will be visible to your container process at runtime. 

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo "configmap: $CONFIGMAPVAR secret: $SECRETVAR"']
    env:
    - name: CONFIGMAPVAR
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: key1
  - name: SECRETVAR
      valueFrom:
         secretKeyRef:
           name: my-secret
           key: username
```

After creating your YAML file containing the pod you can create it with `kubectl create -f $YOUR-POD-FILE.yaml`

To check that this worked, you can run `kubectl logs $YOUR-POD-NAME`

In this example it would be `kubectl logs env-pod`

### Configuration Volumes 

Configuration data from ConfigMaps and Secrets can also be passed to containers in the form of mounted volumes. This will cause the configuration data to appear in files available to the container file system. 

Each top-level key in the configuration data will appear as a file containg all keys below that top-level key. 

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
  volumeMounts:
  - name: configmap-volume
    mountPath: /etc/config/configmap
  - name: secret-volume
    mountPath: /etc/config/secret
  volumes:
  - name: configmap-volume
    configMap:
      name: myconfig-map
  - name: secret-volume
    secret:
      secretName: my-secret
```

After creating your YAML file containing the pod you can create it with `kubectl create -f $YOUR-POD-FILE.yaml`

To show the mounted volume files, you can run `kubectl exec $YOUR-POD-FILE -- ls /etc/config/configmap`

In this example it would be `kubectl exec volume-pod -- ls /etc/config/configmap`

To view the contents of the file, you can run `kubectl exec $YOUR-POD-FILE -- cat /etc/config/configmap/$YOUR-FILE-NAME`

In this example it would be `kubectl exec volume-pod -- cat /etc/config/configmap/key1`

To show the secret mounted volume files, you can run `kubectl exec $YOUR-POD-FILE -- ls /etc/config/secret`

In this example it would be `kubectl exec volume-pod -- ls /etc/config/secret`

To view the secret contents of the file, you can run `kubectl exec $YOUR-POD-FILE -- cat /etc/config/secret/$YOUR-FILE-NAME`

In this example it would be `kubectl exec volume-pod -- cat /etc/config/secret/username`

### Passing Configuration Data to a Kubernetes Container

In this example, I will show you how to pass configuration data that will be used by an Nginx container. 

We'll be doing this from a control plane node.

Since this is Nginx, we'll use the htpasswd utility to create a password file.

```
htpasswd -c .htpasswd user
```

Next, we'll create a secret configuration file using imperative commands. 

```
kubectl create secret generic nginx-htpasswd --from-file=.htpasswd
```

Since the secret is created, we can delete the .htpasswd file for security purposes using a simple `rm .htpasswd`

From here we can create our pod configuration file. 

In this example, I'll call the file pod.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:lastest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/
    - name: htpasswd-volume
      mountPath: /etc/nginx/conf
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
  - name: htpasswd-volume
    secret:
      secretName: nginx-htpasswd
```

We can then apply the configuration file using `kubectl apply -f pod.yml`

You can verify that the pod is running using `kubectl get pods` or `kubectl get pods -o wide`

The output should look similar to this: 

```
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          52m   192.168.194.65   k8s-worker1   <none>           <none>
nginx     1/1     Running   0          61s   192.168.194.67   k8s-worker1   <none>           <none>
```

Now we can use `curl` command from the control plane node to verify that the Nginx server is running correctly. 

Because we set Nginx to require authentication, we'll need to pass the username and password to the curl command. 

```
curl -u user:password http://192.168.194.67
```

If successful, then the output should look similar to this: 

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Managing Container Resources

### Resource Requests

Resource request allow you to define an amount of resource (such as CPU or memory) you expect a container to use. 

The Kubernetes scheduler will use request requests to avoid scheduling pods on nodes that do not have enough available resources.

Containers are allowed to use more or less than the requested amount of resources, but the scheduler will not allow pods to be scheduled on nodes that do not have enough available resources to meet the request.

Here is an example of a pod configuration file that uses the Nginx image and request 0.5 CPU and 256MiB of memory. 

Memory is measured in bytes, so 256MiB is 256 * 1024 * 1024 bytes.

CPU is measured in CPU units, which are 1/1000 of one CPU. This makes 0.5 CPU cores equal 500m CPU units.

When the number of CPU cores or memory is larger than any worker nodes have, then the pod will remain in a pending state until the requested resources are available.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        cpu: "500m"
        memory: "256Mi"
```

To create the pod, you can use `kubectl create -f $YOUR-POD-FILE.yaml`

### Resource Limits

Resource limits provide a way for you to limit the amount of resources your containers can use. 

The container runtime is responsible for enforcing these limits, and different container runtimes may enforce them differently.

Some runtimes will enforce these limits by terminating container processes that exceed the limits, while others will throttle the container processes.

When using Docker as the runtime, the default behavior is to throttle the container processes when the CPU limit is exceeded, and to terminate the container processes when the memory limit is exceeded.

Here is an example of a pod configuration file that uses the Nginx image and limits the container to 0.5 CPU and 256MiB of memory. 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      limits:
        cpu: "500m"
        memory: "256Mi"
```

To create the pod, you can use `kubectl create -f $YOUR-POD-FILE.yaml`

## Monitoring Container Health with Probes 

### Container Health 

Kubernetes provides a number of features that allow you to build robust solutions, such as the ability to automatically restart unhealthy containers.

To make the most of these features, Kubernetes needs to be able to accurately determine the status of the applications within your containers. 

Kubernetes accomplishes this by actively monitoring the health of your containers using probes.

### Liveness Probes

Liveness probes allow you to automatically determine whether or not a container application is in a healthy state. 

By default, Kubernetes will only consider a container to be "down" if the container process stops running.

Liveness probes allow you to customize this detection mechanism and make it more sophisticated.



