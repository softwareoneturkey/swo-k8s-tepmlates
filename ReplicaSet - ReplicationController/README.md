
<p align="center">
  <img src="https://user-images.githubusercontent.com/55376595/133267785-c3f76dc4-bf27-489c-99d0-6d610d84d692.jpg"/>
</p>

Pod bölümünde konteyner imajlarını kullanarak pod'lar oluşturduk ve uygulamalar çalıştırdık  fakat dikkat ettiyseniz bu işlemlerin tamamını manuel olarak yaptık. Örneğin bir pod da problem olduğu zaman gidip manuel olarak pod'u silip tekrar çalıştırmak zorunda kaldık. Başka bir örnek bir işten bir pod çalıştırdık , bizim uygulamamız 3 pod ile çalışacaksa gidip 3 ayrı pod yaml dosyası mı yazacağız ? Kubernetes pod'u kontrol edip otomatik olarak pod'un birine bir şey olduğunda yerine yenisini oluşturamıyor mu veya bizim belirlediğimiz kadar pod'u ayakta tutamıyor mu ? 

Kubernetes controller'lar ile sadece pod'ları değil node'lar veya başka kubernetes objelerini de izleyerek her hangi bir aksama durumunda sistemi istenilen durumlarda çalıştırabilmeyi sağlar. Ne demek bu ? Örneğin biz bir senaryomuz için 3 pod'un sürekli olarak çalışmasını istiyoruz. Replication controller ve replicaset objeleriyle biz pod'umuzu entegre ettiğimizde bu objeler bizim yerimize artık sürekli olarak 3 pod'un hayatta kalmasını sağlar.

##
Replication Controller yaml dosyasını inceleyelim ve neler oluyor görelim : 

```yaml
apiVersion: v1 ## --> v1 olarak görmekteyiz. Burası bize oluşturulacak olan objenin hangi kubernetes versiyonunda geldiğini gösteriyordu. 
kind: ReplicationController ## --> olarak görmekteyiz . Burası bize hangi tipte objenin oluşturulacağını gösteriyordu. 
metadata: ##--> Oluşturacağımız objeyi tanımlayacağımız özellikler verdiğimiz bölüm.
  name: myapp-replicationcontroller ##--> Burada oluşturmuş olduğumuz ReplicationController objemizin isminin ne olacağını belirtmekteyiz.
  labels:
    app: rc-app ##Burada ReplicationController objemize etiket atamaktayız, bunu label and selector dosyasında daha detaylı anlayacağız fakat kısaca kubernetes içinde bir obje "myapp-replicationcontroller" ReplicationController objesini aradığında app:rc-app olarak da filtreleme yapabilmesini sağlar.
spec:
  replicas: 3 ##--> ReplicationController objesinin en önemli alanı "replicas" alanıdır. Burada ReplicationController'in kaç pod oluşturacağının ve takip edeceğinin sayısını     vermekteyiz.
  template: ## --> ReplicationController ile istediğimiz kadar aynı pod'un kopyasından oluşturuyoruz fakat burda pod oluştururken farklılıklar var. Pod bölümünde pod'a ait özellikler vermiştik  apiVersion , kind gibi. Bu iki özelliği vermiyoruz , ReplicationController template altında otomatik olarak bu özellikleri alıyor ve pod oluşturuyor.
    metadata: ## --> Template altında oluşturacağımız pod'un özelliklerini gördiğimiz alan . Burada name özelliğini vermiyoruz , ReplicationController kendi ismine bağlı olarak random devam eden isimlerle podları oluşturuyor. 
      labels:
        app: rc-app-pod ## --> yukarıdaki labels'le aynı işe yaramaktadır. Burada Oluşturulacak olan pod'lara kubernetes üzerindeki başka objelerin eklemiş olduğumuz ek etiketler sayesinde kolayca erişebilmesini sağlıyoruz. 
    spec: ## --> spec altında ise pod'un hangi imajı kullanacağını tanımlıyoruz. İleride göreceğimiz komponentler sayesinde sadece imaj tanımlanmadığını göreceğiz fakat şimdinin konusu değil. 
      containers:
        - name: nginxcontainer
          image: nginx
```

