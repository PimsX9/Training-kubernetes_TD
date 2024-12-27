# TD Kubernetes

## Exercice 1 - Installation
1. Installer la dernière version de Minikube
2. Vérifier que minikube est bien installée ( minikube version )
```
osadmin@PORT-43814:~$ minikube version
minikube version: v1.34.0
commit: 210b148df93a80eb872ecbeb7e35281b3c582c61
```

Rappel pour les exercices suivants :
Impératif : via lignes de commandes
Déclaratif : approche IaC -> via des fichiers yml

## Exercice 2 - Manipulation basique de pods
Le but de cet exercice est de comprendre comment fonctionne la création, la suppression et la modification de pods.

1. Avec la méthode impérative, créer un pod nginx avec pour nom nginx-pod-imperatif.
```
root@PORT-43814:~# kubectl run nginx-pod-imperatif --image nginx:latest
pod/nginx-pod-imperatif created

root@PORT-43814:~# kubectl get pods | grep nginx-pod
nginx-pod-imperatif               1/1     Running   0               37s
```
2. Avec la méthode déclarative, créer un pod nginx avec pour nom nginx-pod-declaratif.
```
root@PORT-43814:~# kubectl run nginx-pod-declaratif --image nginx --dry-run=client -o yaml > nginx-pod-declaratif.yml
root@PORT-43814:~# cat nginx-pod-declaratif.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod-declaratif
  name: nginx-pod-declaratif
spec:
  containers:
  - image: nginx
    name: nginx-pod-declaratif
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@PORT-43814:~# kubectl apply -f nginx-pod-declaratif.yml
pod/nginx-pod-declaratif created
root@PORT-43814:~# kubectl get pods | grep nginx-pod
nginx-pod-declaratif              1/1     Running   0               8s
nginx-pod-imperatif               1/1     Running   0               3m31s

```
3. Avec la méthode impérative, créer un script shell permettant la création de 10 pods nginx nommé de nginx-1 à nginx-10
```
root@PORT-43814:~# for i in $(seq 1 10); do kubectl run nginx-$i --image nginx; done
pod/nginx-1 created
pod/nginx-2 created
pod/nginx-3 created
pod/nginx-4 created
pod/nginx-5 created
pod/nginx-6 created
pod/nginx-7 created
pod/nginx-8 created
pod/nginx-9 created
pod/nginx-10 created

root@PORT-43814:~# kubectl get pods | grep nginx-
nginx-1                           1/1     Running   0             21s
nginx-10                          1/1     Running   0             21s
nginx-2                           1/1     Running   0             21s
nginx-3                           1/1     Running   0             21s
nginx-4                           1/1     Running   0             21s
nginx-5                           1/1     Running   0             21s
nginx-6                           1/1     Running   0             21s
nginx-7                           1/1     Running   0             21s
nginx-8                           1/1     Running   0             21s
nginx-9                           1/1     Running   0             21s
```
4. Stocker le nom et l’adresse IP de tous les pods dans un fichier nommé all-pods.txt
```
root@PORT-43814:~# kubectl get pod -o custom-columns=NAME:metadata.name,IP:status.podIP > all-pods.txt
root@PORT-43814:~# cat all-pods.txt
NAME                              IP
frontend-5n7r9                    10.244.0.32
frontend-fn2nh                    10.244.0.26
frontend-hpg2c                    10.244.0.25
nginx                             10.244.0.34
nginx-1                           10.244.0.37
nginx-10                          10.244.0.44
nginx-2                           10.244.0.38
nginx-3                           10.244.0.39
nginx-4                           10.244.0.40
nginx-5                           10.244.0.42
nginx-6                           10.244.0.45
nginx-7                           10.244.0.43
nginx-8                           10.244.0.46
nginx-9                           10.244.0.41
nginx-deployment-bf56f49c-6n9w5   10.244.0.29
nginx-deployment-bf56f49c-r9h8k   10.244.0.28
nginx-deployment-bf56f49c-rdghx   10.244.0.27
nginx-pod-declaratif              10.244.0.36
nginx-pod-imperatif               10.244.0.35
nginxdeclaratif                   10.244.0.33
testdeployment-654d67997d-7rbqj   10.244.0.30
```
5. Supprimer l’ensemble des pods créés pour cet exercice.
```
root@PORT-43814:~# kubectl delete pods --all
pod "frontend-5n7r9" deleted
pod "frontend-fn2nh" deleted
pod "frontend-hpg2c" deleted
pod "nginx" deleted
pod "nginx-1" deleted
pod "nginx-10" deleted
pod "nginx-2" deleted
pod "nginx-3" deleted
pod "nginx-4" deleted
pod "nginx-5" deleted
pod "nginx-6" deleted
pod "nginx-7" deleted
pod "nginx-8" deleted
pod "nginx-9" deleted
pod "nginx-deployment-bf56f49c-6n9w5" deleted
pod "nginx-deployment-bf56f49c-r9h8k" deleted
pod "nginx-deployment-bf56f49c-rdghx" deleted
pod "nginx-pod-declaratif" deleted
pod "nginx-pod-imperatif" deleted
pod "nginxdeclaratif" deleted
pod "testdeployment-654d67997d-7rbqj" deleted
```
## Exercice 3 - Manipulation de replicasets
Le but de cet exercice est de comprendre comment fonctionne la création, la suppression et la modification d’un replicaset.

