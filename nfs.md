# NFS Deployment

- NFS sunucusunun Kubernetes bölmesi tarafından tüketilmesi için. İlk olarak, nfs-pv.yaml dosyasına aşağıdaki içeriği ekleyerek kalıcı bir hacim oluşturacağız.

- nfs-pv.yaml

~~~
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv 
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany 
  persistentVolumeReclaimPolicy: Retain 
  nfs: 
    path: /data/nfs 
    server: nfs.server1
    readOnly: false
~~~

~~~
# kubectl create -f nfs.pv.yaml

nfs-pvc.yaml dosyasına aşağıdaki içeriği ekleyerek kalıcı hacim talebi oluşturalım
~~~

- nfs-pvc.yaml

~~~
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: nfs - pvc
spec:
 accessModes:
 -ReadWriteMany
resources:
 requests:
 storage: 10 Gi
~~~

~~~
# kubectl create -f nfs-pvc.yaml

Şimdi apache-server.yaml dosyasına aşağıdaki içeriği ekleyerek NFS kalıcı birimiyle web sunucusu bölmesini başlatacağız.
~~~

- apache-server.yaml

~~~
apiVersion: v1
kind: ReplicationController
metadata:
 name: apache
spec:
 replicas: 1
selector:
 app: apache
template:
 metadata:
 name: apache
labels:
 app: apache
spec:
 containers:
 -name: apache
image: apache
ports:
 -containerPort: 80
volumeMounts:
 -mountPath: /var/www / html
name: nfs - vol
volumes:
 -name: nfs - vol
persistentVolumeClaim:
 claimName: nfs - pvc
~~~

- # kubectl create -f apache-server.yaml
