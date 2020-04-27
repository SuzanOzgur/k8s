# GCEPersistantDisk Deployment

- Önce kubernetes bölmesinde GCEPersistantDisk'i kullanmak için,

Üzerinde kübernet bölmelerinin çalıştığı düğümler GCE örnekleridir.
EC2 bulut sunucularının PD ile aynı GCE projesinde ve bölgesinde olması gerekir.
İlk olarak, gcepd-storage.yaml dosyasına aşağıdaki içeriği ekleyerek GCEPersistantDisk diski için kubernet'lerde depolama sınıfı oluşturmanız gerekir.

- gcepd-storage.yaml

~~~
kind: StorageClass
apiVersion: storage.k8s.io / v1beta1
metadata:
 name: fast
provisioner: kubernetes.io / gce - pd
parameters:
 type: pd - ssd
~~~

~~~
# kubectl create -f gcepd-storage.yaml

gcepd-pvc.yaml dosyasına aşağıdaki içeriği ekleyerek bir PVC oluşturacağız.
~~~

- gcepd-pvc.yaml

~~~
apiVersion: extensions / v1beta1
kind: Deployment
metadata:
 name: apache
spec:
 template:
 metadata:
 name: apache
labels:
 app: apache
spec:
 containers:
 -name: apache
image: bitnami / apache
ports:
 -containerPort: 80
volumeMounts:
 -mountPath: /opt/bitnami / apache / htdocs
name: apache - vol
volumes:
 -name: apache - vol
gcePersistentDisk:
 pdName: gce - disk
fsType: ext4
~~~

~~~
# kubectl create -f apache-gce.yaml

Kalıcı disk oluşturduktan sonra, aşağıdaki içeriği dosyaya ekleyerek web sunucusu bölmesi için web verilerini depolamak için kullanacağız.
~~~