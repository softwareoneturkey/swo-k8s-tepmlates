
<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136307241-a23d74b2-28b7-4007-bb5a-adc65d82801d.png"/>
</p>

Önceki bölümlerde Kubernetes objelerinin bazılarını oluşturduk. Pod , Replicaset , Deployment gibi objeleri inceledik. Kubernetes için bunların hepsi farklı objelerdir. Zamanla Cluster içinde bu objelerden yüzlerce veya binlercesi olabilir. Kubernetes objelerinin sayısı bu derece arttıktan sonra bu objeleri gruplamak , farklı kategorilere göre filtrelemek ve görüntülemek için de bir yola ihtiyacımız olacaktır. Bu noktada  label ve selector'ler bizlere yardımcı olacaktır. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136307311-3498f0aa-498a-4eca-8cfe-32f8097924d3.png"/>
</p>

Yukarıdaki resme baktığımızda pod'a ek özellikler eklendiğini görmekteyiz. Bu eklenen ek özellikler sayesinde biz pod2ların arasından app:Firstapp olan podların tamamını getirebiliriz veya type:Frontend olan tüm podları getirebiliriz veya app:Firstapp ve type:Backend olan tüm podları getir diyebiliriz. Eklediğimiz bu ek özellikler sayesinde objelerimizde filtrelemeler yapabilir , birbirleri ile eşleşmelerini sağlayabiliriz. 
Örneğin Servis objesini deployment objesi ile bağlarken ip adresi yerine deloyment objesine eklediğimiz label'ları hedef göstererek servis objesinin matchLabels özelliği ile deployment objesine bağlayabiliriz. Böylece Deployment objesinin ip adresi değişse bile servis objesi bundan etkilenmeden tekrar deployment objesine otmatik olarak bağlanabilecektir. 


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: podnginx1
spec:
  containers:
  - name: nginx
    image: nginx

```
Yukarıdaki yaml dosyası label eklenmemiş düz pod oluşturur.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: rs-app
    type: front-end
spec:
  replicas: 3

  selector:
    matchLabels:
      type: front-end

  template:
    metadata:
      labels:
        app: rs-app
        type: front-end
    spec:
      containers:
        - name: nginx
          image: nginx

```
Yukarıdaki yaml dosyasında ise pod objesine iki tane ekstra özellik tanımlanmış durumda. app:rs-app ve type:front-end özellikleri eklenerek pod'a erişilmek istendiği zaman bu etiketlerle erişilebilir hale getirilmiş durumda. 

replicaset objesinin MatchLabels özelliğine dikkat ederseniz type:front-end yazmakta . Bunun anlamı , type:front-end olan pod ile eşleleceksin demektir.

yine replicaset objesinin metadata altındaki  labels alanına bakacak olursanız app:rs-app ve type:front-end özellikleri tanımlanmış durumda. Eğer başka bir kubernetes objesi bu replicaset ile eşleşmek isterse bu iki özelliği kullanabilir. Örneğin Deployment objesi bu özelliklerle replicaset ile eşleşebilir. 

Bu şekilde tüm kubernetes objelerine özellikler tanımlanabilir. ( Node , Pod , Replicaset, Deployment , Service ...)

--> imperative olarak label ekleme: 

```bash
kubectl label "object_type" "object_name" "key=value"
```
```bash
kubectl label pods nginxpod app=FirstApp
```
--> imperative olarak label Kaldırma:

```bash
kubectl label "object_type" "object_name" "key-"
```
```bash
kubectl label pods nginxpod app-
```
--> imperative olarak label guncelleme
```bash
kubectl label --overwrite "object_type" "object_name" "key=value"
```
```bash
kubectl label --overwrite pods nginxpod app=SecondApp
```

--> Bir namespace'deki tüm objelere toplu label ekleme
```bash
kubectl label --overwrite "object_type" --all  "key=value"
```
```bash
kubectl label --all  env=test
```

--> Objeleri etiketleriyle birlikte görüntüleme
```bash
kubectl get "object_type" --show-labels
```
```bash
kubectl get pods --show-labels
```


```bash
$ kubectl get "object_type" -l "key:value"

Ör: kubectl get pods -l "app=firstapp,tier=frontend" --show-labels
Ör: kubectl get pods -l "app=firstapp,tier!=frontend" --show-labels
Ör: kubectl get pods -l "app,tier=frontend" --show-labels
Ör: kubectl get pods -l 'app in (firstapp)' --show-labels
Ör: kubectl get pods -l "app=firstapp,app=secondapp" --show-labels
Ör: kubectl get pods -l 'app in (firstapp,secondapp)' --show-labels
Ör: kubectl get pods -l 'app notin (firstapp)' --show-labels
Ör: kubectl get pods -l 'app,app notin (firstapp)' --show-labels
Ör: kubectl get pods -l '!app' --show-labels
Ör: kubectl get pods -l 'app in (firstapp),tier notin (frontend)' --show-labels
```

* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Deployment) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Namespace) 
