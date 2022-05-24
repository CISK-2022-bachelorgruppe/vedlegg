# Vedlegg C - Fremgangsmåte test én

**Innhold:**
1. [Introduksjon](#1-introduksjon)
2. [Fremgangsmåte](#2-fremgangsmåte)
3. [Gjennomføring](#3-gjennomføring)  
\- [Ved testens start](#ved-testens-start)  
\- [Sched er deployert](#sched-er-deployert)  
\- [Pod 5 ble akkurat slettet](#pod-5-ble-akkurat-slettet)  
\- [Figuren viser at pod 18 lages](#figuren-viser-at-pod-18-lages)  
\- [Sched termineres](#sched-termineres)  
\- [Testen er ferdig](#testen-er-ferdig)  

<br>
<br>

# 1. Introduksjon
Dette er en detaljert veiledning i hvordan test én ble gjennomført! Denne fremgangsmåten går ut ifra at alle filer som er nødvendige for å eksekvere test én, er lastet ned til hjem-mappen på maskinen "`~/`". Testen ble gjennomført på maskin A.
> **MERK:** _Se [Vedlegg B - Kildekode](https://github.com/CISK-2022-bachelorgruppe/vedlegg/blob/master/Vedlegg%20B%20-%20Kildekode.md) for å se hvor og hvilke versjoner av kildekodene som tilhører test én_

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
$ minikube kubectl -- apply -k .
  
$ cd $HOME/kubernetes-config/django
$ minikube kubectl -- apply -k .
```
<br>

Deretter ble sched-appen klargjort for kjøring i minikube. Dette gjøres med:
```shell
$ cd $HOME/applikasjon/sched
$ eval \$(minikube docker-env)
$ docker build -t sched .
```
<br>

For å starte selve testen med å tvangstoppe podder og ta tiden på dette ble følgende kommandoer benyttet: 
```shell
$ cd $HOME/applikasjon/pod-sletting/
$ ./skript.sh
```
<br>

Testen vil lagre sine resultater i mappen hvor kjører _skript.sh_ ifra. Siden _skript.sh_ eksekveres fra mappen _$HOME/applikasjon/pod-sletting/_, vil resultatene legge seg i en mappe _$HOME/applikasjon/pod-sletting/resultater/_

<br>
<br>

# 3. Gjennomføring
Under vil seks figurer listes opp, hvor tittelen til hver figur er en kort beskrivelse av når figuren ble tatt eller hva som kan observeres på figuren.
Alle figurene er tatt underveis i _skript.sh_ sin utførelse. 

<br>

**Ved testens start**  
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/d58ef1a739af2d989223295ae508c693e76433c6/Vedlegg%20E%20-%20Resultater%20Test%20%C3%A9n/Bearbeidet%20resultater/Redigerte%20bilder/Skjermdump%20fra%202022-04-29%2014-23-47.png)
<br>
---
<br>

**Sched er deployert**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/d58ef1a739af2d989223295ae508c693e76433c6/Vedlegg%20E%20-%20Resultater%20Test%20%C3%A9n/Bearbeidet%20resultater/Redigerte%20bilder/Skjermdump%20fra%202022-04-29%2014-24-15.png)
<br>
---
<br>

**Pod 5 ble akkurat slettet**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/d58ef1a739af2d989223295ae508c693e76433c6/Vedlegg%20E%20-%20Resultater%20Test%20%C3%A9n/Bearbeidet%20resultater/Redigerte%20bilder/Skjermdump%20fra%202022-04-29%2014-28-00.png)
<br>
---
<br>

**Figuren viser at pod 18 lages**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/d58ef1a739af2d989223295ae508c693e76433c6/Vedlegg%20E%20-%20Resultater%20Test%20%C3%A9n/Bearbeidet%20resultater/Redigerte%20bilder/Skjermdump%20fra%202022-04-29%2014-34-00.png)
<br>
---
<br>

**Sched termineres**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/d58ef1a739af2d989223295ae508c693e76433c6/Vedlegg%20E%20-%20Resultater%20Test%20%C3%A9n/Bearbeidet%20resultater/Redigerte%20bilder/Skjermdump%20fra%202022-04-29%2014-41-03.png)
<br>
---
<br>

**Testen er ferdig**
![Bilde](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/d58ef1a739af2d989223295ae508c693e76433c6/Vedlegg%20E%20-%20Resultater%20Test%20%C3%A9n/Bearbeidet%20resultater/Redigerte%20bilder/Skjermdump%20fra%202022-04-29%2014-41-38.png)
<br>
---
<br>


> **MERK:** _For videre informasjon om scriptene som er benyttet i dette prosjektet, les kildekoden som finnes i [Vedlegg B - Kildekode](https://github.com/CISK-2022-bachelorgruppe/vedlegg/blob/master/Vedlegg%20B%20-%20Kildekode.md)_
