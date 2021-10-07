<p align="center">
  <img src="https://user-images.githubusercontent.com/55376595/133259669-772cdfbf-4a9b-4f51-ad65-771615ee18c6.png"/>
</p>

```yaml
apiVersion: ##oluşturulacak olan komponentin hangi api versiyonunda geldiğini belirtmektedir.örneğin deployment objesi apps/v1  api versiyonunda gelmiştir.
kind: ##oluşturulacak olan komponentin tipinin belirtildiği alandır. Örneğin " Pod , Deployment , Service , ReplicaSet" gibi.
metadata: ##oluşturulacak olan komponentin temel özelliklerinin belirtildiği alandır , name , label , annotations gibi eklemeler burada yapılır.
spec: ##oluşturulacak olan komponentin özelliklerinin , bağımlılıklarının ve tüm detaylarının belirtildiği alandır.

```



**1_pod.yaml dosyasını inceleyelim :**


```yaml
apiVersion: v1 ## --> Pod objesin api versiyonu v1 dır
kind: Pod ## --> Pod objesi bu şekilde tanımlanır.
metadata:
    name: podnginx1 ## --> Pod'un kubernetes cluster'ı içindeki ismini burada belirtiyoruz.
spec:
    - name: nginx ##--> konteyner imajından oluşturulacak olan konteynerin ismini burada belirtiyoruz.
      image:nginx ## --> Pod'un kullanacağı konteyner imajını burada belirtiyoruz.

```
ports ve containerPort'u ilerleyen bölümlerde inceleyeceğiz.

Pod yaml dosyasmızı yazdık geçelim oluşturma işlemine:

```bash
kubectl apply -f 1_pod.yaml
```

 --> komutu ile oluşturmuş olduğumuz yaml dosyasını kubernetes de işleme aldırıyoruz.

```bash
kubectl get pods
```
 --> komutu ile çalışmakta olan podları listeleyebiliyoruz. Burada podnginx1 podunun çalıştığını göreceğiz.


#####  
**2_pod_imageError.yaml dosyasını inceleyelim :** 

1.yaml dosyamızla herşey aynı fakat burada container's in altındaki kullanılacak olan imaja baktığımızda adının nginex olarak yazıldığını görmekteyiz. 
kubernetes referans olarak verilen imajı bulamadığında , pod'un statüsünü  " ErrImagePull " olarak ayarlamakta ve çalıştıramamaktadır. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imageerrorpod
spec:
  containers:
  - name: imageerrorcontainer
    image: nginex

```

```bash
kubectl apply -f 2_pod_imageError.yaml
```
 --> komutu ile oluşturmuş olduğumuz yaml dosyasını kubernetes de işleme aldırıyoruz.
```bash
Kubectl get pods 
```
--> dediğimizde , imageerrorpod isimli pod'un statüsünün " ErrImagePull " olduğunu göreceğz.

###
**3_complated_pod.yaml dosyasını inceleyelim :**

Bir web sitesini düşünelim , web sitesi sürekli erişilebilir olması gerekmektedir. Hal böyle olunca web sitesi için oluşturmuş olduğumuz pod sürekli running durumda olması gerekmektedir. Peki sürekli çalışması gerekmeyen bir işi biz container olarak ayarlar ve kubernetes de pod içinde çalıştırırsak ne olur ? 

Bunun için 3_complated_pod.yaml dosyasında ubuntu imajını çalıştırdık. Ubuntu imajı bir shell olarak çalışır ve verilen komutları yerine getirerek görevini tamamlar . Biz yaml dosyamızda command alanında bir komut girdik ve 20 saniye boyunca beklemesini söyledik. 20 saniye bekledikten sonra başka komut olmayacağından dolayı pod'un işi biter ve complated statüsüne geçer. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complatedpod
spec:
  restartPolicy: Never
  containers:
  - name: complatedcontainer
    image: ubuntu
    command: ['sh', '-c', 'echo "Selam ben 20 saniye calistim ve isim bitti!" && sleep 20']

```


