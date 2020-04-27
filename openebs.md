# OpenEBS Deployment

- İlk olarak, OpenEBS Hizmetlerini Operatör kullanarak başlatacağız

~~~
# kubectl create -f https://github.com/openebs/openebs/blob/master/k8s/openebs-operator.yaml

Ardından, bazı default storage classes oluşturacağız

# kubectl create -f https://raw.githubusercontent.com/openebs/openebs/master/k8s/openebs-storageclasses.yaml

Şimdi demo-openebs-jupyter.yaml adlı dosyaya aşağıdaki içeriği ekleyerek jupyter'ı OpenEBS kalıcı birimiyle başlatacağız
~~~

- demo-openebs-jupyter.yaml

~~~
apiVersion: apps
kind: Deployment
metadata:
 name: jupyter - server
namespace: default
spec:
 replicas: 1
template:
 metadata:
 labels:
 name: jupyter - server
spec:
 containers:
 -name: jupyter - server
imagePullPolicy: Always
image: satyamz / docker - jupyter: v0 .4
ports:
 -containerPort: 8888
env:
 -name: GIT_REPO
value: https: //github.com/vharsh/plot-demo.git
 volumeMounts:
 -name: data - vol
mountPath: /mnt/data
volumes:
 -name: data - vol
persistentVolumeClaim:
 claimName: jupyter - data - vol - claim
 -- -
 kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 name: jupyter - data - vol - claim
spec:
 storageClassName: openebs - jupyter
accessModes:
 -ReadWriteOnce
resources:
 requests:
 storage: 5 G
 -- -
 apiVersion: v1
kind: Service
metadata:
 name: jupyter - service
spec:
 ports:
 -name: ui
port: 8888
nodePort: 32424
protocol: TCP
selector:
 name: jupyter - server
sessionAffinity: None
type: NodePort
~~~

- # kubectl create -f demo-openebs-juypter.yaml

- Bu, jvyter kapsülü için pv ve pvc'yi otomatik olarak sağlayacaktır.