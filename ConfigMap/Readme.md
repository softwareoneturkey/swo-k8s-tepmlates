
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
| `ReadWriteMany`  | `Okuma Yazma - Coklu Node`  | Aynı anda birden fazla pod’a bağlanarak hem yazılabilir hem okunabilir. |

