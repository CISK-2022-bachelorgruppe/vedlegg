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

Da ble kommandoene under kjørt:
```shell
$ cd $HOME/git/CISK-2022-bachelorgruppe/applikasjoner/python-script-get
$ ./test-gjennomføring $(minikube ip) 30001 200 10 $HOME/git sj $HOME/git
```

Dette er et script som vil gjøre hele testen selv. Det har tre `for`-løkker inne i hverandre som hører til antall pods, antall tråder og antall gjennomføringer. Nedenfor er et utkast av de viktigste linjene fra scriptet.

```shell
for pod in {1..10}
    do
        minikube kubectl -- scale --replicas=$pod -f $path/k8s-bachelor/k8s-config/django/django-deployment.yaml
        sleep 12
        for z in {1..10}
        do
            fil="$path/bachelor-applikasjon/python-script-get/resultater/$tid/$pod-podder.$z-trader-$tid.txt"
            for i in $(seq 1 $gjennomforinger)
            do
                /usr/bin/time -a -o "$fil" -f "%E" nice bash -c "python3 $path/bachelor-applikasjon/python-script-get/req.py --host $host --port $port --antall $foresporsler --thr $z"
                
                if [ $i != 10 ]; then
                    sleep 12
                else
                    echo "Test med $z tråd(er) ferdig!"
                fi
            done
        done
    done
```
De viktigste linjene å legge merke til her er linje X og linje Y. Linje X skalerer opp antall podder mellom hver 100 gjennomføring av req.py scriptet mens linje Y gjør selve testen ved å ta tiden på hvor lenge det tar å gjennomføre 200 forespørsler med `$z` tråder mot `$pod` podder.

Nedenfor vises et utdrag av python scriptet. Som vist vil alle trådene sammarbeide om å nå 200 forespørsler. Rorespørslene fordeles ikke nødvendigvis likt utover trådene, men neste forespørsel blir delt ut til neste tråd som er ledig.
De siste forespørslene er det `master`tråden som utfører selv for å sikre at det blir sendt akkuratt 200 forespørsler.

```python
def foresporsel(host, port, antall, thr):
    global value
    while value < antall-thr:
        r = requests.get(f"http://{host}:{port}")
        if r.status_code == 200:
            with threadLock:
                value += 1
            if value % 50 == 0:
                print(f"Det er gjennomført {value}/{antall} tester.")
                
value = 0
threads = []
for i in range(args.thr):
    t = threading.Thread(target=foresporsel, args=(args.host, args.port, args.antall, args.thr))
    threads.append(t)
    t.start()
```
```python
while value < args.antall:
        r = requests.get(f"http://{args.host}:{args.port}")
        if r.status_code == 200:
            with threadLock:
                value += 1
            if value % 50 == 0:
                print(f"Det er gjennomført {value}/{args.antall} tester.")
```
