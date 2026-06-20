kubectl get pods --selector env=dev

kubectl get pods --selector bu=finance

kubectl get all --selector env=prod

kubectl get pods --selector env=prod,bu=finance,tier=frontend

kubectl create -f ./replicaset-definition-1.yaml