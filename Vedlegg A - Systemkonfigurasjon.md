# Systemkonfigurasjon

**Innhold:**
* [Generelt](#generelt)
1. [GitHub](#1-github)
2. [Basiskonfigurasjon](#2-basiskonfigurasjon)
3. [Egenutviklet applikasjon](#3-egenutviklet-applikasjon)
4. [Kubernetes konfigurasjon](#4-kubernetes-konfigurasjon)
5. [Test én - Pod-terminering](#5-test-én---pod-terminering)
6. [Test to - Skalering](#6-test-to---skalering)
7. [Test tre - HorisontalPodAutoskalering (HPA)](#7-test-tre---horisontalpodautoskalering-hpa)

## Generelt

Dette vedlegget tar for seg hvordan lab-PCene ble konfigurert til å gjennomføre testene.  

I prosjektet ble tre like lab-PCer benyttet. Disse ble utdelt av CISK og modellen er en ThinkBook
14 G2 ITL. Alle PCene har samme maskinvare, men det er forekomster av ulike versjoner med
tanke på OS og programmer. Maskinvaren til PCene er:
* 16GB RAM
* 512GB SSD
* 11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz prosessor

For enkelthetsskyld blir PCene navngitt hhv. A, B og C, hvor PC A ble benyttet til test 1, PC B ble benyttet til test 2 og PC C ble benyttet til test 3.
Lab-PCene ble tanket med unix-basert operativsystem. Operativsystemet på PC A og C er Ubuntu og PC B er Pop!
OS. [Tabell 1](#tabell-1-oversikt-lab-pcer) viser hvilke versjoner av operativsystemer og programvarer som er installert på lab-PCene.

<br>


#### Tabell 1: Oversikt lab-PCer
|   PC  |   OS-versjon          |   docker-versjon  |   minikube-versjon    |   kubectl-versjon |
|   -   |   -                   |   -               |       -               |   -               |
|   A   |   Ubuntu 20.04.4 LTS  | 20.10.14          |   v1.25.2             |   v1.23.3         |
|   B   |   Pop!_OS 21.10       | 20.10.12          |   v1.25.2             |   v1.23.3         |
|   C   |   Ubuntu 20.04.4 LTS  | 20.10.14          |   v1.25.2             |   v1.23.3         |

De systemene som er koblet opp er et kubernetescluster i minikube. <span style="color:red">I tillegg er det satt opp et kubernetescluster ved hjelp av Tanzu versjon V0.10.0.</span>. 


<br>


## 1 GitHub
Git ble benyttet til versjonsontroll av applikasjonen som er laget og et fellesområde hvor filer tilknyttet dette prosjektet er blitt lagret. Alle konfigurasjonsfiler, applikasjoner og vedlegg ligger i GitHub. Alle repoene er samlet i en organisasjon som finnes på denne lenken: https://github.com/CISK-2022-bachelorgruppe

Organisasjonen inneholder tre repoer. Dette er [applikasjoner](https://github.com/CISK-2022-bachelorgruppe/applikasjoner), [kubernetes-config](https://github.com/CISK-2022-bachelorgruppe/kubernetes-config) og [vedlegg](https://github.com/CISK-2022-bachelorgruppe/vedlegg). Se [tabell 2](#tabell-2-gitrepo-oppsummering)

#### Tabell 2: Gitrepo oppsummering
|        Gitrepo        |        Hva finnes i repoet?                       |
|            --         |             --                                    |
| [applikasjoner](https://github.com/CISK-2022-bachelorgruppe/applikasjoner)         |   I dette repoet finnes egenutviklede mikrotjenester som danner én applikasjon. Mikrotjenestene er django-applikasjon, http-server, pythonscript-get, pod-sletting og sched.  |
|    [kubernetes-config](https://github.com/CISK-2022-bachelorgruppe/kubernetes-config)  |   Dette repoet inneholder alle kubernetes YAML-filer som benyttes ved deployering av f. eks. _Statefulset_, _Deployments_, _PersistentVolume_ osv.                                                |
|   [vedlegg](https://github.com/CISK-2022-bachelorgruppe/vedlegg)             |   Dette repoet har vedlegg til dette prosjektet |

<br>
<br>

## 2 Basiskonfigurasjon
Dette kapitlet tar for seg basiskonfigurasjon på lab-PCene som ble benyttet i prosjektet. Dette er for å kunne få et likt oppsett ved en senere anledning, og for å kunne ha likt utgangspunkt for testing. I basiskonfigurasjonen ligger det hovedsakelig to programmer. Dette er Docker og Minikube.
De neste kapitlene vil gjennomgå de prosedyrene som ble utført ved installasjon på lab-pcene som ble benyttet i prosjektet.

> **MERK:** _Denne installasjonen av programvarene vil installere de nysete versjonene som eksisterer. For å etterprøve testene i rapporten er det hensiktsmessig og spesifisere de versjonene som ble benyttet i testene! ved installasjon av programvarene!_  
_For å se hvilke versjoner av de ulike programvarene som ble installert, se [Tabell 1](#tabell-1-oversikt-lab-pcer)_

<br>

### 2.1 Docker
Docker blir i dette prosjektet benyttet som en driver for minikube og må derfor installeres før
minikube. For installasjon av Docker, ble [installasjonsguiden](https://docs.docker.com/engine/install/ubuntu/) til "Docker Inc" fulgt.  
Under vil kommandoene som ble benyttet i installasjonen bli listet opp.

Først ble all gammel konfigurasjon av Docker fjernet med følgende kommando:
```shell
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```
<br>

Så ble Docker installert med et _repository_. Med de neste kommandoene oppdateres og installeres det pakker som gjør det mulig å tillate _repositories_ over HTTPS.
```shell
$ sudo apt-get update

$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
<br>

Her legges Docker Inc sin offisielle GPG-nøkkel til PC-en:
```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
<br>

Så kjøres en kommando for å sette opp et stabilt _repository_ og gjør det klart til innstallering:
```shell
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
<br>

Deretter installeres siste versjon av Docker Engine med følgende kommandoer:  
> **MERK:** _Skal testen etterprøves, bør det spesifiseres hvilken versjon som skal lastes ned her, for å få samme versjon som ble benyttet under forsøket i dette prosjektet. Dette er for å skape et likt testmiljø_
```shell
$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
<br>
Dette er det som må til for å installere Docker på PC-en. 


<br>

### 2.2 Minikube
For å utføre konseptbevisene som er laget, trengs et kubernetescluseter. På grunn av bacheloroppgavens tidsbegrensning var det ikke tid nok til å sette opp et fullskala kubernetescluster.
Derfor ble det installert minikube på egne lab-PCer. Minikube lager et virtuelt kubernetescluster som tillater å teste funksjoner som finnes i fullt oppsatte clustere.

For installasjon av minikube, ble [installasjonsguiden](https://minikube.sigs.k8s.io/docs/start/) til "The Kubernetes Authors" fulgt.  
Under vil kommandoene som ble benyttet i installasjonen bli listet opp.

For å installere siste versjon av minikube på x86-64 Linux med binær nedlastning:
```shell
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
<br>

Etter dette legges brukeren til gruppen docker, så docker kan kjøres uten å benytte sudo foran hver kommando.
```shell
$ sudo usermod -aG docker $USER && newgrp docker
```
<br>

For å starte minikube og gjøre installasjonen ferdig ved førstegangsinstallasjon:
```shell
$ minikube start --driver=docker
```
<br>

For å installere kubectl må denne kommandoen skrives:
```shell
$ minikube kubectl -- get pods -A
```




<br>
<br>

## 3 Egenutviklet applikasjon
Til testene ble det utviklet en applikasjon som består av noen mikrotjenester. Ikke alle tjenestene benyttes i hver test, men den tjenesten som er nødvendig i det testen gjøres blir aktivert.
[Tabell 3](#tabell-3-mikrotjenester) viser oppsummert de mikrotjenestene som ble benyttet i prosjektet, hva de gjør og
hvilken test de ble benyttet i.
Disse tjenestene, utenom mysql-set, kan finnes i repoet applikasjoner.

<br>

#### Tabell 3: Mikrotjenester
| Navn på tjeneste | Hva gjør tjenesten | Benyttes i test: |
| - | - | - |
| djangoapplikasjon | API endepunkt og visualisering av database  | 1 og 2 |
| mysql-set |  MySQL database | 1 og 2 |
| schedapplikasjon | En tjeneste som regelmessig gjør GET forespørsler til API endepunktet i django-applikasjon | 1 |
| pod-sletting  | En tjeneste som regelmessig sletter django-applikasjonpods | 1 |
| python-script-get | Sender X antall forespørsler og skalerer opp poddene etterhvert | 2 |

<span style="color:red">Figur 5.1: Visualisering av pods</span>

<br>

### 3.1 django-applikasjon
<span style="color:red">Denne ligger i midten i figur 5.1.</span> Django-applikasjonen står for brukergrensesnittet til selve applikasjonen. Ved å åpne en nettleser kan brukergrensesnittet til denne tjenesten nås. I dette prosjektet ble denne tjenesten kjørt i minikube med en NodePort, noe som gjør at den kan nås med minikube sin ip-adresse og en port som er forhåndsbestemt. Her var dette 192.168.49.2 med port 30001.
Åpnes nettsiden til django vil dette gjøre et databasesøk og alle treff fra databasen dyttes inn i nettsiden før det vises for brukeren. Dette gjør funksjonen mer krevende å kjøre, men det gjør også at funksjonen er avhengig av at databasen kjører og har nok kapasitet. Se figur 2 for bilde av brukergrensesnittet.

![Django brukergrensesnitt](https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/master/Bilder%20til%20vedlegg/Django-brukergrensesnitt.png)
#### Figur 2: Django brukergrensesnitt
<br>

I tillegg er django-applikasjonen utstyrt med et API hvor databaseforespørsler til mysql-databasen kan utføres. Dette APIet kan nås ved å gå inn på samme nettsted som over, men legge til /api. Åpnes denne nettsiden vil ordet _Suksess_ dukke opp og en ny rad er blitt lagt til i databasen som befinner seg i mysql-set.
<span style="color:red">Bør yaml-filen nevnes her?</span>

<br>

### 3.2 mysql
Mysql-databasen står for persistent lagring av all data lagt inn i databasen. Mysql-set-tjenesten ligger til venstre i figur 5.1. Til å lage mysql-tjenesten ble Oracles offesielle mysql Dockerimage benyttet. I repoet _kubernetes-konfig_ er det definert i mysql-statefulset.yaml-filen at image mysql:5.7 skal benyttes. Dette gjør at nyere versjoner av mysql ikke vil bli benyttet, men sikrer etterprøving av resultater. Det eneste som er endret på er at mysql skal opprette en database med navn _bachelor\_db_ og et passord til databasebrukeren når mysql starter opp. 

I [tabell 4](#tabell-4-database-oppsett) vises alle felt i databasenavnet _bachelor\_db_ og et mysql-tabell som heter _api\_appdb_, samt hva som lagres i feltene.

<br>

#### Tabell 4: Database oppsett
| Felt   | Forklaring av felt  |
| -  | -  |
| id  | indeksnummer på raden  |
| tid  | tid i millisekunder  |
| tidspunkt  | tid i leslig format  |
| intervall  | hvilket intervall sched-applikasjonen er konfigurert til  |
| tid_siden_siste  | tid siden forrige forespørsel i databasen  |
| pod_navn  | hvilket navn podden som gjør GET forespørselen har  |
 
 


<br>


### 3.3 sched-applikasjon
Sched-applikasjonen kjører et pythonscript som sender HTTP GET-forespørseler til APIet i django-applikasjonen med jevne mellomrom. Siden GET-forespørslene er rettet mot APIet vil dette skape databasespørringer som lager én ny rad i databasen for hver GET-forespørsel som gjøres. Denne tjenesten ble opprettet for å kunne se hvor lang tid det tok før en pod svarte igjen hvis den ble terminert, noe som skal simulere en error i podden.

Siden det blir lagt inn en ny rad i databasen om django-podden fungerer, vil det derfor være mulig å se hvor lang tid det tar fra django-applikasjons-podden blir terminert til den er oppe igjen. Dette er fordi raden inneholder klokkeslettet den ble lagt inn.

<br>

### 3.4 pod-sletting
Pod-sletting er en tjeneste som automatiserer mye av test én. Tjenesten startes ved å kjøre _skript.sh_ (bash-script) lokalt på lab-PCen som benyttes til testing. Overordnet vil _skript.sh_ vil først starte sched-applikasjonen, så vil den vente i to minutter før selve testen starter. Når testen starter vil denne tjenesten lagre django-poddens navn i en fil, for så å terminere denne podden. Dette skjer 30 ganger, med 30 sekunders ventetid mellom hver terminering.

Når testen er ferdig, avsluttes sched-applikasjonen og resultatet som ligger i databasen _bachelor\_db_ eksporteres til en .csv-fil på host-maskinen.

<br>

### 3.5 python-script-get
Python-script-get er en tjeneste som automatiserer mye av test 2. Tjenesten startes ved å kjøre _test-gjennomføring.sh_ (bash-script) lokalt på lab-PCen som benyttes til testing. Overordnet vil _test-gjennomføring.sh_ starte testen ved hjelp av 7 argumenter som må legges til. Dette er argumenter som styrer testen, i form av hva ip-adressen og porten til django-applikasjonen er, hvor mange forespørsler som skal sendes og hvor mange gjennomføringer som skal gjøres. For hver gjennomføring starter bash-scriptet et nytt python-script som står for GET-forespørslene mot django-applikasjonen. Bash-scriptet tar tiden på hvor lang tid python-scriptet bruker på å gjennomføre alle GET-forespørslene og gir dermed den avhengige variabelen som skal testes i denne testen.

<br>
<br>

## 4 Kubernetes konfigurasjon
For å kjøre opp applikasjonen beskrevet i kapittel 5.3 i K8s er det benyttet yaml-filer. Disse
YAML-filene ligger i repoet _kubernetes-konfig_. Når YAML-filene skal legges inn i K8s blir
_kustomization.yaml_ benyttet. Dette bidrar til at ku én kommando må skrives for å deployere én
mikrotjeneste. Skal mikrotjenesten django-applikasjon deployeres, vil det derfor være nok å ha
en terminal åpen i mappen django i _kubernetes-konfig_-repoet og kjøre kommandoen: 
```shell
$ minikube kubectl −− apply −k .
```

<br>

### 4.1 django-applikasjon
Til django-applikasjonen benyttes kun objektene _Deployment_ og _Service_. _Deploymenten_ henter et Docker image fra Docker Hub som er utviklet ifb. med dette prosjektet. Docker imaget og versjonen som er benyttet er sjohans1/django-bachelor:6.0.
I tillegg er det definert fem miljøvariabler i denne _Deploymenten_. Dette er variabler som setter opp forbindelse med mysql-tjenesten og inneholder databasenavn, databasebruker, databasebrukerens passord, databasens IP/DNS og databasens port. _Servicen_ er satt opp som en NodePort og gir derfor tilgang til django-applikasjonen ved hjelp av en port. Denne gjør slik at _django-entrypoint_, som er navnet på _Servicen_, blir eksponert som på porten 30001. Dette vil si at det går an å nå tjenesten fra utsiden av K8s-clusteret ved hjelp av ip-adressen til minikube og porten 30001.

<br>

### 4.2 mysql
Til mysql-databasen benyttes objektene _PersistenVolume_, _Secret_, _ConfigMap_, _StatefulSet_ og _Service_. 

I _PersistentVolume_ defineres hvor i minikube dataen, som linkes mot dette volumet, skal lagres. Stien til denne mappen i minikube er: ”_/data/bachelor-mysql_” 

I _Secret_ defineres passordet til databasebrukeren, mens i _ConfigMap_ bestemmes databasens navn. Disse verdiene er hhv. ”_cGFzc3dvcmQK_” og ”_bachelor_db_”. 

_StatefulSettet_ henter et Docker image, på samme måte som django-applikasjonen over, fra Docker Hub. Dette er derimot ikke spesielt utviklet til dette prosjektet og er derfor et offisielt Docker image av mysql. Docker imaget og versjonen som er benyttet er mysql:5.7. Siden dette er et _StatefulSet_ er det montert et volum som bidrar til persistent lagring av data. Det er også definert en _PersistentVolumeClaim_ i _StatefulSettet_ som kobles opp mot _PersistentVolumet_. 

I tillegg er det definert to miljøvariabler. Disse variablene ligger ikke i selve _StatefulSettet_, men de ligger i objektene _Secret_ og _ConfigMap_. Variablene bestemmer databasepassordet til databasen og databasenavnet til databasen som skal opprettes ved deployering av dette StatefulSettet. Service er en vanlig ClusterIP K8s-service, noe som gjør at databasen kun kan aksesseres fra
innsiden av K8s-clusteret.

<br>

### 4.3 sched-applikasjon
Sched-applikasjonen er Som nevnt i kap 5.4 kan YAML-filene deployeres med en kommando. Ved test 1 skal sched-applikasjonen benyttes, men deployeres via scriptet pod-sletting. Kommandoen nevnt i kap 5.trenger derfor ikke å benyttes for sched-applikasjonen.

Til sched-applikasjonen benyttes objektet _Deployment_.  
_Deploymenten_ henter et Docker image, kalt sched:latest, fra lokal maskin, og kjører dette opp med to miljøvariabler. Den ene variabelen bestemmer intervallet til GET-forespørseler, altså tiden det skal ta mellom hver GET-forespørsel, mens den andre variabelen bestemmer hvilken IP/DNS API-et befinner seg på. Her er django-entrypoint:8000 benyttet, ettersom djangoentrypoint er Servicen til django-applikasjonen.

<br>
<br>

## 5 Test én - Pod-terminering
Test én ble gjennomført 29. april 2022 på lab-PC A og benytter fire egenlagde mikrotjenester i denne testen. Dette er django-applikasjon, mysql, sched-applikasjon og pod-sletting. Før denne testen kan kjøres må alle filer tilknyttet testen bli lastet ned. For eksakt versjon av repoet som ble benyttet til test én, se [tabell 5](#tabell-5-gitrepo-versjoner-test-én).

<br>

#### Tabell 5: GitRepo versjoner test én
| GitRepo               | Commit-hash:                              |
| -                     | -                                         |
| applikasjon           | [16caeede03f35f435224556b7a7f0c5a696c78c7](https://github.com/CISK-2022-bachelorgruppe/applikasjoner/tree/16caeede03f35f435224556b7a7f0c5a696c78c7)  |
| kubernetes-config     | [b895df683cd9758f0dd3b6cd559106a5cae26b61](https://github.com/CISK-2022-bachelorgruppe/kubernetes-config/tree/b895df683cd9758f0dd3b6cd559106a5cae26b61)  |



<p align="center" id="Figur 3">
<img src="https://media.istockphoto.com/vectors/missing-rubber-stamp-vector-vector-id1213374148?k=20&m=1213374148&s=612x612&w=0&h=A3_Ku27Jf_XRfsWCZYvwJWQGNR2hbHDh9ViLLaAdJ5w=" style="width:500px">
<br>
<b>Figur 3: Logisk oversikt over test én</b>
</p>

Test én benytter django-applikasjon og mysql-databasen til å danne grunnmuren i applikasjonen. Her dannes grensesnittet til mysql-databasen og muligheten til å lagre persistent data. Til selve testen ble sched-applikasjon og pod-sletting benyttet. Siden denne testen gikk ut på å vise hvor lang tid det tar fra en pod termineres til den fungerer igjen, er sched-applikasjonen konfigurert til å sende en GET-forespørsel til django-applikasjon sitt API hvert millisekund, noe som gjør at det opprettes en ny rad i databasen hvert millisekund. Ved å gjøre dette er det mulig å se tiden hver databaserad er lagt inn og med det finne tiden siden forrige databaserad. Dette gir en indikasjon på hvor lenge APIet til django-applikasjonen har vært nede. 

Denne testen vil ta omtrentlig 17 minutter og 15 sekunder.

> MERK: Detaljerte instruksjoner på gjennomføring av Test én kan finnes i Vedlegg B - Fremgangsmåte Test 1 og for informasjon om kildekoden, kan kommentarene i kildekoden leses.


<br>


## 6 Test to - Skalering
Test to ble gjennomført 30. april 2022 på lab-PC B og benytter tre egenlagde mikrotjenester i denne testen. Dette er django-applikasjon, mysql og python-script-get. Før denne testen kan kjøres må alle filer tilknyttet testen bli lastet ned. For eksakt versjon av repoet som ble benyttet til test to, se [tabell 6](#tabell-6-gitrepo-versjoner-test-to).

#### Tabell 6: GitRepo versjoner test to


| GitRepo               | Commit-hash:                              |
| -                     | -                                         |
| applikasjon           | [76d350309044b92898ae797c95100c09ffa1c232](https://github.com/CISK-2022-bachelorgruppe/applikasjoner/tree/76d350309044b92898ae797c95100c09ffa1c232)  |
| kubernetes-config     | [b895df683cd9758f0dd3b6cd559106a5cae26b61](https://github.com/CISK-2022-bachelorgruppe/kubernetes-config/tree/b895df683cd9758f0dd3b6cd559106a5cae26b61)  |

<p align="center" id="Figur 4">
<img src="https://raw.githubusercontent.com/CISK-2022-bachelorgruppe/vedlegg/master/Bilder%20til%20vedlegg/test2-oppsett.png" style="width:500px">
<br>
<b>Figur 4: Oppsett i Test to</b>
</p>




<br>

Denne testen benytter django-applikasjon og mysql-databasen til å danne grunnmuren i applikasjonen på samme måte som skjer i test 1. Til selve testen ble python-script-get tjenesten benyttet. Siden denne testen gikk ut på å teste ytelsen til en applikasjon opp mot antall django-applikasjons podder som kjører, er python-script-get konfigurert til å sende et valgfritt antall GET-forespørsel til django-applikasjon så fort som mulig. Da kan tiden det tar å utføre alle GET-forespørslene måles, og resultatet bidrar til å måle applikasjonens ytelse.

Tanken med testen er derfor å ta tiden på hvor lang tid det tar å sende alle GET-forespørslene til django-applikasjon og få igjen svaret ”200 OK”. Det som endres på underveis i testen vil være antall tråder som benyttes til å sende GET-forespørsler og antall podder som benyttes til å motta GET-forespørslene.

Når python-script-get ble kjørt, ble argumentene _$(minikube ip)_, _30001_, _200_, _10_, _$HOME/git_, _sj_ og _$HOME/git_ lagt til. Dette førte til en test som kjøres mot den lokale minikubes ip-adresse på port 30001 som er django-applikasjonens _NodePort_. I tillegg ble det definert at 200 GET-forespørsler skulle sendes og at dette skal gjennomføres 10 ganger for hver tråd som legges til. python-script-get definerer at det skal testes fra en til ti tråder og en til ti podder. [Tabell 7](#tabell-7-tester-i-test-2) gir en liten oversikt over et utsnitt av testene som gjøres i test to.


#### Tabell 7: Tester i test 2
|  Testnr. |  Antall pods  | Antall tråder   | Antall GET-forespørsler  |  Antall gjennomføringer |
| -  | -  | -  |  - |  - |
|1     |    1    |    1    |    200    |    10|
|2     |    1    |    2    |    200    |    10|
|3     |    1    |    3    |    200    |    10|
|4     |    1    |    4    |    200    |    10|
|5     |    1    |    5    |    200    |    10|
|6     |    1    |    6    |    200    |    10|
|7     |    1    |    7    |    200    |    10|
|8     |    1    |    8    |    200    |    10|
|9     |    1    |    9    |    200    |    10|
|10    |    1    |    10   |    200    |    10|
|11    |    2    |    1    |    200    |    10|
|12    |    2    |    2    |    200    |    10|
|13    |    2    |    3    |    200    |    10|
|14    |    2    |    4    |    200    |    10|
|15    |    2    |    5    |    200    |    10|
|16    |    2    |    6    |    200    |    10|
|17    |    2    |    7    |    200    |    10|
|18    |    2    |    8    |    200    |    10|
|19    |    2    |    9    |    200    |    10|
|20    |    2    |    10   |    200    |    10|
|21    |    3    |    1    |    200    |    10|
|22    |    3    |    2    |    200    |    10|
|...   |    ...  |    ...  |    ...    |   ...|



> MERK: Detaljerte instruksjoner på gjennomføring av Test to kan finnes i Vedlegg C - Fremgangsmåte Test 2 og for informasjon om kildekoden, kan kommentarene i kildekoden leses.


<br>
<br>


## 7 Test tre - HorisontalPodAutoskalering (HPA)
Test 3 ble gjennomført 28. april 2022 på lab-PC C og benytter konfigurasjonsfiler fra K8s sine nettsider. Konfigurasjonsfilene ble lagt inn i GitHub. Før denne testen kan kjøres må alle filer tilknyttet testen bli lastet ned. For eksakt versjon av repoet som ble benyttet til test tre, se [tabell 8](#tabell-8-gitrepo-versjoner-test-tre).

#### Tabell 8: GitRepo versjoner test tre
| GitRepo               | Commit-hash:                              |
| -                     | -                                         |
| vedlegg/test_3_hpa           | [86f3047459bb0e858809a3049841976329110567](https://github.com/CISK-2022-bachelorgruppe/vedlegg/tree/86f3047459bb0e858809a3049841976329110567/test_3_hpa) |


<p align="center" id="Figur 5">
<img src="https://media.istockphoto.com/vectors/missing-rubber-stamp-vector-vector-id1213374148?k=20&m=1213374148&s=612x612&w=0&h=A3_Ku27Jf_XRfsWCZYvwJWQGNR2hbHDh9ViLLaAdJ5w=" style="width:500px">
<br>
<b>Figur 5: Logisk oversikt over test tre</b>
</p>


Denne testen benytter seg av K8s konfigurasjonsfiler fra K8s sine nettsider som kan finnes her: HorizontalPodAutoscaler Walkthrough.

Denne testen tar ibruk to YAML-filer, hvor den ene filen er en _Deployment_ og den andre er en _HorizontalPodAutoscaler_. _Deploymenten_ starter en php-apache server som har begrenset kapasitet for å kunne nå ytterverdiene av poddenes tilgjengelige ressurser. _HorizontalPodAutoscaler_ definerer at minimum antall replicas skal være én og maksimum ti. I tillegg defineres et ønske om gjennomsnittelig CPU-bruk på php-apache poddene til 50%.

For å sjekke oppskalering må det genereres trafikk på php-apache poddene. Dette gjøres ved å kjøre opp en ny pod med et BusyBox iamge. BusyBox sender forespørseler til php-apache serveren i en uendelig løkke. Etter en stund termineres denne podden manuelt for å stoppe trafikken inn til php-apache serveren. Dette gir resultater angående K8s nedskalering.

> MERK: Detaljerte instruksjoner på gjennomføring av Test tre kan finnes i Vedlegg D - Fremgangsmåte Test 3 og for informasjon om kildekoden, kan kommentarene i kildekoden leses