Özetle ; ReplicationController objesi oluşturuyoruz ve diyoruz ki ; 
        nginx imajından bir konteyner oluştur. Bunun adı  nginxcontainer olsun. Bu nginxcontainer i oluşturacağın pod'ların içinde çalıştır. Oluşturacağın pod'lara label ata ve app: rc-app-pod etiketiyle bu pod'lara erişilebilsin.  Toplam 3 pod oluştur ve bunların takibini yap. 
		
Şimdi yaml dosyamızı çalıştıralım ve neler olacak inceleyelim : 

```bash
kubectl apply -f replication_controller.yaml
```
--> komutuyla yaml dosyasından objenin oluşturması için talimat verdik. 

```bash
kubectl get all -o wide --show-labels
```
--> komutunu çalıştıralım. -o wide daha detaylı oluşturulan objelerin özelliklerini bize getirir. --show-labels ise oluşturmuş olduğumuz objelere atamış olduğumuz label'ları getirir.

Yukarıdaki komutu çalıştırdıktan sonra aşağıdakine benzer sonuçlar göreceğiz . 


    |                  Name                   | Ready | STATUS  | RESTARTS | AGE | 	IP      |    NODE  | NOMINATED NODE | READINESS	|      LABELS    |
    | --------------------------------------- | ----- | ------- | -------- | --- | ------------ | -------- | -------------- | --------- | -------------- |
    | pod/myapp-replicationcontroller-2txrb   | 1/1   | Running |    0     | 15s | 10.200.1.112 | worker-1 |    <none>      |  <none>   | app=rc-app-pod |
    | pod/myapp-replicationcontroller-bfqxd   | 1/1   | Running |    0     | 15s | 10.200.1.113 | worker-1 |    <none>      |  <none>   | app=rc-app-pod |
    | pod/myapp-replicationcontroller-kmk6m   | 1/1   | Running |    0     | 15s | 10.200.1.114 | worker-0 |    <none>      |  <none>   | app=rc-app-pod |
    
    |                             Name                    | DESIRED | CURRENT  | READY | AGE | 	CONTAINERS    | IMAGES | SELECTOR      |      LABELS    |
    | --------------------------------------------------- | ------- | -------- | ----- | --- | -------------- | ------ | ------------- | -------------- |
    | replicationcontroller/myapp-replicationcontroller   |   3     |    3     |   3   | 15s | nginxcontainer | nginx  | pp=rc-app-pod | app=rc-app-pod |

Burada alttaki name sütununun altına baktığımızda "replicationcontroller/myapp-replicationcontroller" görmekteyiz. / dan önceki replicationcontroller , objenin tipini bize vermekte , bir üstte de pod yazmakta mesela , oradaki objelerin de pod olduğunu anlıyoruz.   
Desired --> burada 3 rakamını görmekteyiz , 3 bize kaç tane pod replikası oluşturmak istediğimizi söylüyor , yaml dosyasında replicas'a 3 yazmıştık. 
Current --> şuanda çalışan pod sayısını göstermekte ,
Ready --> çalışan pod'lardan kaç tanesi hazır durumda onu gösterir . 
Containers --> yaml dosyasında imajdan oluşturduğumuzda konteyner isminin ne olcağını belirtmiştik , o isimle konteyner'lar oluşmuş duurmda.
images --> yaml dosyasında hangi imajın kullanılacağını göstermiştik , nginx'den oluşmuş durumda.
Selector : app=rc-app-pod isimli pod'ları seçtiğini gösteriyor.
Labels : app=rc-app --> bu da bize replication controller objemize bağlanacak olan başka objeler erişmek istediğinde app=rc-app-pod etiketiyle erişebileceğini gösteriyor.

Üstteki name sütununu inceleyecek olursak : 

