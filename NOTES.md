# Notes on k8s

Just a dump of cool stuff I learned from [the book](https://www.manning.com/books/kubernetes-in-action-second-edition).

## Debugging

* Connecting to the pod from the worker nodes
When running a single pod (without a deployment / service), you can make requests to a pod from one of the nodes.

| Provider 	|              Command             	|
|----------	|:--------------------------------:	|
| minkube  	|           minikube ssh           	|
| kind     	| docker exec -it <node_name> bash 	|
| gcloud   	|        gcloud compute ssh        	|

When inside one of the node, you can then run a curl command.

```sh
curl <pod_ip_address>:<pod_port>
curl 10.244.1.3:8080
```

* Connecting from a one-off client pod
Another way to test the connectivity of your application is to run `curl` in another pod that you create specifically for this task. You use this method to test if other pods will be able to access your pod.

```sh
kubectl run --image=curlimages/curl -it --restart=Never --rm client-pod <command_to_execute>
kubectl run --image=curlimages/curl -it --restart=Never --rm client-pod curl <pod_ip_address>:<pod_port>
kubectl run --image=curlimages/curl -it --restart=Never --rm client-pod curl 10.244.1.3:8080
```

* Connecting to pods via kubectl port forwarding

This is the easiest way to talk to applications running in a pod. The command allows you to communicate with a specific pod through a proxy bound to a network port on your local computer.

```sh
kubectl port-forward kiada 8080
```

## Logs

You can view logs by using the command:

```sh
kubectl logs <pod_name>
```

Stream them with the command:

```sh
kubectl logs -f <pod_name>
```

View their timestamps with the command:

```sh
kubectl logs <pod_name> --timestamps
```

View recent logs (for example in the last 2 minutes) with the command:

```sh
kubectl logs <pod_name> --since=2m
kubectl logs <pod_name> --since-time=2020-02-01T09:50:00Z
```

Constraint the amount of lines of logs to display with the command:

```sh
kubectl logs <pod_name> --tail=10
```

## Accessing files in containers

Some applications store their logs in a file instead of the `stdout` or `stderr`, you can access them with kubernetes in two ways:

* Copying the file from the running container into your local machine

```sh
kubectl cp kiada:html/index.html /tmp/index.html
```

The above comand copies the file `/html/index.html` from the pod named `kiada`  to the `/tmp/index.html` file on your computer.
You can edit this file, and then copy it back into the container with the same command but switching the source and destination:

```sh
kubectl cp /tmp/index.html kiada:html/index.html
```

The `kubectl cp` command requires the `tar` binary to be present in your container.

* Executing commands in running containers

You can invoke a command in a running container with `kubectl`.

```sh
kubectl exec kiada -- ps aux
kubectl exec kiada -- bash
```

### Ephemeral containers

To keep images small and improve security in the container, most containers used in production don't contain any binary files other than those required for the containers primary process. This significantly reduces the attack surfae, but also means that you can't run shells or other tools in production containers. Fortunately, a new Kubernetes feature called [`Ephemeral Containers`](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) allows you to debug running containers by attaching a debug container to them.

### Attaching to the Application's Standard input

You can use the `kubectl attach -i <pod_name>` command to interact with the container's standard input. We added this line in our nodejs project to handle this in code:

```js
let rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
    terminal: false
});

rl.on('line', function (line) {
    if (line.trim() === "") {
        console.error("Enter new status message and press ENTER.");
    } else {
        statusMessage = line;
        console.log("Status message set to: " + line);
    }
});
```

### Multiple containers in a pod

When running multiple containers in a pod, you sometimes want to **Portforward across multiple ports**. The command to do that is listed below:

```sh
kubectl port-forward kiada 8080 8443 9901
```

To get the logs of a specific container in the pod, you'll use the container:

```sh
kubectl logs kiada -c envoy
```

### Init containers

A pod manifest can specify a list of containers to run when the pod starts and before the pod's normal containers are started. These containers are intended to initialize the pod and are appropriately called `init containers`. Init containers run one after the other and must all finish successfully before the main containers of the pod are started.

They're like the pod's regular containers, but they don't run in parallel - only one init container runs at a time.
