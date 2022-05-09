# 1. Introduksjon
I dette vedlegget vil grunnkonfigurasjonen som er utført på lab-pcene gjennomgås. Dette er for
å kunne få et likt oppsett ved en senere anledning, og for å kunne ha likt utgangspunkt for testing.
> **MERK:** _Denne installasjonen av programvarene vil installere de nysete versjonene som eksisterer. For å etterprøve testene i rapporten er det hensiktsmessig og spesifisere de versjonene som ble benyttet i testene! ved installasjon av programvarene!_

<br>
<br>

# 2. Installasjon av programvare
I basiskonfigurasjonen ligger det hovedsakelig to programmer. Dette er Docker og Minikube.
De neste kapitlene vil gjennomgå de prosedyrene som ble utført ved installasjon på lab-pcene
som ble benyttet i prosjektet.

> **MERK:** _For å se hvilke versjoner av de ulike programvarene som ble installert, se rapporten  [SYSTEMKONFIKURASJON]_

<br>
<br>

## 2.1 Docker
For installasjon av Docker, ble [installasjonsguiden](https://docs.docker.com/engine/install/ubuntu/) til "Docker Inc" fulgt.  
Under vil kommandoene som ble benyttet i installasjonen bli listet opp.

Først ble all gammel konfigurasjon av Docker fjernet med følgende kommando:
```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
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
<br>



## 2.2 minikube
For installasjon av minikube, ble [installasjonsguiden](https://minikube.sigs.k8s.io/docs/start/) til "The Kubernetes Authors" fulgt.  
Under vil kommandoene som ble benyttet i installasjonen bli listet opp.

For å installere siste versjon av minikube på x86-64 Linux med binær nedlastning:
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
<br>

Etter dette legges brukeren til gruppen docker, så docker kan kjøres uten å benytte sudo foran hver kommando.
```shell
sudo usermod -aG docker $USER && newgrp docker
```
<br>

For å starte minikube og gjøre installasjonen ferdig ved førstegangsinstallasjon:
```shell
minikube start --driver=docker
```
<br>

For å installere kubectl må denne kommandoen skrives:
```shell
minikube kubectl -- get pods -A
```