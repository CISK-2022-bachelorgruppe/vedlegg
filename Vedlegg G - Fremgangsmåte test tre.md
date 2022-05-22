
# Vedlegg G - Fremgangsmåte test tre

**Innhold:**
1. [Innledning](#1-innledning)
2. [Gjennomføring av test tre](#2-gjennomføring-av-test-3)  
2.1 [For å få like resultater](#21-for-å-få-like-resultater)  
2.2 [Fremgangsmåte](#22-fremgangsmåte)
3. [Forklaring av teknisk innhold](#3-forklaring-av-teknisk-innhold)  
3.1 [php-apache.yaml](#31-php-apacheyaml)  
3.2 [hpa-php-apache.yaml](#32-hpa-php-apacheyaml)  
3.3 [Lastgenerering med BusyBox](#33-lastgenerering-med-BusyBox)



<br>
<br>

# 1 Innledning
Denne testen er basert på [HorizontalPodAutoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/), og er en detaljert veiledning i hvordan test tre ble gjennomført!

<br>
<br>

# 2 Gjennomføring av test tre:

### Pre-install:
- Kubernetes _cluster_ → minikube
- Kubectl
- Docker


## 2.1 For å få like resultater:
1. Åpne to Terminalvinduer, heretter referert til Terminal A og Terminal B 
2. Følg testinstruksene under frem til punkt `6. Overvåk HPA i Terminal A:`
3. Overvåk autoskaleringen frem til CPU er stabil rundt `targetCPUUtilizationPercentage=50` i 5 minutter.
4. Deretter følges testinstruksene videre fra punkt `7. Stopp lasten BusyBox genererer i Terminal B:`
5. Overvåk autoskaleringen frem til den går ned til én replika.

<br>

## 2.2 Fremgangsmåte
### 1. Start minikube med denne kommandoen i Terminal A:
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
<br>


### 2. Start den innebygde tilleggsfunksjonen 'metrics-server' i minikube:
```shell
$ minikube addons enable metrics-server
```
Svar fra kommandoen:
```
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
🌟  The 'metrics-server' addon is enabled
```
For å se om metrics-server fungerer:
```shell
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
<br>

### 3. Start Deployment og eksponer Serivcen:
```shell
$ kubectl apply -f php-apache.yaml
```

Svar fra kommandoen:
```
deployment.apps/php-apache created
service/php-apache created
```
<br>

### 4. Lag den horisontalepodautoskalereren og sjekk current status:
```shell
$ kubectl apply -f hpa-php-apache.yaml
```
Svar fra kommandoen:
```
horizontalpodautoscaler.autoscaling/php-apache created
```

Sjekk status til HPA:
```shell
$ kubectl get hpa
```
Dette kan ta ett minutt eller to før den registreres. Legg merke til `<unknown>` og `<0%>`, den fungerer når det vises `<0%>`
Svar fra kommandoen med engang:
```
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   <unknown>/50%   1         10        0          12s
```
Svar fra kommandoen etter ett minutt:
```
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          70s
```
<br>


### 5. Generer mer last med BusyBox i Terminal B:
```shell
$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
Svar fra kommandoen:
```
If you don't see a command prompt, try pressing enter.
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```
<br>

### 6. Overvåk HPA i Terminal A:
```shell
$ kubectl get hpa php-apache --watch
```
Denne kommandoen kjøres på et intervall på 15 sekunder.

Svar fra kommandoen etter noen minutter:
```
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          58s
php-apache   Deployment/php-apache   132%/50%   1         10        1          75s
php-apache   Deployment/php-apache   171%/50%   1         10        3          90s
php-apache   Deployment/php-apache   157%/50%   1         10        4          105s
php-apache   Deployment/php-apache   61%/50%    1         10        4          2m
php-apache   Deployment/php-apache   65%/50%    1         10        4          2m15s
php-apache   Deployment/php-apache   78%/50%    1         10        4          2m30s
php-apache   Deployment/php-apache   72%/50%    1         10        7          2m45s
php-apache   Deployment/php-apache   57%/50%    1         10        7          3m
php-apache   Deployment/php-apache   43%/50%   1         10        7          3m21s
php-apache   Deployment/php-apache   51%/50%   1         10        7          3m30s
```
<br>

### 7. Stopp lasten BusyBox genererer i Terminal B:
```
$ <ctrl> + c
```
Svar fra kommandoen:
```
OK!OK!OK!OK!OK!^Cpod "load-generator" deleted
pod default/load-generator terminated (Error)`
```
<br>

### 8. Overvåk HPA i Terminal A og se den skalere ned:
```shell
$ kubectl get hpa php-apache --watch  
```
Når HPA detekterer at CPU=0% skalerer den automatisk ned til 1 replika. Dette kan ta noen minutter.

Svar fra kommandoen:
```
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   42%/50%   1         10        7          4m46s
php-apache   Deployment/php-apache   5%/50%    1         10        7          5m1s
php-apache   Deployment/php-apache   0%/50%    1         10        7          5m16s
php-apache   Deployment/php-apache   0%/50%    1         10        7          7m
php-apache   Deployment/php-apache   0%/50%    1         10        7          7m52s
php-apache   Deployment/php-apache   0%/50%    1         10        7          9m31s
php-apache   Deployment/php-apache   0%/50%    1         10        6          9m46s
php-apache   Deployment/php-apache   0%/50%    1         10        1          10m
```
<br>
<br>

# 3 Forklaring av teknisk innhold:
Her blir det  en mer detaljert beskrivelse av de forskjellige tekniske komponentene i testen. Det vil ikke bli gjort en detaljert linje-for-linje beskrivelse av .yaml-filene.
<br>

## 3.1 php-apache.yaml
Denne filen lager en Deployment og eksponerer den som en _Service_. _Deploymenten_ henter et ferdiglaget php-apache-image fra Docker Hub. Image er `k8s.gcr.io/hpa-example` og har php-image-versjon `php:5-apache` og er bygget opp slikt:
```
FROM php:5-apache
COPY index.php /var/www/html/index.php
RUN chmod a+rx index.php
```
Koden over refererer til en `index.php`-side som utfører komplekse regnestykker som krever mye CPU. Dette er for å simulere lasten i _clusteret_ vårt og er som følger:
```
<?php
  $x ) 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    4x += sqrt($x);
  }
  echo "OK!";
?>
```
<br>

## 3.2 hpa-php-apache.yaml
Denne filen konstruerer en horisontal autoskalerer som vedlikeholder mellom én og ti replikeringer av podder som blir kontrollert av _Deploymenten_ php-apache. Denne HPA-en vil da enten øke eller minke antall replikeringer for å opprettholde ønsket gjennomsnittlig CPU-utnyttelse på tvers av alle poder på cirka 50%. Algoritmen som HPA benytter for å bestemme antall replikeringer baserer seg på forholdet mellom ønsket metricsverdi og gjeldende metricsverdi hentet fra `metrics-server`. Den forenklede algoritmen er som følger:
```
ønsketreplikeringer = ceil[GjeldendeReolikas * ( GjeldendeMetriskVerdi / ØnsketMetriskVerdi )]
```
Hvis skaleringsforholdet befinner seg nært 1.0 så vil _control plane_ hoppe over skaleringen.
<br>

## 3.3 Lastgenerering med BusyBox
For å generere en økt last mot php-apache servicen, så blir dette utført via `kubectl run`-kommandoen. Denne lar oss eksekvere en lastgenerator som heter BusyBox. Her henter den image busybox:1.28. Videre utfører kommandoen en uendelig løkke som sender forespørsler til `http://php-apache`. 
