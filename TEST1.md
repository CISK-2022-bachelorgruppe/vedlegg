## Introduksjon
Dette er en veildedning for å eksportere en database-tabell i podden mysql til host-maskinen

### FOR MOUNTING:
```
minikube mount /home/ole-k/Desktop/mysql-container:/data/bachelor-mysql
```
<br>

### Eksportere sql fra pod til minikube 

```
docker exec some-mysql sh -c 'exec mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" bachelor_db' > /tmp/all-databases.sql
```
<br>

### Kopiere fil fra minikube til HOST-maskin
```
POD=$(kubectl get pod -l app=mysql -o jsonpath="{.items[0].metadata.name}")
kubectl cp default/$POD:/tmp/all-databases.sql /home/ole-k/Desktop/all-databases.sql
```

### Importere mysql-database på hostmaskinens mysql-server
```
sudo mysql -uroot < all-databases.sql
```



mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" bachelor_db > /tmp/all-databases.sql