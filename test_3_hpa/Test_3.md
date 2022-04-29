# VEDLEGG CCC - Test 3: HorizontalPodAutoscaling (HPA):
Denne testen er basert på [HorizontalPodAutoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

Innhold:
- Innledende informasjon om Test 3
- Gjennomføring av Test 3
- Forklaring av teknsik innhold

 


# 1. Innledende informasjon om Test 3:
Denne testen vil ikke ta for seg vår egenproduserte applikasjon. Grunnen til dette er fordi applikasjonen består av flere variabler som kan påvirke testen og igjen kan bli unødvedig komplisert for demonstrere at den horisontal autoskaleringsfunksjonen i Kubernetes fungerer. For simpelhetens skyld vil denne testen benytte seg av en Deployment som eksekverer en konteiner som igjen bruker et php-apache image hentet fra DockerHub, og eksponerer denne som en Serivce. Dette kommer frem i php-apache.yaml. Denne php-apache serveren vil bli eksekvert sammen med hpa-php-apache.yaml. Deretter vil det bli generert en last som sender forespørseler til php-apache servicen i en uendelig løkke. Etter en stund skal lasten avsluttes manuelt.

# 2. Gjennomføring av Test 3:

### Pre-install:
- Kubernetes cluster --> minikube
- Kubectl
- Docker


## 2.1 For å få like resultater:
1. Åpne to Terminalvinduer, heretter referert til Terminal A og Terminal B 
2. Følg testinstruksene under frem til punkt `6. Overvåk HPA i Terminal A:`
3. Overvåk autoskaleringen frem til CPU er stabil rundt `targetCPUUtilizationPercentage=50` i 5 minutter.
4. Deretter følges testinstruksene videre fra punkt `7. Stopp lasten busybox genererer i Terminal B:`
5. Overvåk autoskaleringen frem til den går ned til én replika.

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

### 2. Start den innebygde tilleggsfunksjonen 'metrics-server' i minikube:
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
### 3. start Deployment og eksponer Serivcen:
```
$ kubectl apply -f php-apache.yaml
```

Svar fra kommandoen:
```
deployment.apps/php-apache created
service/php-apache created
```

### 4. Lag den horisontalepodautoskalereren og sjekk current status:
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

### 5. Generer mer last med busybox i Terminal B:
```
$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
Svar fra kommandoen:
```
If you don't see a command prompt, try pressing enter.
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```
### 6. Overvåk HPA i Terminal A:
```
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
### 7. Stopp lasten busybox genererer i Terminal B:
```
$ <ctr> + c
```
Svar fra kommandoen:
```
OK!OK!OK!OK!OK!^Cpod "load-generator" deleted
pod default/load-generator terminated (Error)`
```
    
### 8. Overvåk HPA i Terminal A og se den skalere ned:
```
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

# 3. Forklaring av teknisk innhold:
Her blir det  en mer detaljert beskrivelse av de forskjellige tekniskekomponentene i testen. Det vil ikke bli gjort en detaljert linje-for-linje beskrivelse av .yaml-filene.

## 3.1 php-apache.yaml
Denne filen lager en Deployment og eksponerer den som en Serive. Deplymentet henter et ferdiglaget php-apache-image fra Docker Hub. Image er `k8s.gcr.io/hpa-example` og har php-image-versjon `php:5-apache` og er bygget opp slikt:
```
FROM php:5-apache
COPY index.php /var/www/html/index.php
RUN chmod a+rx index.php
```
Koden over refererer til en `index.php`-side som utfører komplekse regnestykker som krever mye CPU. Dette er for å simulere lasten i clusteret vårt og er som følger:
```
<?php
  $x ) 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    4x += sqrt($x);
  }
  echo "OK!";
?>
```

## 3.2 hpa-php-apache.yaml
Denne filen konstruerer en horisontal autoskalerer som vedlikeholder mellom 1 og 10 replikas av poder som blir kontrollert av Deploymenten php-apache. Denne HPA-en vil da enten øke eller minke antall replikas for å opprettholde ønsket gjennomsnittlig CPU-utnyttelse på tvers av alle poder på cirka 50%. Algoritmen som HPA benytter for å bestemme antall replikas baserer seg på forholdet mellom ønsket metriksverdi og gjeldende metriksverdi hentet fra `metrics-server`. Den forenklete algoritmen er som følger:
```
ønsketReplikas = ceil[GjeldendeReolikas * ( GjeldendeMetriskVerdi / ØnsketMetriskVerdi )]
```
Hvis skaleringsforholdet befinner seg nært 1.0 så vil control plane hoppe over skaleringen.
## 3.3 Lastgenerering med Busybox
For å generere en økt last mot php-apache servicen, så blir dette utført via `kubectl run`-kommandoen. Denne lar oss eksekvere en lastgenerator som heter BusyBox. Her henter den image busybox:1.28. Videre utfører kommanoen en uendelig løkke som sender forespørsler til `http://php-apache`. 
