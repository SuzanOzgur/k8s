# Ceph Deployment

- Ceph için mevcut bir ceph kümesine sahip olmamız gerekir. Ya Ceph kümesini Bare Metal'e yerleştirmeliyiz ya da Docker Container'ları kullanabiliriz. Ardından kubernetes hostuna ceph istemcisi kurulur.

- CephRBD için oluşturulan havuz için ayrı bir havuz ve kullanıcı yaratmamız gerekiyor.

~~~
Creating new/separate pool
# ceph osd pool create kube 200 200
Kube havuzuna tam erişime sahip bir kullanıcı oluşturma
# ceph auth get-or-create client.kubep mon ‘allow r’ osd ‘allow class-read object_prefix rbd_children, allow rwx pool=kube’ -o /etc/ceph/ceph.client.kube.keyring
Kullanıcı istemcisi için kimlik kümesinden kimlik doğrulama anahtarını alın.
# ceph –cluster ceph auth get-key client.kubep
Kubernet'lerde varsayılan ad alanında yeni bir secret oluşturma
# kubectl create secret generic ceph-secret-kube –type=”kubernetes.io/rbd” –from-literal=key=’AQBvPvNZwfPoIBAAN9EjWaou6S4iLVg/meA0YA==’ –namespace=default
kube-controller-manager depolama sağlama yetkisine sahip olmalı ve bunu yapmak için Ceph'den yönetici anahtarına ihtiyacı vardır. 
# sudo ceph –cluster ceph auth get-key client.admin

Kubernetes'te varsayılan ad alanında yönetici için yeni bir secret oluşturma
# kubectl create secret generic ceph-secret –type=”kubernetes.io/rbd” –from-literal=key=’AQAbM/NZAA0KHhAAdpCHwG62kE0zKGHnGybzgg==’ –namespace=ceph-storage

Secret ekledikten sonra ceph-rbd-storage.yml adlı dosyada aşağıdaki içeriği kopyalayarak yeni Storage Class tanımlamamız gerekir
~~~

- ceph-rbd-storage.yml

~~~
apiVersion: storage.k8s.io / v1
kind: StorageClass
metadata:
 name: rbd
provisioner: kubernetes.io / rbd
parameters:
 monitors: 192.168 .122 .110: 6789
adminID: admin
adminSecretName: ceph - secret
pool: kube
userId: kubep
userSecretName: ceph - secret - kube
fsType: ext4
imageFormat: "2"
imageFeatures: "layering"
~~~

~~~
# kubectl create -f ceph-rbd-storage.yml –namespace=ceph-storage
ceph-vc.yml adlı dosyada rbd StorageClass kullanarak bir volume oluşturalım
~~~

- ceph-vc.yml

~~~
apiVersion: v1
kind: PersistentVolume
metadata:
 name: apache - pv
namespace: ceph - storage
spec:
 capacity:
 storage: 100 Mi
accessModes:
 -ReadWriteMany
rbd:
 monitors:
 -192.168 .122 .110: 6789
pool: kube
image: myvol
user: admin
secretRef:
 name: ceph - secret
fsType: ext4
readOnly: false
~~~

~~~
# kubectl create -f ceph-vc.yml –namespace=ceph-storage
ceph-pvc.yml adlı dosyada rbd StorageClass kullanarak bir volume claim oluşturma 
~~~

- ceph-pvc.yml

~~~
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: apache-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      Storage: 500Mi
~~~

~~~
# kubectl create -f ceph-pvc.yml –namespace=ceph-storage
Şimdi talep edilen birimi kullanarak Apache podu başlatacağız. Aşağıdaki içeriğe sahip yeni bir dosya oluşturun
~~~

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
image: bitnami / apache
ports:
 -containerPort: 80
volumeMounts:
 -mountPath: /var/www / html
name: apache - vol
volumes:
 -name: apache - vol
persistentVolumeClaim:
 claimName: apache - pvc
~~~

~~~
# kubectl create -f apache-pod.yml –namespace=ceph-storage
CephFS için bir Ceph havuzu oluşturacağız

# ceph osd pool create cephfs_data 200

# ceph osd pool create cephfs_metadata 200

# ceph fs new newfs cephfs_metadata cephfs_data

Ceph admin kullanıcısı için kubernet'lerde varsayılan ad alanında yeni bir secret oluşturma.

# sudo ceph –cluster ceph auth get-key client.admin

# kubectl create secret generic ceph-secret –type=”kubernetes.io/rbd” –from-literal=key=’AQDkTeBZLDwlORAA6clp1vUBTGbaxaax/Mwpew==’ –namespace=default

Secret ekledikten sonra ceph-fs-storage.yml adlı dosyaya aşağıdaki içeriği kopyalamamız gerekir
~~~

- ceph-fs-storage.yml

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
name: mypvc
volumes:
 -name: mypvc
cephfs:
 monitors:
 -192.168 .100 .26: 6789
user: admin
secretRef:
 name: ceph - secret
~~~