1. Créer un réplicaset nommé webapp-replicaset-declaratif créé à partir de l’image que vous avez poussé sur votre repository lors du précédent TP ( si vous n’avez pas fait l’exercice partir d’une image nginx ) de manière déclarative.
```
root@PORT-43814:~# vim webapp-replicaset-declaratif.yaml
root@PORT-43814:~# cat webapp-replicaset-declaratif.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp-replicaset-declaratif
  labels:
    app: docker-db-connection
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: docker-db-connection
        image: pimsx9/docker-db-connection:1.0.0

root@PORT-43814:~# kubectl apply -f webapp-replicaset-declaratif.yaml
replicaset.apps/webapp-replicaset-declaratif created

root@PORT-43814:~# kubectl get replicaset | grep webapp
webapp-replicaset-declaratif   3         3         2       32s

root@PORT-43814:~# kubectl get pods | grep webapp
webapp-replicaset-declaratif-2szg9   1/1     Running   0          49s
webapp-replicaset-declaratif-ktw8x   1/1     Running   0          49s
webapp-replicaset-declaratif-z5rm5   1/1     Running   0          49s
```
2. Scaler le premier replicas à 5 instances de manière impérative.
```
root@PORT-43814:~# kubectl scale replicaset webapp-replicaset-declaratif --replicas 5
replicaset.apps/webapp-replicaset-declaratif scaled
root@PORT-43814:~# kubectl get replicaset | grep webapp
webapp-replicaset-declaratif   5         5         5       5m37s
```
3. Scaler le second replicas à 7 instances de manière déclarative.
```
Modification du fichier webapp-replicaset-declaratif.yaml :

root@PORT-43814:~# vim webapp-replicaset-declaratif.yaml
root@PORT-43814:~# cat webapp-replicaset-declaratif.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp-replicaset-declaratif
  labels:
    app: docker-db-connection
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 7
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: docker-db-connection
        image: pimsx9/docker-db-connection:1.0.0

root@PORT-43814:~# kubectl apply -f webapp-replicaset-declaratif.yaml
replicaset.apps/webapp-replicaset-declaratif created

root@PORT-43814:~# kubectl get replicaset | grep webapp
webapp-replicaset-declaratif   7         7         7       32s
```
4. Quel mécanisme proposé par Kubernetes devrons-nous utiliser ici pour exposer nos
applications sur le web ?
```
Dans le cadre de minikube, nous pourrons utiliser un service de type NodePort, mais dans un environnement de production, on utilisera un service de type LoadBalancer. 

```
5. Supprimer le replicaset.
```
root@PORT-43814:~# kubectl delete replicaset webapp-replicaset-declaratif
replicaset.apps "webapp-replicaset-declaratif" deleted
root@PORT-43814:~# kubectl get replicaset | grep webapp
```
## Exercice 4 - Manipulation de déploiements.
Objectif: Le but de cet exercice est de comprendre comment fonctionne la création, la suppression et la modification et la mise à jour d’un déploiement.

