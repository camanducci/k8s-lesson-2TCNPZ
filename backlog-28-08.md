kubectl config get-contexts
kubectl config use-context docker-desktop
kubectl get nodes
kubectl get pods -A
kubectl run nginx-test --image=nginx --port=80
kubectl expose pod nginx-test --type=NodePort --port=80
kubectl get pods
kubectl get svc
kubectl scale pod nginx-test --replicas=3
kubectl delete svc nginx-test
kubectl delete pod nginx-test
kubectl delete pods --all
kubectl delete svc --all
kubectl delete all --all