```bash
kubectl apply -f 3_complated_pod.yaml 
```
--> komutu ile oluşturmuş olduğumuz yaml dosyasını kubernetes de işleme aldırıyoruz.


```bash
kubectl get pods
```
--> komutunu girdiğimizde compatedpod isimli pod'un statüsünün complated olduğunu görmekteyiz. 


###
**4_error_pod.yaml dosyasını inceleyelim :**

2_pod_imageerror.yaml dosyasında imajın bulunamadığı durumdan bashetmiştik. Burada ise konteyner'in beklediği paremetrelerde hatalı giriş yaparsek eğer imaj bulunsa bile konteyner'a hatalı parametre gönderildiğinden dolayı pod'umuz çalışmaya başlayamacak. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: errorpod
spec:
  restartPolicy: Never
  containers:
  - name: errorcontainer
    image: ubuntu
    command: ['sh', '-c', 'abc']
```
```bash
kubectl apply -f 4_error_pod.yaml
```
--> komutu ile oluşturmuş olduğumuz yaml dosyasını kubernetes de işleme aldırıyoruz.

```bash
kubectl get pods 
```
--> komutunu girdiğimizde errorpod isimli pod'un statüsünün "Error" olduğunu görmekteyiz.


###

**5_CrashLoopBackUp.yaml dosyasını inceleyelim :**

Kubernetes içinde bir pod oluşturduk. Oluşturduğumuz pod ise sürekli restart oluyor olsun. Kubernetes bir kaç restart boyunca her hangi bir müdahelede de bulunmaz fakat süreklilik arz etmeye başladıysa bu restart durumu , burada anormal giden bir şeyler olduğunu farkeder ve "CrashLoopBackOff" statüsüne pod'u alır. Bu statüye alınarak çalıştırılmaya çalışılan pod'un sağlıklı olmadığını kontrol edilmesi gerektiğini ilgili birimlere gösterilmesi sağlanır. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crashlooppod
spec:
  restartPolicy: Always
  containers:
  - name: crashloopcontainer
    image: ubuntu
    command: ['sh', '-c', 'echo "Selam ben 3 saniye calistim ve isim bitti!" && sleep 3']
```

```bash
kubectl apply -f 5_CrashLoopBackOff_pod.yaml
```
--> komutu ile oluşturmuş olduğumuz yaml dosyasını kubernetes de işleme aldırıyoruz.

```bash
kubectl get pods
```
--> komutunu girdiğimizde crashlooppod pod'unun bir süre sonra " CrashLoopBackOff " statüsüne geçtiğini görmekteyiz. 


Kubernetes komutlarının bazıları:


Bir objenin özelliklerini ayrıntılı incelemek  : 
```bash
kubectl describe obje_tipi obje_name
```
Bir pod'un loglarını görüntülemek :
```bash
kubectl logs pod_name
```
Kubernetes de bir objeyi silmek :
```bash
kubectl delete obje_tipi obje_name
```

Yaml dosyasından obje oluşturma :
```bash
kubectl apply -f example.yaml
```

Yaml dosyasından oluşturulan objeyi silme : 
```bash
kubectl delete  -f example.yaml
```

Bir pod'un içine bağlanıp komut çalıştırabilmek :
```bash
kubectl exec -it pod_name --sh
```
Bir pod'un içindeki konteyner'a  bağlanıp komut çalıştırabilmek :
```bash
kubectl exec -it pod_name -c container_name --/bin/sh

```
Label ve port özelliklerini  ekleyerek  pod oluşturma.
```bash
kubectl run "pod_name" --image="image_name" --port="port_numarası" --labels"key:value" --restart=Never
```

impertive yöntemler pod olusturma 
```bash
kubectl run pod_name --image= image_name
```

Cluster'da bulunan bir Kubernetes objesini varsayılan text editörü ile açarak özelliklerini güncelleme.

```bash
kubectl edit "obje_tipi" "obje_name"
```

kubernetes pod'ları listeleme
```bash
kubectl get pods
```


Tüm kubernetes objelerini listeleme
```bash
kubectl get all
```




* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Minikube%20Install) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/ReplicaSet%20-%20ReplicationController) 
