## BACKUP & RESTORE Mongodb Database ON KUBERNETES

Script que funciona correctamente
 - Sistema Operativo: Ubuntu 20.04
 - Necesario: kubectl y .kube/rsie_config file
 - Paquete: mongodb-database-tools-ubuntu2004-x86_64-100.7.3.deb

Script que funciona correctamente
 - Sistema Operativo: Redhat
 - Necesario: kubectl y .kube/rsie_config file
 - Paquete: mongodb-database-tools-x86_64 
 	Descargar mongodb-database-tools-rhel70-x86_64-100.7.3.rpm de https://www.mongodb.com/try/download/database-tools
 	Instalacion:
  ```bash
  sudo yum install -y mongodb-database-tools-rhel70-x86_64-100.7.3.rpm
  ```

# Codigo
```bash
#!/bin/bash
# Javier Martin Peña
#
#
unset MONGODB_ROOT_PASSWORD POD DATE DUMP_FILE BACKUP_DIR PID
export BACKUP_DIR="/msdp/pool1/k8s_backup/backup_mongodb"
export MONGODB_ROOT_PASSWORD=$(/usr/local/bin/kubectl get secret --namespace mongodb-system  mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
export POD=$(/usr/local/bin/kubectl get pods -n mongodb-system | tail -n 1 | awk '{print $1}')
export DATE=`date +"%Y%m%d_%H%M%S"`
export DUMP_FILE="$BACKUP_DIR/mongobackup_$DATE"
/usr/local/bin/kubectl port-forward -n mongodb-system $POD 27017:27017 &
export PID=$!
find $BACKUP_DIR -type f -name "*.gz" -ctime +30 | xargs rm -fr
mongodump -h 127.0.0.1:27017 -u root -p $MONGODB_ROOT_PASSWORD --gzip --archive=$DUMP_FILE.gz
kill -9 $PID
unset MONGODB_ROOT_PASSWORD POD DATE DUMP_FILE BACKUP_DIR PID
```

# Desarrollo

De esta forma se haria la copia en el propio POD de mongodb
export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace mongodb-system  mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
export POD=$(kubectl get pods -n mongodb-system | tail -n 1 | awk '{print $1}')
export DATE=`date +"%Y%m%d_%H%M%S"`
export DUMP_DIR=mongobackup_$DATE
kubectl exec -i -t -n mongodb-system $POD -- sh -c "mongodump -h 127.0.0.1:27017 -u root -p $MONGODB_ROOT_PASSWORD -o /tmp/$OUT_DIR"

Version 3.2 introduce gzip and archive option:
mongodump --db <yourdb> --gzip --archive=/path/to/archive

## PROCESO DE RESTAURACION

Then you can restore with:
mongorestore --gzip --archive=/path/to/archive
