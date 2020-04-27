# Azure Disk Deployment

- Azure diskini kubernet bölmeleri için kalıcı depolama olarak kullanmak için. sc-azure.yaml adlı dosyaya aşağıdaki içeriği ekleyerek depolama sınıfı oluşturmalıyız

- sc-azure.yaml

~~~
kind: StorageClass
apiVersion: storage.k8s.io / v1beta1
metadata:
 name: slow
provisioner: kubernetes.io / azure - disk
parameters:
 skuName: Standard_LRS
location: eastus
~~~

~~~
# kubectl create -f sc-azure.yaml

Depolama sınıfı oluşturduktan sonra, azure-pvc.yaml dosyasına aşağıdaki içeriği ekleyerek PVC oluşturacağız.
~~~

- azure-pvc.yaml

~~~
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 name: azure - disk - storage
annotations:
 volume.beta.kubernetes.io / storage - class: slow
spec:
 accessModes:
 -ReadWriteOnce
resources:
 requests:
 storage: 10 Gi
~~~

~~~
# kubectl create -f azure-pvc.yaml

Şimdi, aşağıdaki içeriği apache-web-azure.yaml ekleyerek AZURE DISK ile kalıcı depolama alanı olarak Apache-web sunucusu bölmesini başlatacağız.
~~~

- apache-web-azure.yaml

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
name: azure - disk
volumes:
 -name: azure - disk
persistentVolumeClaim:
 claimName: azure - disk - storage
~~~

- # kubectl create -f apache-web-azure.yaml