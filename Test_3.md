# Hvordan gjennomføre Test 3 - HorizontalPodAutoscaling (HPA):

Denne testen er basert på https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
For at man skal kunne gjengi testen 100% må `hpa-php-apache.yaml` og `php-apache.yaml` eksikveres og ikke kommandolinjene som inneholder `kubectl apply -f https://......`

### Pre-install:
- Docker
## 1. Start minikube med denne kommandoen:
```
$ minikube start --driver docker --extra-config=kubelet.housekeeping-interval=10s
```
For at metrics-server skal fungere i neste steg, så må `--extra-config=kubelet.housekeeping-interval=10s` være med under oppstart av minikube.

Svar fra kommandoen:
```
😄  minikube v1.25.1 on Ubuntu 20.04
🎉  minikube 1.25.2 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.25.2
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'


✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🏃  Updating the running docker "minikube" container ...
🐳  Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=10s
🔎  Verifying Kubernetes components...
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass, metrics-server
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## 2. Start den innebygde tilleggsfunksjonen 'metrics-server' i minikube:
```
$ minikube addons enable metrics-server
```
Svar fra kommandoen:
```
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
🌟  The 'metrics-server' addon is enabled
```
For å se om metrics-server fungerer:
```
$ kubectl top pods -n kube-system            
```
Svar fra kommandoen vil være noe lik denne:
```
NAME                               CPU(cores)   MEMORY(bytes)   
coredns-64897985d-q52bj            3m           13Mi            
etcd-minikube                      25m          48Mi            
kube-apiserver-minikube            81m          263Mi           
kube-controller-manager-minikube   32m          48Mi            
kube-proxy-zrn6b                   1m           10Mi            
kube-scheduler-minikube            4m           16Mi            
metrics-server-6b76bd68b6-g5klg    6m           17Mi            
storage-provisioner                2m           9Mi    
```
## 3. Deployer php-apache.yaml
```
$ kubectl apply -f php-apache.yaml
```

Svar fra kommandoen:
```
deployment.apps/php-apache created
service/php-apache created
```

## 4. Lag den horisontalepodautoskalereren og sjekk current status:
```
$ kubectl apply -f hpa-php-apache.yaml
```
Svar fra kommandoen:
```
horizontalpodautoscaler.autoscaling/php-apache created
```

Sjekk status til HPA:
```
$ kubectl get hpa
```
Dette kan ta ett minutt eller to før den registreres.
Svar fra kommandoen med engang:
```
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   **<unknown>**/50%   1         10        0          12s
```
Svar fra kommandoen etter ett minutt:
```
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   **0%**/50%    1         10        1          70s
```

### 4.1 andre muligheter for HPA:
```
$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
Denne komandoen gjør akkurat det samme som filen over.
## 5. Generer en last med busybox i et nytt terminalvindu:
```
$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
Svar fra kommandoen:
```

```
## 6. Overvåk HPA i den første terminalvinduet:
```
$ kubectl get hpa php-apache --watch
```
Svar fra kommandoen:
```

```
## 7. Stopp busybox:
```
$ <ctr> + c
```
Svar fra kommandoen:
```

```
## 8. Overvåk HPA og se den skalere ned:
```
$ kubectl get hpa php-apache --watch  
```
Når HPA detekterer at CPU=0% skalerer den automatisk ned til 1 replika. Dette kan ta noen minutter.
Svar fra kommandoen:
```

```
