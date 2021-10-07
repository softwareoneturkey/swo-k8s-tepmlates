Kubernetes Servis Objesi 

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136458267-948bbf01-8fdc-410f-8e89-2ddaa02cba85.png"/>
</p>

Önceki bölümlerde kubernetes'e ait objeleri gördük pod, replicaset ve deployment gibi objeler oluşturduk. Bu objeler sayesinde çeşitli imajları kullanarak uygulamalar ayağa kaldırdık fakat bu uygulamaları hiç birbiri ile bağlamadık veya uygulamaları canlıda çalışır halde göremedik. Bu bölümde servis objesinin ne olduğunu öğrenecek ve basit bir uygulama ayağa kaldırarak bu zamana kadar yaptıklarımızı pekiştireceğiz. 

Bir örnek senaryo ile başlayalım. Front-end , Back-end ve DB olacak şekilde 3 parça halinde çalışan bir uygulamamız olsun.Front-end tarafında kullanıcı arayüzünden data girilsin  , front-end bunu back-end tarafına göndersin , back-end de datayı işleyip db ye yazsın.  Yukarıdaki resimdeki gibi ilgili objeleri kubernetes cluster'ı içinde ayağa kaldıralım ve bir kaç soruyla devam edelim .

-   Front-end pod'unu oluşturduk  fakat kullanıcı bu uygulamaya nasıl erişilecek ? 
-   Front-end uygulamasını erişebilir hale getirdikten sonra gelen data back-end pod'una nasıl gönderilecek ? 
-  Pod'lardan birinde bir problem oldu ve kill olup yeniden aynı imajı kullanan pod ayağa kalktı iletişim bundan etkilenecek mi ? 

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136466330-c22a4199-fe07-40b6-8ba0-c187f860c8a0.png"/>
</p>


Kubernetes  cluster'ı içinde oluşturulan objeler için sanal bir ip bloğundan objelere ip ataması yapar.  Bu objelere atanan ip'ye direkt olarak erişim mümkün değildir.

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
Örneğin yukarıdaki yaml dosyasından bir pod oluşturalım. Bu pod'a kubernetes otomatik olarak bir ip ataması yapar.  Atadığı ip kubernetes cluster'ı içindeki sanal bir bloktan alındığı için  bu ip yi kullanarak kendi bilgisayarımızdan erişemeyiz. Erişebilmek için kubernetes cluster'ına önce ssh ile bağlanmamız gerekmektedir. SSH ile bağlandıktan sonra bu pod'a bağlanabiliriz. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136467710-b8d18170-af13-4389-bd96-2965fc345ecc.png"/>
</p>

Biz front-end uygulamamızın dış dünyaya açılmasını istiyorduk. SSH ile bağlandık fakat bu tam olarak bizim isteğimizi yerine getirmedi. 
Kubernetes bize bu problemin çözümü için servis objesini sunmakta. Servis objesi ile biz pod'larımızı erişebilir hale getirebiliriz. Yukarıdaki resimde yeşil alanda  30008 sayısını görmekteyiz. Burada 30008 portundan dış dünyaya bir kapı açıyoruz. Servis objesi bu açılan kapıyı dinliyor ve gelen talebi pod'a ileterek bizim uygulamamızı erişebilir hale getiriyor.

curl http://192.168.1.2:30008 komutunu çalıştırdığımızda artık pod'umuza direkt olarak kendi bilgisayarımızdan erişebilir hale geliyoruz. 
<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136468624-3658d816-d2b0-478f-a120-e9cadfed3923.png"/>
</p>

Yukarıda bahsedilen senaryodaki problemi servis tiplerinin birtanesi ile çözdük. Farklı servis tipleri de vardır . Bunlar : 
- NodePort
- ClusterIp
- LoadBalancer

NodePort Service İle başlayalım  : 
<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136469091-6eb0e346-964f-4bfd-b7c4-7a5159de9b9c.png"/>
</p>

