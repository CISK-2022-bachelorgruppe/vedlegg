# 1. Introduksjon

Vedlegg E er en detaljert beskrivelse av både kode og gjennomføring til test 2.

Denne testen brukte den egenutviklede applikasjonen

# 2. Programvare

Programvarer utover det som er beskrevet i oppgaven.

| Programvare                   | Versjon   |
|-------------------------------|-----------|
| Python                        | 3.9.7     |
| Requests (python pakke)       | 2.25.1   |
| pip3                          | 20.3.4    |

# 3. Oppsett

Som i test 1 slettes eventuelle tidligere minikube clustere for å sikre at det ikke ligger tidligere konfigurasjon som kan påvikre testingen. Deretter lages et nytt med docker som driver.

```shell
$ minikube delete
$ minikube start --driver=docker
```

Deretter ble django og mysql fra den egenutviklede applikasjonen satt opp i det nye clusteret. Oppsettet av testen fortsatte ikke før siste kommando i kodeboksen under viste at begge podder hadde "Running" status.

```shell
$ cd $HOME/git/kubernetes-config/k8s-config
$ minikube kubectl -- apply -k ./django/ && minikube kubectl -- apply -k ./mysql/
$ minikube kubectl -- get pods --watch
```

Deretter ble NodePort til `django-entrypoint` testet med kommandoen

```shell
$ curl $(minikube ip):30001
```

Når kommandoen over returnerte en HTML side er oppsettet ferdig og testen er klar for å utføres.

Da ble kommandoen under kjørt:
```shell
$ ./test-gjennomføring $(minikube ip) 30001 200 10 $HOME/git sj $HOME/git
```
