# AWS EBS Deployment

- Önce Kubernetes bölmesinde Amazon EBS kullanmak için,

- Üzerinde kübernet bölmelerinin çalıştığı düğümler Amazon EC2 örnekleridir.
- EC2 bulut sunucularının EBS ile aynı bölgede ve AZ konumunda olması gerekir.
- İlk olarak, awsebs-storage.yaml dosyasına aşağıdaki içeriği ekleyerek EBS diski için kubernetes'te depolama sınıfı oluşturmalıyız

- awsebs-storage.yaml

~~~
kind: StorageClass
apiVersion: storage.k8s.io / v1
metadata:
 name: slow
provisioner: kubernetes.io / aws - ebs
parameters:
 type: io1
zones: us - east - 1 d, us - east - 1 c
iopsPerGB: "10"
~~~

~~~
# kubectl create -f aws-ebs-storage.yaml

Sonra aws-ebs.yaml dosyasına aşağıdaki içeriği ekleyerek PVC oluşturacağız
~~~

- aws-ebs.yaml

~~~
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 name: aws - ebs
annotations:
 volume.beta.kubernetes.io / storage - class: standard

spec:
 accessModes:
 -ReadWriteOnce
resources:
 requests:
 storage: 10 Gi
storageClassName: io1
~~~

~~~
# kubectl create -f aws-ebs.yaml

Şimdi, aşağıdaki içeriği apache-web-ebs.yaml ekleyerek AWS EBS ile kalıcı depolama alanı olarak apache-webserver bölmesini başlatacağız
~~~

- apache-web-ebs.yaml

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
name: aws - ebs - storage
volumes:
 -name: aws - ebs - storage
persistentVolumeClaim:
 claimName: aws - ebs
~~~

- # kubectl create -f apache-web-ebs.yaml