pod/ dan sonrasına baktığımızda bizim replicationcontroller objemize verdiğimiz isin aynıyna başladığını sonunun sadece random değerler aldığını görmekteyiz. Yaml dosyasını incelerken bahsetmiştik , biz isim vermeyiz , kendine bağlı isimlerle objeleri oluşturur.

Örneğin biz bir pod'u silmek istedik ne olacak ? 

```bash
kubectl delete pod -f myapp-replicationcontroller-2txrb 
```
--> komutuyla pod'u silelim 



    |                  Name                   | Ready | STATUS      | RESTARTS | AGE | 	IP          |    NODE  | NOMINATED NODE | READINESS |      LABELS    |
    | --------------------------------------- | ----- | ----------- | -------- | --- | ------------ | -------- | -------------- | --------- | -------------- |
    | pod/myapp-replicationcontroller-2txrb   | 1/1   | Terminating |    0     | 15s | 10.200.1.112 | worker-1 |    <none>      |  <none>   | app=rc-app-pod |
    | pod/myapp-replicationcontroller-bfqxd   | 1/1   | Running     |    0     | 15s | 10.200.1.113 | worker-1 |    <none>      |  <none>   | app=rc-app-pod |
    | pod/myapp-replicationcontroller-kmk6m   | 1/1   | Running     |    0     | 15s | 10.200.1.114 | worker-0 |    <none>      |  <none>   | app=rc-app-pod |
    | pod/myapp-replicationcontroller-948xz   | 1/1   | Running     |    0     | 15s | 10.200.1.115 | worker-0 |    <none>      |  <none>   | app=rc-app-pod |
    
    |                             Name                    | DESIRED | CURRENT  | READY | AGE | 	CONTAINERS    | IMAGES | SELECTOR      |      LABELS    |
    | --------------------------------------------------- | ------- | -------- | ----- | --- | -------------- | ------ | ------------- | -------------- |
    | replicationcontroller/myapp-replicationcontroller   |   3     |    3     |   3   | 15s | nginxcontainer | nginx  | pp=rc-app-pod | app=rc-app-pod |


myapp-replicationcontroller-2txrb --> pod ' u silinirken  pod/myapp-replicationcontroller-948xz --> pod'unun running duruma geçtiğini göreceğiz.  Replication Controller istenilen durum yani 3 pod'un sürekli olarak ayakta kalması durumunu bizim için sağladı ve bir pod' da problem olduğu anda anında aynı pod'dan yine oluşturarak çalışabilir hale getirdi.

##

Temel olarak Replication Controller ve ReplicaSet objeleri aynı işi yapmakla görevlidir. ReplicaSet objesi apps/v1 versiyonuyla birlikte gelmiş olup eşleşeceği podları özelleştirebileceğimiz hale gelmiştir. 

Replicaset.yaml dosyasını inceleyelim : 

```yaml
apiVersion: apps/v1
kind: ReplicaSet ##  --> replicaset objesi oluşturabilmek için kind alanında bu şekilde yazmak gerekmektedir.
metadata:
  name: myapp-replicaset
  labels:
    app: rs-app
    type: front-end
spec:
  replicas: 3

  selector: ## --> bu alan eklenerek çalışan onlarca pod içerisinde takip edeceği pod'ları daha çok detaylandırarak sağlıklı çalışabilmesini sağlar. Örneğin type: front-end tipinde başka pod'lar da olabilir , biz matchLabels kısmına örneğin  app: maximo ' da dediğimizde hem front-end hem de app: maximo olan uygulamayı takip ediyor hale gelecektir. 
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

Replication controller artık yerini replicaset'lere bırakmış durumdadır.

```bash
kubectl get replicationcontroller
```

```bash
kubectl get rc
```
--> komutları replicationcontroller' ları listeler.

```bash
kubectl get replicaset
```

```bash
kubectl get rs 
```

--> komutları replicaset' leri listeler.

* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/ReplicaSet%20-%20ReplicationController) 