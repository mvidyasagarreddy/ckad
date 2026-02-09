# ckad
This respository consists of all the practiced files related to ckad certification


kubectl get pods --namespace=dev

kubectl get pods

kubectl get pods --namespace=prod

kubectl config set-context $(kubectl config current-context) --namespace=dev

kubectl get pods --namespace=default

kubectl get pods --all-namespaces

kubectl create -f compute-quota.yaml