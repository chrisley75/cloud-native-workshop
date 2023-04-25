#### Prerequisites

Install Docker 

- Have a K8s cluster (provide eks cluster)

#### Creating a Deployment config

For example, a Deployment, which is a type of Pod, would look like this example from the documentation:



- Create a file deployment.yaml as following

```yaml
# hello-kubernetes.custom-message.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes-custom
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-kubernetes-custom
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes-custom
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-kubernetes-custom
  template:
    metadata:
      labels:
        app: hello-kubernetes-custom
    spec:
      containers:
      - name: hello-kubernetes
        image: chrisley75/hello-kubernetes:v0
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: Welcome French Team SA Core - Palo Alto Networks !!!!!
```

If we put this in a file named `deployment.yaml`, then we can apply this file to the cluster as a new desired state:

```shell
$ kubectl create ns sa-workshop
$ kubectl apply -f deployment.yaml -n sa-workshop
```

looking again at `kubectl get pods` we see our newly created pods:

```shell
$ kubectl get pods -n sa-workshop
NAME                                       READY   STATUS         RESTARTS   AGE
hello-kubernetes-custom-79bd487664-lz9df   0/1     ErrImagePull   0          12s
hello-kubernetes-custom-79bd487664-pfh68   0/1     ErrImagePull   0          12s
hello-kubernetes-custom-79bd487664-rls4l   0/1     ErrImagePull   0          12s       <none>
```

If we want to see why the Pod's status is `ErrImagePull` we can examine the logs using:

```shell
$ kubectl logs <pod-name> -n sa-workshop
```

To further investigate this failure, we use `kubectl describe pod`:

```shell
$ kubectl describe pod <pod-name>
```