Yukarıdaki resimde NodePort Servisin çalışma prensibini görmekteyiz. 
- Nodeport (30008) : Dış dünyadan gelecek olan taleplerin karşılanacağı , uygulamanın dış dünyadan erişilebilmesini sağlayan porttur. NodePort değerleri 30000 - 32767 aralığında verilebilmektedir.
- Port(80 servis objesinin yanındaki ): Dış dünyadan gelen talepleri karşılayan servis objesinin gelen talebi POD'a hangi porttan göndereceğinin belirtildiği parametredir.
- TargetPort: Servis objesinin yönlendirdiği talebin POD objesine hangi porttan ulaşacağının belirtildiği parametredir. 

Aşağıda servis objesi ve pod objesinin yaml örnekleri bulunmaktadır. Label'ı frontend olarak ayarlanan ve 80 portu açılan bir pod ve cluster'ın 30008 portunu dış dünyaya açan , gelen talepleri kendi 80 portundan pod'un 80 portuna gönderen ve app:frontend label'ı olan pod ile eşleşen bir servis objesi görmekteyiz. 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30008
      targetPort: 80
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: podnginx1
  labels:
    app: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136471202-7f4185cf-4b44-4108-8e28-d7e988a4136d.png"/>
</p>


ClusterIp Service : 

ClusterIp Servisleri yalnızca kubernetes cluster'ı içinden erişilebilen servislerdir. NodePort servis gibi dış dünyaya açılmazlar. Bu varsayılan servis tipidir. Peki neler yapar inceleyelim.


Nodeport Service ile uygulamayı dışarı açtık ve servis talebi aldı. Yukarıdaki resimdeki gibi 3 tane frontend pod'u olduğunu varsayalım. Servis talebi hangi pod'a yönlendirecek ? Aslında burada cevap basit. Random olarak talepleri iletmekte. Servis frontend'e iletti ve kullanıcı frontend üzerinde işlemler yapmaya başladı , yaptığı işlemlerin sonucunda backend poduna gitmesi gerekti. Frontend pod'u back'end pod'una nasıl gidecek ? Burada işte ClusterIp tipindeki servis devreye girmekte. Frontend pod'undan çıkan talep backend servis'ine gider ve backend servis'i de talebi backend pod'una iletir. Böylece backend pod'u dış dünyaya açılmadan cluster içinde izole kalarak görevini tamamlayabilir hale gelir. Aynı senaryo backend pod'undan çıkıp db pod'una giderkende gerçekleşir ve işlem tamamlanmış olur. 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```
Burada gördüğümüz tüm objelerin ip adresleri vardır aslında fakat ip adresinin kullanılması tercih edilmez. Örneğin bir backend pod'u kill oldu ve yerine yenisi kalktı burada yeni kalkanın ip adresinin aynı olacağının garantisini veremeyiz bunun yerine objeler arasındaki bağlantıyı label'lar üzerinden yaparız. Böylece herhangi bir obje kill olup yeniden oluşturulduğunda otomatik olarak kaldığı yerden çalışmaya devam eder.


Üçüncü servis tipimiz ise LoadBalancer tipi servislerdir. Birden fazla aynı cluster'a hizmet veren ip adresimiz olabilir.

- 192.168.1.3 
- 192.168.1.4
- 192.168.1.5

Gibi ip adresleri aynı cluster'a bağlantı sağlıyor olabilir .  Bu ip adreslerinin önüne Loadbalancer tipinde bir servis koyduğumuzda random şekilde ip adreslerine gelen talepleri yönlendirir ve cluster'a erişilebilmesini sağlar. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontendlb
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

--> Servisleri listeleme :
```bash
kubectl get svc
```
--> Servisleri silme :
```bash
kubectl delete svc service_name
```

--> Servisin detayını görüntüleme :
```bash
kubectl describe  svc service_name
```

* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Deployment) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates) 
