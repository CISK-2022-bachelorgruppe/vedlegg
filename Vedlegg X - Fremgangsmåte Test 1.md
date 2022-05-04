# Introduksjon
Dette er en detaljert veiledning i hvordan test 1 ble gjennomført! Denne fremgangsmåten går ut ifra at repoene tilknyttet testen er lastet ned til hjem-mappen på lab-PCen. "`~/`"
<br>
<br>

# Fremgangsmåte
For å sikre at minikube ikke inneholder endrede filer som kan påvirke testingen, installeres minikube på nytt ved starten av hver test. Dette ble gjort med følgende kommandoer:
```
$ minikube delete
$ minikube start --driver=docker
```
<br>

Når minikube var ferdiginstallert, ble utrullingen av mysql-databasen og django-applikasjonen startet med:
```
$ cd ~/kubernetes-config/mysql
$ kubectl apply -k .
  
$ cd ~/kubernetes-config/django
$ kubectl apply -k .
```
<br>

Deretter ble sched-appen klargjort for kjøring i minikube. Dette gjøres med:
```
$ cd ~/applikasjon/sched
$ eval \$(minikube docker-env)
$ docker build -t sched .
```
<br>

For å starte selve testen med å tvangsstoppe podder og ta tiden på dette ble følgende kommandoer benyttet: 
```
$ cd ~/applikasjon/pod-sletting/
$ ./skript.sh
```
<br>

Testen vil lagre sine resultater i mappen du kjører _skript.sh_ ifra. Siden _skript.sh_ eksekveres fra mappen _~/applikasjon/pod-sletting/_ så vil resultatene legge seg i en mappe _~/applikasjon/pod-sletting/resultater/_

<br>

> **MERK:** _For videre informasjon om scriptene som er benyttet i dette prosjektet, les kildekoden og kommentarene som er skrevet der._