# GlusterFS Deployment

- Glusterfs clusterini Bare Metal sunucularında veya Heketi kullanan containerlarda dağıtabiliriz. Dağıtımdan sonra bir GlusterFS volume oluşturacağız.

- Verileri saklamak istediğimiz yerde sunucuda aşağıdaki dizini oluşturalım.

~~~
# mkdir -p /data/brick1/myvol

Ardından, kullanacağımız volume oluşturacağız

# gluster volume create myvol replica 2 node1:/data/brick1/myvol node2:/data/brick1/myvol

# gluster volume start gv0

Şimdi içeriği gluster-endpoint.yaml adlı dosyaya ekleyerek kübernetler için çok amaçlı endpoints yaratmalıyız.
~~~

- gluster-endpoint.yaml

~~~
kind: Endpoints
apiVersion: v1
metadata:
 name: glusterfs - cluster
subsets:
 -addresses:
 -ip: 10.240 .106 .152
ports:
 -port: 1 - addresses:
 -ip: 10.240 .79 .157
ports:
 -port: 1
~~~

~~~
< # kubectl create -f gluster-endpoint.yaml

glusterfs-service.yaml dosyasına aşağıdaki içeriği ekleyerek kübernet'de gluster hizmetini oluşturalım

~~~

- glusterfs-service.yaml

~~~
kind: Service
apiVersion: v1
metadata:
  name: glusterfs-cluster
spec:
  ports:
  - port: 1
~~~

~~~
# kubectl create -f glusterfs-service.yaml

Sonra backend storage olarak gluster kullanarak Apache podunu başlatacağız ve apache-pod.yaml dosyasına aşağıdaki içeriği ekleyeceğiz
~~~

- apache-pod.yaml

~~~
apiVersion: v1
kind: Pod
metadata:
  name: glusterfs
spec:
  containers:
  - name: glusterfs
    image: apache
    volumeMounts:
    - mountPath: "/var/www/html"
      name: glusterfsvol
  volumes:
  - name: glusterfsvol
    glusterfs:
      endpoints: glusterfs-cluster
      path: kube_vol
      readOnly: true
~~~

- # kubectl create -f apache-pod.yaml