1. Créer un deployment nommé nginx-deployment-imperatif créer à partir de l’image nginx:1.15.0
```
root@PORT-43814:~# kubectl create deployment nginx-deployment-imperatif --image nginx:1.15.0
deployment.apps/nginx-deployment-imperatif created
```
2. Créer un deployment nommé webapp-deployment-declaratif créer à partir de l’image nginx:1.15.0
```
root@PORT-43814:~# kubectl create deployment nginx-deployment-declaratif --image nginx:1.15.0 --dry-run=client -o yaml > webapp-deployment-declaratif.yaml
root@PORT-43814:~# cat webapp-deployment-declaratif.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment-declaratif
  name: nginx-deployment-declaratif
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment-declaratif
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment-declaratif
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources: {}
status: {}

root@PORT-43814:~# kubectl apply -f webapp-deployment-declaratif.yaml
deployment.apps/nginx-deployment-declaratif created

```
3. Lister les déploiements présents dans votre cluster.
```
root@PORT-43814:~# kubectl get deployment
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment-declaratif   1/1     1            1           45s
nginx-deployment-imperatif    1/1     1            1           12m

```
4. Upgrader l’image du premier déploiement de manière impérative vers la dernière version de nginx.
```
root@PORT-43814:~# kubectl set image deployment nginx-deployment-imperatif nginx=nginx:latest
deployment.apps/nginx-deployment-imperatif image updated

root@PORT-43814:~# kubectl get deployment.apps nginx-deployment-imperatif -o yaml | grep image
      - image: nginx:latest
```
5. Upgrader l’image du dernier déploiement de manière déclarative vers la dernière version de nginx.
```
root@PORT-43814:~# vim webapp-deployment-declaratif.yaml
root@PORT-43814:~# cat webapp-deployment-declaratif.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment-declaratif
  name: nginx-deployment-declaratif
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment-declaratif
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment-declaratif
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        resources: {}
status: {}

root@PORT-43814:~# kubectl apply -f webapp-deployment-declaratif.yaml
deployment.apps/nginx-deployment-declaratif configured

root@PORT-43814:~# kubectl get deployment.apps nginx-deployment-declaratif -o yaml | grep image
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"app":"nginx-deployment-declaratif"},"name":"nginx-deployment-declaratif","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"nginx-deployment-declaratif"}},"strategy":{},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"nginx-deployment-declaratif"}},"spec":{"containers":[{"image":"nginx:latest","name":"nginx","resources":{}}]}}},"status":{}}
      - image: nginx:latest

```
6. Lister les replica sets présent dans votre cluster.
```
root@PORT-43814:~# kubectl get replicaset
NAME                                     DESIRED   CURRENT   READY   AGE
nginx-deployment-declaratif-5dc495f56d   1         1         1       58s
nginx-deployment-declaratif-78cddfcc96   0         0         0       7m45s
nginx-deployment-imperatif-5b974cf868    0         0         0       19m
nginx-deployment-imperatif-7f6d8699d8    1         1         1       5m46s
```
7. Qu’en constatez-vous ?
```
Je constate que les anciens déploiements sont toujours présents mais ils ne sont plus actifs. Cela permettra de faire des rollbacks en cas d'erreur sur les nouveaux déploiements.
```
8. Faire un rollback impératif à la version antérieure des deux déploiements.
```
root@PORT-43814:~# kubectl rollout undo deployment nginx-deployment-declaratif nginx-deployment-imperatif
deployment.apps/nginx-deployment-declaratif rolled back
deployment.apps/nginx-deployment-imperatif rolled back

root@PORT-43814:~# kubectl get deployment.apps nginx-deployment-declaratif -o yaml | grep image
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"app":"nginx-deployment-declaratif"},"name":"nginx-deployment-declaratif","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"nginx-deployment-declaratif"}},"strategy":{},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"nginx-deployment-declaratif"}},"spec":{"containers":[{"image":"nginx:latest","name":"nginx","resources":{}}]}}},"status":{}}
      - image: nginx:1.15.0
        imagePullPolicy: IfNotPresent
root@PORT-43814:~# kubectl get deployment.apps nginx-deployment-imperatif -o yaml | grep image
      - image: nginx:1.15.0
        imagePullPolicy: IfNotPresent

```
9. Supprimer les deux déploiements.
```
root@PORT-43814:~# kubectl delete deployment nginx-deployment-imperatif nginx-deployment-declaratif
deployment.apps "nginx-deployment-imperatif" deleted
deployment.apps "nginx-deployment-declaratif" deleted
root@PORT-43814:~# kubectl get deployment
No resources found in default namespace.
```
## Exercice 5 - Manipulation de Services
1. Créer un déploiement nginx-deploiement à partir d’une image nginx.
```
root@PORT-43814:~# kubectl create deployment nginx-deployment --image nginx:latest
deployment.apps/nginx-deployment created
```
2. Lier les containers du déploiement nginx-deployment avec un nouveau service de type ClusterIp nommer nginx-svc-cluster-ip.
```
root@PORT-43814:~# kubectl expose deployment nginx-deployment --type ClusterIP --port 8080 --target-port 80 --name nginx-svc-cluster-ip
service/nginx-svc-cluster-ip exposed

root@PORT-43814:~# kubectl get services | grep nginx-svc
nginx-svc-cluster-ip   ClusterIP      10.108.225.125   <none>        8080/TCP         2m29s
```
3. Récupérer l’adresse IP du service et le stocker dans un fichier nommer service-ip.txt
```
root@PORT-43814:~# kubectl get services nginx-svc-cluster-ip -o custom-columns=SERVICE-IP:.spec.clusterIP > service-ip.txt
root@PORT-43814:~# cat service-ip.txt
SERVICE-IP
10.108.225.125
```
4. Créer un pods nommé utils à partir d’une image nginx.
```
root@PORT-43814:~# kubectl run utils --image nginx:latest
pod/utils created
```
5. Se connecter au pods utils ( kubectl exec -ti <nom-du-pod> – /bin/bash )
```
root@PORT-43814:~# kubectl exec -ti utils -- /bin/bash
root@utils:/#
```
6. Installer curl
```
root@utils:/# apt update && apt install curl

curl est déjà présent sur le pods.
```
7. Lancer un curl sur le service nginx-svc-cluster-ip.
```
root@utils:/# curl 10.108.225.125:8080
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
8. Avec vos mots, expliquez le concept de service dans kubernetes.
```
Les services permettent de gérer la partie réseau au sein des pods, par exemple avec le service nodeport ou loadbalancers. Nous pouvons donner un accès externe aux utilisateurs ou bien via le service clusterIP qui permet la communication entre plusieurs pods, par exemple avec un Front-end et Back-end. Sans ses services, nous serions bien embêtés pour faire communiquer nos pods entre eux ou tout simplement accéder au service rendu via un accès externe.

```
9. Avec vos mots, expliquez les différences entre un service de type LoadBalancer, ClusterIp && NodePort.
```
LoadBalancer :
Le service n'est disponible qu'auprès d'un cloud provider, il permet d'exposer l'application à l'extérieur avec un mécanisme de partage de charge pour partager le trafic sur l'ensemble des Pods d'un même service. Le service utilise également les services ClusterIP et NodePort pour fonctionner.

ClusterIP :
Il permet la communication entre différents pods à l'intérieur d'un cluster Kubernetes.

NodePort :
Il permet d'exposer un service sur un numéro de port fixe, grâce à cela nous pouvons accéder à l'application depuis l'extérieur, mais le service n'effectue pas de partage de charge.
Il peut être couplé avec le service ClusterIP dans le cas où il y a plusieurs pods du même service.
```
