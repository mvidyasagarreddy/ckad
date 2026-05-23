This is a simple application that generates logs. We can view the logs using the following command:
kubectl get logs simple-app

The above command will show the logs of the `simple-app` pod. If there are multiple containers in the pod, you can specify the container name using the `-c` flag:
kubectl logs simple-app -c simple-container