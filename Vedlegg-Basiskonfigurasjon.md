# Introduksjon
I dette vedlegget vil grunnkonfigurasjonen som er utført på lab-pcene gjennomgås. Dette er for
å kunne få et likt oppsett ved en senere anledning, og for å kunne ha likt utgangspunkt for testing.

# Installasjon av programvare
I basiskonfigurasjonen ligger det hovedsakelig to programmer. Dette er Docker og Minikube.
De neste kapitlene vil gjennomgå de prosedyrene som ble utført ved installasjon på lab-pcene
som ble benyttet i prosjektet.

## Docker
For installasjon av Docker, ble [installasjonsguiden](https://docs.docker.com/engine/install/ubuntu/) til Docker Inc fulgt.  
Under vil kommandoene som ble benyttet i installasjonen bli listet opp.

Først ble all gammel konfigurasjon av Docker fjernet med følgende kommando:
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
<br>

Så ble Docker installert med et _repository_. Med de neste kommandoene oppdateres og installeres det pakker som gjør det mulig å tillate _repositories_ over HTTPS.
```
sudo apt-get update

 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
<br>

Her legges Docker Inc sin offisielle GPG-nøkkel til PC-en:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
<br>

Så kjøres en kommando for å sette opp et stabilt _repository_ og gjør det klart til innstallering:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
<br>

Deretter installeres siste versjon av Docker Engine med følgende kommandoer:  
> MERK: _Skal testen etterprøves, bør det spesifiseres hvilken versjon som skal lastes ned her, for å få samme versjon som ble benyttet under forsøket i dette prosjektet. Dette er for å skape et likt testmiljø_
```
 sudo apt-get update

 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
<br>
Dette er det som må til for å installere Docker på PC-en. 


<br>
<br>



## Minikube
Programvare Versjon
Minikube v1.25.2, Debian 11.0
Kubernetes v1.23.3