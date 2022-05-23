
# Vedlegg G - Fremgangsmåte test tre

**Innhold:**
1. [Innledning](#1-innledning)
2. [Fremgangsmåte og gjennomføring](#2-fremgangsmåte-og-gjennomføring)  
3. [Forklaring av teknisk innhold](#3-forklaring-av-teknisk-innhold)  
3.1. [php-apache.yaml](#31-php-apacheyaml)  
3.2. [hpa-php-apache.yaml](#32-hpa-php-apacheyaml)  
3.3. [Lastgenerering med BusyBox](#33-lastgenerering-med-busybox)  


<br>
<br>

# 1 Innledning
Denne testen er basert på [HorizontalPodAutoscaler Walkthrough _(https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)_](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/), og er en detaljert veiledning i hvordan test tre ble gjennomført!

> **MERK:** _Se [Vedlegg B - Kildekode](https://github.com/CISK-2022-bachelorgruppe/vedlegg/blob/master/Vedlegg%20B%20-%20Kildekode.md) for å se hvor og hvilke versjoner av kildekodene som tilhører test tre_

<br>
<br>

# 2 Fremgangsmåte og gjennomføring
Åpne to terminalvinduer, heretter referert til terminal A og terminal B.

### 1. Start minikube med denne kommandoen i terminal A:
```shell
$ minikube start --driver docker --extra-config=kubelet.housekeeping-interval=10s
```
<br>

For at metrics-server skal fungere i neste steg, så må `--extra-config=kubelet.housekeeping-interval=10s` være med i oppstart av minikube.

Svar fra kommandoen:
```shell
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
<br>

Svar fra kommandoen:
```
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
🌟  The 'metrics-server' addon is enabled
```
<br>

For å se om metrics-server fungerer:
```shell
$ minikube kubectl -- top pods -n kube-system            
```
<br>

Svar fra kommandoen vil være noe lik denne:
```shell
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

### 3. Start _Deployment_ og eksponer _Serivcen_:
```shell
$ minikube kubectl -- apply -f php-apache.yaml
```
<br>

Svar fra kommandoen:
```shell
deployment.apps/php-apache created
service/php-apache created
```
<br>

### 4. Lag HPA og sjekk _current_ status:
```shell
$ minikube kubectl -- apply -f hpa-php-apache.yaml
```
<br>

Svar fra kommandoen:
```shell
horizontalpodautoscaler.autoscaling/php-apache created
```
<br>

Så sjekkes status til HPA med en gang:
```shell
$ minikube kubectl -- get hpa
```
<br>
  
Svar fra kommandoen:
```shell
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   <unknown>/50%   1         10        0          12s
```

Legg merke til `<unknown>`. Dette kan skje hvis statusen sjekkes, før HPA er ferdig deployert. Derfor ble de spurt om status inntil `0%` vises. HPA fungerer selv om `0%` vises.

<br>

Svar fra kommandoen etter ett minutt:
```shell
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          70s
```
<br>


### 5. Generer mer last med BusyBox i Terminal B:
```shell
$ minikube kubectl -- run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
<br>

Svar fra kommandoen:
```
If you don't see a command prompt, try pressing enter.
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```
<br>

### 6. Overvåk HPA i Terminal A:
```shell
$ minikube kubectl -- get hpa php-apache --watch
```
<br>

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

Overvåk autoskaleringen frem til prosessorkraften er stabil rundt `targetCPUUtilizationPercentage=50` i 5 minutter.

<br>

### 7. Stopp lasten BusyBox genererer i Terminal B:
```shell
$ <ctrl> + c
```
<br>

Svar fra kommandoen:
```
OK!OK!OK!OK!OK!^Cpod "load-generator" deleted
pod default/load-generator terminated (Error)`
```
<br>

### 8. Overvåk HPA i Terminal A og se den skalere ned:
```shell
$ minikube kubectl -- get hpa php-apache --watch  
```
<br>

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
Her er en mer detaljert beskrivelse av de forskjellige tekniske komponentene i testen. Det vil ikke bli gjort en detaljert linje-for-linje beskrivelse av .yaml-filene.
[Kapittel 1](#1-innledning) forklarer hvor scriptene kan finnes. 
<br>

## 3.1 php-apache.yaml
Denne filen lager en _Deployment_ og eksponerer den med en _Service_. _Deploymenten_ henter et ferdiglaget php-apache image fra Docker Hub. Imaget er `k8s.gcr.io/hpa-example` og har php-image versjon `php:5-apache` og er bygget opp ved hjelp av en Dockerfile som inneholder:
```yaml
FROM php:5-apache
COPY index.php /var/www/html/index.php
RUN chmod a+rx index.php
```
<br>

Koden over refererer til en `index.php`-side som utfører komplekse regnestykker som krever mye prosessorkraft. Dette er for å simulere lasten i _clusteret_ vårt og er som følger:
```php
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
Denne filen konstruerer en HPA som inneholder beskrivelser om at _Deploymenten_ php-apache skal ha minst én pod og maks ti podder. Denne HPA-en vil enten øke eller minke antall replikeringer innenfor dette intervallet, for å opprettholde ønsket gjennomsnittlig prosessorutnyttelse på tvers av alle podder på cirka 50%. Algoritmen som HPA benytter for å bestemme antall replikeringer baserer seg på forholdet mellom ønsket metrics-verdi og gjeldende metrics-verdi hentet fra `metrics-server`. Her vises algoritmen, som er forenklet for å lett kunne forstå hva den gjør:
```yaml
ønsketreplikeringer = ceil[GjeldendeReplikas * ( GjeldendeMetricVerdi / ØnsketMetricVerdi )]
```
Funksjonen `ceil[]` runder opp til neste heltall.
<br>

## 3.3 Lastgenerering med BusyBox
For å generere en økt last mot php-apache servicen, eksekveres en lastgenerator som heter BusyBox. Den henter et image busybox:1.28 og oppretter en pod som sender forespørsler til `http://php-apache` helt til podden termineres manuelt.

<br>

> **MERK:** _For videre informasjon om scriptene som er benyttet i dette prosjektet, les kildekoden som finnes i [Vedlegg B - Kildekode](https://github.com/CISK-2022-bachelorgruppe/vedlegg/blob/master/Vedlegg%20B%20-%20Kildekode.md)_
