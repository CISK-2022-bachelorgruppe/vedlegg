# Introduksjon
I dette vedlegget vil grunnkonfigurasjonen som er utført på lab-pcene gjennomgås. Dette er for
å kunne få et likt oppsett ved en senere anledning, og for å kunne ha likt utgangspunkt for testing.

# Installasjon av programvare
I basiskonfigurasjonen ligger det hovedsakelig to programmer. Dette er Docker og Minikube.
De neste kapitlene vil gjennomgå de prosedyrene som ble utført ved installasjon på lab-pcene
som ble benyttet i prosjektet.

## Docker
For installasjon av Docker, ble [installasjonsguiden](https://docs.docker.com/engine/install/ubuntu/) til Docker Inc fulgt.

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## Minikube
Programvare Versjon
Minikube v1.25.2, Debian 11.0
Kubernetes v1.23.3