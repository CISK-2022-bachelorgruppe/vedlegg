# 1. Introduksjon
Dette er en detaljert veiledning i hvordan test 1 ble gjennomført! Denne fremgangsmåten går ut ifra at repoene tilknyttet testen er lastet ned til hjem-mappen på lab-PCen. "`~/`"
<br>
<br>

# 2. Fremgangsmåte
For å sikre at minikube ikke inneholder endrede filer som kan påvirke testingen, installeres minikube på nytt ved starten av hver test. Dette ble gjort med følgende kommandoer:
```shell
$ minikube delete
$ minikube start --driver=docker
```
<br>

Når minikube var ferdiginstallert, ble utrullingen av mysql-databasen og django-applikasjonen startet med:
```shell
$ cd $HOME/kubernetes-config/mysql
$ kubectl apply -k .
  
$ cd $HOME/kubernetes-config/django
$ kubectl apply -k .
```
<br>

Deretter ble sched-appen klargjort for kjøring i minikube. Dette gjøres med:
```shell
$ cd $HOME/applikasjon/sched
$ eval \$(minikube docker-env)
$ docker build -t sched .
```
<br>

For å starte selve testen med å tvangsstoppe podder og ta tiden på dette ble følgende kommandoer benyttet: 
```shell
$ cd $HOME/applikasjon/pod-sletting/
$ ./skript.sh
```
<br>

Testen vil lagre sine resultater i mappen du kjører _skript.sh_ ifra. Siden _skript.sh_ eksekveres fra mappen _$HOME/applikasjon/pod-sletting/_ så vil resultatene legge seg i en mappe _$HOME/applikasjon/pod-sletting/resultater/_
<br>
<br>

# 3. Gjennomføring
**Ved testens start**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/applikasjoner/main/pod-sletting/resultater/Bilder/Redigerte%20bilder/Bilde%201.png)
<br>
---
<br>

**Sched er deployert**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/applikasjoner/main/pod-sletting/resultater/Bilder/Redigerte%20bilder/Bilde%202.png)
<br>
---
<br>

**Pod 5 er slettet**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/applikasjoner/main/pod-sletting/resultater/Bilder/Redigerte%20bilder/Bilde%203.png)
<br>
---
<br>

**Pod 18 lages**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/applikasjoner/main/pod-sletting/resultater/Bilder/Redigerte%20bilder/Bilde%204.png)
<br>
---
<br>

**Sched termineres**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/applikasjoner/main/pod-sletting/resultater/Bilder/Redigerte%20bilder/Bilde%205.png)
<br>
---
<br>

**Testen er ferdig**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/applikasjoner/main/pod-sletting/resultater/Bilder/Redigerte%20bilder/Bilde%206.png)
<br>
---
<br>


> **MERK:** _For videre informasjon om scriptene som er benyttet i dette prosjektet, les kildekoden og kommentarene som er skrevet der._
