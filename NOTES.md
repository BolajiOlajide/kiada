# Notes on k8s

Just a dump of cool stuff I learned from [the book](https://www.manning.com/books/kubernetes-in-action-second-edition).

### Debugging

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

Another way to test the connectivity of your application is to run `curl` in another pod that you create specifically for this task. You use this method to test if other pods will be able to access your pod.

```sh
kubectl run --image=curlimages/curl -it --restart=Never --rm client-pod curl <pod_ip_address>:<pod_port>
kubectl run --image=curlimages/curl -it --restart=Never --rm client-pod curl 10.244.1.3:8080
```
