
# Persistent Volume - Persistent Volume Claim - Storage Class


Volume bölümünde pod'un yaşam süresince saklamak ve yeni container oluştuğunda kaybetmemek istediğimiz dosyaları
nasıl saklayacağımızı **emptyDir** ve **hostPath** volume'ler ile görmüştük. Bu iki çözüm de sadece pod yaşam süresi
boyunca geçerli . Bu bölümde pod'un yaşam süresinden bağımsız şekilde verileri nasıl saklayabileceğimizi 
öğreneceğiz.

![App Screenshot](https://user-images.githubusercontent.com/38957716/148639762-a8d5eece-d926-4302-865a-e49ad79c408f.png)


İlk olarak 3 node'lu bir cluster'ımız olduğunu düşünelim. Bu worker node'lar üzerinde tek pod'lu çalışacak bir 
mysql veritabanı olduğunu varsayalım. Dataları kaybetmemek adına container tanımında emptyDir tipinde bir volume 
eklemiş olalım .  Tüm hazırlıkları yaptıktan sonra deployment objesi ile mysql pod'unu oluşturalım. Mysql pod'u  **WorkerNode2** node üzerinde
oluşturuldu ve çalışmaya başladı . Bu noktada eğer container'da bir problem olursa
kubelet container'ı yeniden oluşturacak ve **emptyDir** tipinde volume eklediğimizden dolayı data kaybına uğramadan
pod çalışmaya devam edecek . 

Kurgulamış olduğumuz senaryoyu değiştirelim. Diyelim ki pod'un çalıştığı **WorkerNode2** node üzerinde bir problem çıktı ve 
mysql pod'u  kubernetes tarafından **WorkerNode3** node'una atandı. Bu durum stateless uygulamalar için problem değilken 
mysql pod'u için problemdir çünkü mysql bir veritabanıdır ve üzerinde geçici yeniden kolayca oluşturulabilecek dataları değil
uzun süre zorunlu olarak tutmamız gereken dataları üzerinde tutar. Pod'a ne olursa olsun bu datalara bir şey 
olmamalı ve erişebiliyor olmamız gerekiyor. Bu çok büyük bir problem . 

![App Screenshot](https://user-images.githubusercontent.com/38957716/148639309-0ce9f96b-71ba-461b-a34c-b2a1b425ea35.png)

Bunun tek bir çözümü var , uzun süre saklamak zorunda olduğumuz dataları cluster dışında , ayrı bir alanda saklamamız gerekiyor. 
Bu ayrı alana tüm worker node'ların erişebiliyor olması gerekiyor. Eğer bunu yapabilirsek pod ne kadar restart olursa olsun,
hangi worker node üzerine taşınırsa taşınsın yeniden oluştuğu anda eski datalara ulaşabilecek ve yenilerini yazmaya başlayabilecek.

Bu noktada ihtiyacımız olan çözümün adı Persistent Volume. Persistent volume'ler cluster dışında cluster dışında 
storage'lere mount olarak dataların kaybolmamasını sağlayan objelerdir. O halde Persistent Volume ( PV )'ler 
nasıl oluşturuluyor inceleyelim ; 


Persistent volume'leri oluşturmadan önce bir kaç ayar yapılması gerekiyor. İlk olaral volume oluşturmak istediğimiz 
storage ile cluster'ımızın iletişim kurabiliyor olması gerekiyor. Bunun için de cluster içinde bu storage'in driver'larının
kurulu olması gerekiyor. Kubernetes NFS , ISCSI , Azure Disk , Azure File , AWS EBS , Google PD , Cephfs gibi
bir çok storage ' in driver'larını bünyasinde bulunduruor. Yani Kubernetes varsayılan olarak bu tipteki storage'ler ile iletişim
kurabiliyor durumda . 

Bunu bir örnek üzerinden anlatmak gerekirse , Kubernetes Cluster'ımızın erişebildiği NFS tabanlı bir storage 
olduğunu varsayalım. Kubernetes altında NFS driver'ları bulunduğu için ek bir driver yüklememize gerek kalmadan 
bu iki ortam birbiri ile iletişim kurabilir durumda. Bu adım zaten tamamlanmış durumda. Sırada bu storage üzerinde
bizlerin erişip kullanabileceği depolama alanları oluşturmakta. 

![App Screenshot](https://user-images.githubusercontent.com/38957716/148639333-a10ceed5-2fc3-4336-be6f-ee4a66e9d0eb.png)


Bizler NFS cihazımıza bağlanıyor , örneğin **tmp** diye bir alan oluşturuyoruz. Bu işi tamamladıktan sonra sıra geldi bunun 
Kubernetes tarafındaki karşılığını oluşturmaya. İşte bu noktada oluşturacağımız objenin adı **Persistent Volume**


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
   name: mysqlpv
   labels:
     app: mysql
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /
    server: 10.255.255.10
```

apVersion: v1 --> PersistentVolume eklenme versiyonu
kind: PersistentVolume

spec:
    capacity:
        storage: --> mount edilecek stroage'deki ayrılacak alan
    accessMode:

## Access Modes

AccessModes kısmında bu volume’un aynı anda birden fazla pod’a bağlandığı zaman ne şekil bir 
davranış sergileyebileceğini seçebiliyoruz. Burada 3 seçenek mevcut. 

| Parameter         | Type                        | Description                |
| :--------         | :-------                    | :------------------------- |
| `ReadWriteOnce`   | `Okuma Yazma Tek Node`      | Bu volume aynı anda sadece tek bir node’a bağlanabilir ve bu pod hem bu volume’e yazabilir hem de okuyabilir  |
| `ReadOnlyMany`    | `Sadece Okuma`              | Bu volume aynı anda birden fazla pod’a bağlanabilir fakat pod’lar sadece bu volume’de daha önceden yazılmış dosyalar varsa onları okuyabilir. Dosya yazamazlar .  |
| `ReadWriteMany `  | `Okuma Yazma - Coklu Node`  | Aynı anda birden fazla pod’a bağlanarak hem yazılabilir hem okunabilir. |


Burada kullanacağımız seçenekler storage driver'larının yeteneklerine göre değişiyor. Örneğin Azure üzerinde 
**Azure Disk** kullanılırsa sadece ReadWriteOnce desteklenirken , Azure File tipi bir stroage ve driver kullanılırsa 
bu sefer 3 seçenek de kullanılabilmekte. 

## PersistentVolumeReclaimPolicy

Pod volume'ü kullanmayı bıraktıktan sonra bu kullanılan volume'e ne yapılacağına dair seçimimizi yapıyoruz.

| Parameter | Description                |
| :-------- | :------------------------- |
| `Retain`  | Bu volume aynı anda sadece tek bir node’a bağlanabilir ve bu pod hem bu volume’e yazabilir hem de okuyabilir  |
| `Recycle` | Bu volume aynı anda birden fazla pod’a bağlanabilir fakat pod’lar sadece bu volume’de daha önceden yazılmış dosyalar varsa onları okuyabilir. Dosya yazamazlar .  |
| `Delete ` | Aynı anda birden fazla pod’a bağlanarak hem yazılabilir hem okunabilir. |


Yukarıdaki yaml dosyasını bu bilgilerle inceleyecek olursak :

  NFS tabanlı erişim sağlayan ve 172.17.0.2 ip adresi üzerinden erişilebilen bir stroage üzerindeki tmp isimli 
  paylaşımı kullanacak. 5 gigibyte boyutunda sadece tek bir pod'a bağlanabilecek ve bu pod tarafından hem dosya yazmak hemde
  dosya okumak için kullanılabilecek . Pod'un işi bittiği zaman içerisindeki dosyaların silineceği app:mysql label'ına sahip
  bir **PersistentVolume** oluşturuyoruz.
![App Screenshot](https://user-images.githubusercontent.com/38957716/148639342-34bc9098-c719-4f0a-85d4-e9ce59f8ce7a.png)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqlclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: ""
  selector:
    matchLabels:
      app: mysql
```



  Bir volume'ü direkt olarak pod ile eşleştirme imkanına sahip değiliz , Bizler bir persistent volume'ü pod'a bağlamak için öncelikle 
  **PersistentVolumeClaim** tipinde bir obje daha oluşturmamız gerekiyor.

  **PersistentVolumeClaim** kısaca **PVC** , bizlerin sistemde bulunan **PV** yani **PersistentVolume** ' lerden uygun olanı işimiz için seçmemize imkan veren objelerdir.

  Yukarıdaki resimde bulunan yaml dosyasını inceleyelim : 

  apiVersion: v1 --> v1 versiyonunda eklenmiş kubernetes ' e.
  kind: PersistentVolumeClaim --> obje tip isimli

  metedata : --> mysqlclaim adında oluşturuyor.

  Spec kısmı persistent volume’e çok benziyor burada da nekadarlık bir volume talep ettiğimizi , 
  hangi mode ile bağlanmak istediğimizi belirtiyoruz.Son olarak pvc ile pv yi label kısmında eşleştiriyoruz. PVC’nin talebinin  
  app:mysql olan pv tarafından sağlanmasını istiyoruz. 

  Peki Kubernetes neden bu depolama oluşturma ve bağlanma işini iki farklı obje üzerinden hallediyor ? 

  Bunu şöyle düşünelim , bir developer olarak kullandığımız Kubernetes cluster’ını biz yönetmiyor olabiliriz , o bizim işimiz olmayabilir.
  Bunun için çalışan ayrı bir ekip olabilir. Dolayısı ile oradaki işlemleri bizim bilmemiz mümkün olmayabilir. A firmasında NFS tabanlı bir 
  depolama ünitesi kullanılıyorken B firmasında Azure File tiğinde bir depolama ünitesi kullanılıyor olabilir. Bu ikisi için PV ayarları değişik olabilir. 
  Yukarıda bahsetmiştik , Spec altındaki ayarlar depolama ünitelerine göre değişkenlik gösterebilir. Dolayısı ile bu volume oluşturma işini developer’a verirsek ,
  developerin her cihaz için ayrı ayrı ayar yapmayı bilmesi gerekir ki bu oldukça zorlayıcı olacaktır. Kubernetes bu problem volume oluşturma ve oluşturulan volume’ü talep etme

![App Screenshot](https://user-images.githubusercontent.com/38957716/148639350-b6fa2079-9f9e-456e-b0ce-b6e408941203.png)

Elimizde pv ve buna bağlanabilecek pvc objelerini oluşturduk . Pod üzerinde kullanmak istediğimiz volume’ü talep eden persistent volume claim ile bir bağ kurarız.  
Pod tanılarında volumes anahtarı altında yeni bir volume oluşturur ve bunu hedefini oluşturduğumuz persistent volume claim olarak belirler sonrasında bunu pod altında gerkeli path’e mount ederiz. 
Bu pod oluşturulduğu anda bu talep devreye girer. Bu talebi karşılayan persistent volume objesindeki tanıma göre bu pod’un oluşturulacağı worker node üzerinde ayarlamalar yapılır. Storage cihazına bağlanılır.  
Arkada driver yapması gerekenleri yapar ve sonucunda bu pod içerisindeki konteyner’in ilgili path’ine depolama cihazında bulunan bu volume bağlanır. Bu noktadan itibaren bu path’e yazılan dosyalar aslında bu volume’e yazılır. 
Eğer bu pod bulunduğu worker node üzerinden bir şekilde silinir ve başka bir node’a taşınırsa bu sefer aynı işlemler o worker node üzerinde de gerçekleşir ve yeni pod’a bu volume bağlanır. 
Bu sayede pod’un yaşam süresinden daha uzun süre veri saklama işlemini sağlayabileceğimiz bir altyapıya sahip olmuş oluruz. 


## PV - PVC Uygulama 

Uygulama Minikube tool'u kullanılarak yapılmaktadır. Docker Uygulamasının yüklü olması gerekmektedir. NFS server container'ı çalıştırıp 
oradan storage mount etme işlemini yapacağız.

![App Screenshot](https://user-images.githubusercontent.com/38957716/148938098-a8c4a250-236a-4b81-b643-c1a3aeb97559.png)

Docker komutlarına geçmeden önce docker desktop uygulamasını açıyoruz ve volumes altına bakıyoruz . Göründüğü üzere herhangi bir volume tanımlanmamış durumda.


```bash

  docker volume create nfsvol

```
![App Screenshot](https://user-images.githubusercontent.com/38957716/148938198-826faf10-ce18-4d0b-b353-2c114f0fcb5b.png)

Docker create vol komutunu çalıştırdık ve volumes altında **nfsvol** adında volume ayağa kalktığını görünüyoruz , fakat henüz hazır değil.


```bash

  docker network create --driver=bridge --subnet=10.255.255.0/24 --ip-range=10.255.255.0/24 --gateway=10.255.255.10 nfsnet

```
![App Screenshot](https://user-images.githubusercontent.com/38957716/148938425-8beb0ea5-ecbd-41fd-b828-e0d195fe233e.png)

İkinci komutumuzla local ağımızda bir docker network oluşturuyoruz. Bu network sayesinde nfs server olarak çalışacak container'a bağlanabilecek ve data tutabileceğiz.


```bash

  docker run -dit --privileged --restart unless-stopped -e SHARED_DIRECTORY=/data -v nfsvol:/data --network nfsnet -p 2049:2049 --name nfssrv enespekdas/nfs:v1

```
![App Screenshot](https://user-images.githubusercontent.com/38957716/148938530-bf825222-16c8-4b17-a9a8-0739aecf3454.png)


Volume oluşturduk  , Container'a bağlanabilmek için network oluşturduk . Şimdi de artık nfs olarak çalışacak container'ımızı ayağa kaldırdık ve artık nfs 
container'ına bağlanıp data saklayabileceğiz. Tamamen bir simülasyon yaptığımızdan ve nfs server'ı ayarlamanın zor olacağından dolayı container olarak ayağa kaldırdık.


Hazırlıklarımız tamamlandığına göre artık kubernetes tarafında işlem yapmaya başlayabiliriz. 

## PersistentVolume - pv.yaml 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
   name: mysqlpv
   labels:
     app: mysql
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /
    server: 10.255.255.10
```

```bash

  kubectl apply -f pv.yaml

```

**mysqlpv** adında bir persistent volume oluşturuyoruz .

```bash

  kubectl get pv 

```

Oluşturulan pv'leri listeliyoruz.


## PersistentVolumeClaim - pvc.yaml
PV oluşturma işlemini tamamladıktan sonra , Bu PV'ye mount olacak PVC objesini oluşturacağız , oradan da deployment
objesine PVC ' yi mount ederek işlemi tamamlayacağız. 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqlclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: ""
  selector:
    matchLabels:
      app: mysql
```

```bash

  kubectl apply -f pvc.yaml

```

**mysqlclaim** adında bir persistent volume claim oluşturuyoruz.

```bash

  kubectl get pv 

```
![App Screenshot](https://user-images.githubusercontent.com/38957716/148940314-6a509cae-50a7-4554-b3fa-b5c45f66cd85.png)

