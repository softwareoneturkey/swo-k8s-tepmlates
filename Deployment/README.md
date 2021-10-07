
<p align="center">
  <img src="https://user-images.githubusercontent.com/55376595/133255923-22d3cfe6-77b3-4ecd-9a94-17aece2f2290.jpg"/>
</p>

Daha önce Pod ve ReplicaSet objelerini incelemiştik. Örneğin bir replicaset içerisinde pod dosyası hazırladık ve koyduk. Daha sonra bu pod dosyasında bir değişiklik oldu  ve yeni versiyona geçmek istiyoruz. Yeni versiyonu ReplicaSet dosyasında hazırlayıp kubectl apply -f rs.yaml komutuyla canlıya aldığımızda tüm podlar yeni versiyonda çalışmaya başlar. Peki yeni versiyonda bir problem var ve eski versiyona geçmemiz lazım , o zaman ne yapacağız ? Eğer biz bu işlemi ReplicaSet ile yapacak olursak eski versiyondaki yaml dosyasını tekrar hazırlayıp yüklememiz gerekiyor. Replicaset ile versiyn yönetimi yapamıyoruz fakat bunun için kubernetes bize deployment objesini veriyor.

Deployment objesi ile versiyon geçişlerimizi kolaylıkla yapabilir , rollback işlemi gerektiği anda çok basit bir şekilde eski versiyona geçebiliriz.

Deployment objesini template olarak inceleyelim :

```yaml
apiVersion: apps/v1  ## apps/v1 api versiyonunda gelmekte
kind: Deployment
metadata:
  name: deploymenttemplate
  labels:
    team: development
spec:
  replicas: 3 ## 3 replika pod çalıştıracak
  selector:
    matchLabels:
       team: development ## label'ı team:developlemt olan pod ile eşleşecek.
  template:
```
yaml dosyası replicaset ile neredeyse aynıdır. Deployment'ı oluşturduğumuz anda , hem deployment objesi oluşturulur , hem deployment objesine bağlı olarak replicaset objesi oluşturulur hem de replica sayısı kadar pod oluşturulur.



<p align="center">
  <img src="https://user-images.githubusercontent.com/55376595/133255702-974c21f2-1294-4f8c-afe1-f89de8d47eec.png"/>
</p>

Örneğin nginx 1.7.0 versiyonunu kullanan 10 tane pod  olsun. Nginx 1.7.1 versiyonuna geçiş yapacağız. Deployment objesi versiyon geçişi yaparken 2 farklı stratejiyle bu geçişi sağlar. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/55376595/133255683-5ad88345-6b5e-479c-b449-851c07bf8cee.png"/>
</p>	
-     Rolling Update : 
   	
	Bu strateji tipinde oluşturulan pod'lar sırayla yeni versiyona geçiş yapar. Eski versiyondan bir pod ölür ve yeni versiyondan bir pod ayağa kalkar. Tüm podların yeni versiyona geçişi tamamlanana kadar bu işlem böyle devam eder. Uygulama geçiş sırasında kesintiye uğramadan hem eski versiyonda hem yeni versiyonda hizmet vermeye devam eder. Böylece kullanıcılara kesinti vermeden versiyon geçişi tamamlanır. Kubernetes'in default geçişi " Rolling Update' dir . 
	
	Rolling Update yaml dosyasını inceleyelim : 
	
```yaml
## deployment_rolling_update.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rollingupdate
  labels:
    app: siebel
spec:
  replicas: 10
  selector:
    matchLabels:
      app: siebel
  strategy:
    type: RollingUpdate ### strategy altındaki type key'i ile strateji tipi belirlenir.
    rollingUpdate:
      maxUnavailable: 1 ## eski versiyondan aynı anda kaç pod öldürülecek parametresi burada belirlenir
      maxSurge: 1 ## yeni versiyondan aynı anda kaç pod ayağa kaldırılacak parametresi burada belirlenir.
  template:
    metadata:
      labels:
        app: siebel
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
ikinci bir terminal açıp watch kubectl get all -o wide komutunu çalıştırdık.
	
kubectl apply -f deployment_rolling_update.yaml -->  dosyasını çalıştırdık , ilk versiyonumuza ait deployment ve repicaset ve pod objelerimiz oluştu. Daha sonra yaml dosyasındaki image alanını nginx:latest olarak degistirip kaydettik.

Daha sonra kubectl apply -f deployment_rolling_update.yaml --> komutunu tekrar çalıştırdığımızda pod'ların sırasıyla ölüp yeni versiyonunun ayağa kalktığını göreceğiz.
	

-  Recerate :
Bu strateji tipinde ise eski versiyondaki podların tamamı ölür. Tüm pod'lar öldükten sonra yeni versiyondaki pod'lar ayağa kalkar. Bu süre içerisinde uygulamada kesinti verilmiş olur.
asd
	
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rcdeployment
  labels:
    team: development
spec:
  replicas: 3
  selector:
    matchLabels:
      app: recreate
  strategy:
    type: Recreate ## deployment stratejisi recreate olarak ayarlandı.
  template:
    metadata:
      labels:
        app: recreate
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
ikinci bir terminal açıp watch kubectl get all -o wide komutunu çalıştırdık.
	
kubectl apply -f deployment_recreate.yaml -->  dosyasını çalıştırdık , ilk versiyonumuza ait deployment ve repicaset ve pod objelerimiz oluştu. Daha sonra yaml dosyasındaki image alanını nginx:latest olarak degistirip kaydettik.

Daha sonra kubectl apply -f deployment_recreate.yaml --> komutunu tekrar çalıştırdığımızda eski pod'ların tamamının öldükten sonra yeni pod'ların toplu bir şekilde ayağa kalktığını göreceğiz 


- Komutlar
	kubectl get deployments --> deployment'ları listeler. 
	
	kubectl delete deployment deployment_name --> deployment ve ona bağlı olan replicaset ile podların tamamını siler
	
	kubectl rollout undo deployment deployment_name --> deployment'da yapılan son değişikliğin geri alınmasını eski versiyona geçilebilmesini sağlar. 
	
	kubectl rollout history deployment deployment_name --> deployment'da yapılan değişikliklerin listelenmesini sağlar.
	
	kubectl rollout status deployment deployment_name --> deployment'da yapılan değişikliklerin izlenmesini sağlar.
	
	kubectl rollout puse deployment deployment_name --> deployment üstünde yapılan değişikliklerin durdurulmasını sağlar. 

	kubectl rollout resume deployment deployment_name --> Durdurulan rollout'un devam ettirilmesini sağlar. 

* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/ReplicaSet%20-%20ReplicationController) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Label%20and%20Selectors) 
