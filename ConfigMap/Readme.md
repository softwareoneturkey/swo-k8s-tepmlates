

## Container File Systems

![App Screenshot](https://user-images.githubusercontent.com/38957716/148538758-e624ae0a-0a43-458b-94d3-0143d0a86809.png)


Container teknolojisinin temeline inmek gerekirse çalışması istenen uygulama tüm dependency'leriyle birlikte 
bir paket haline getirilir. Bu pakete image denir ve bu image ' ler image repository'lerde tutulur.
Uygulama runtime da çalıştırılmak istendiği zaman bu uygulama image repository den indirilir ve 
image den bir container üretilir. Bu container içerisinde de bizim uygulamamızın kodları ve dependency
leri vardır. Uygulamanın çalışması için tüm ihtiyaçlar contaienr içerisinde bulunduğundan dolayı 
containerin ayağa kaldırılmasıyla birlikte yazılmış olan uygulama çalışmaya başlar. 


![App Screenshot](https://user-images.githubusercontent.com/38957716/148538821-32aa6d80-20fa-4dc1-af12-e4966e34d446.png)


Container'lar stateless uygulamalar için uygundur. Container'lar , Containerfile ( Dockerfile) da ne yazıyorsa 
o adımları işleyerek ayağa kalkarlar , çalıştığı süre zarfında üzerinde data birikebilir fakat container'ın işi bittiğinde veya herhangi bir 
sebepten ötürülü öldüğünde üzerinde biriken data da container'la birlikte gidecektir. Container oluşturulduğunda 
Container'a ait veriler ayrı bir layer'da tutulur. Container'in sistemden silinmesiyle birlikte
bu layer'da tutulan datalarda silinir. Aynı imajdan yeni bir container oluştuğunda yeni container ,  eski container'ın oluşturmuş oluşturduğu
verileri göremez. 

Peki uygulama stateless değilde stateful ise , container sistemden kaldırıldığında çalıştığı zamanda oluşturduğu
verilerin silinmesi istenmiyorsa kalması isteniyorsa ne olacak ? Buna bir çözüm var mı ? 

Bu gibi durumlar için Kubernetes'de farklı farklı çözümler bulunmakta. Kademe kademe bu çözümleri inceliyor olacağız. 

![App Screenshot](https://user-images.githubusercontent.com/38957716/148538957-7f4558ab-fae3-4dac-b1e8-21a84086ab63.png)


Yukarıdaki resimde kubernetes içinde çalışan bir uygulama görmekteyiz. Kullanıcı 172.16.24.68 ip adresinden loadbalancer'a ulaşıyor ve loadbalancer'da gelen talebi 
frontend poduna yönlendiriyor.  Frontend pod'u yaptığı işlemleri cash de tutuyor olsun. Pod yaml dosyasında da 
livenessprob eklenmiş olsun ve bir sebepten ötürü livenessprob fail etsin. Livenessprob fail ettiğinde 
frontend pod'unun restart edilir. Aslında kubernetes'de pod'un restart olması diye bir durum yoktur.
Pod un içerisindeki container öldürülür ve aynı imajdan yeni bir container oluşturulur. 


![App Screenshot](https://user-images.githubusercontent.com/38957716/148538972-ee420b0a-254b-4e04-9541-4b152c87df98.png)


Saf container de olduğu gibi Kubernetes'de de container yeniden oluştuğunda eski containerin oluşturmuş olduğu data silinir
ve erişilemez. O halde bu problemi nasıl çözeceğiz inceleyelim : 


![App Screenshot](https://user-images.githubusercontent.com/38957716/148538985-6913860f-514b-4eb0-b73c-5172e0180164.png)


Kubernetes yukarıda bahsedilen problemlere çözüm olarak ephemeral volume özelliğine sahiptir. 

Ephemeral ( Geçici ) volume'ler container'ın yaşam süresinden bağımsız olarak çalışmaktadır. Ephemeral volume'ler 
sayesinde Pod yaşam süresince oluşturulan verileri tutabiliriz. Pod içindeki container ne kadar yeniden oluşturulursa oluşsun
farketemeksizin datalar kaybolmaz. Peki nasıl çalışır ? 

Pod yaml dosyasını oluştururken ephemeral volume tanımlamasını yaptığımızda Pod'un çalıştığı worker node üzerinde 
bir alan ayrılır ve bu alan Pod hayatta kaldığı sürece silinmez. Yukarıdaki resimde volume diye gördüğümüz
obje worker node üzerinde ayrılmış bir alandır. 

İki tip Ephemeral volume vardır : 

-- emptydir volume

-- hostpath volume

EmptyDir Volume : 

Pod yaml dosyası içerisinde gerekli ayarlamalar yapıldığında kubernetes u pod'un oluşturulduğu node 
üzerinde boş bir klasör yaratır. Daha sonra bu klasör container içindeki herhangi bir path'e mount edilebilir.
Container'ın mount edilen klasör üzerinde oluşturduğu her dosya fiziksel olarak node üzerinde oluşturulan
bu klasöre yazılır. Dolayısıyla container çalışıp data oluşturup ölse yerine yeni bir container oluşturulsa bile 
yeni oluşturulan container eski container'ın oluşturmuş olduğu dataya ulaşabilir. Bu dosyalar Pod'un 
yaşam süresi boyunca ayaktadır. Pod silinirse otomatik olarak oluşturulan bu Ephemeral volume'de silinir.


HostPath Volume : 

Temel olarak EmptyDir volume ile mantık olarak aynıdır fakat hostpath volume'de rastgele bir 
klasör oluşturmak yerine Pod'un oluşturulacağı Worker Node üzerinde spesifik bir klasör ya da dosyayı 
belirtebiliriz. Hostpath volume niş use-case'ler de kullanılabilir. Eğer statik bir dosyaya container'ın
erişmesi gerekiyorsa veya statik bir dosyadan veri okuması gerekiyorsa bu işlem yapılabilir.

3 tip HostPath Volume vardır : 

    1) Directory : Belirtilecek olan folder worker node üzerinde zaten var , varolan bu folder'ı pod'a mount edilir.
    
    2) DirectoryOrCreate : Eğer folder var ise kullan , yok ise folder'ı oluştur ve kullan anlamına gelmektedir.
    
    3) FieOrCreate : Eğer file var ise kullan , yok ise file'ı oluştur ve kullan anlamına gelmektedir. 

Ephemeral volume'lere , aynı pod içersiindeki tüm container'ların aynı anda bağlanabilirler. Aynı pod içerisindeki
container'ların ortak erişebilieceği bir alan oluşturulmuş olur . 



Genel olarak Ephemeral Volume'ler teorik olarak budur. Örneklerle teorik olarak öğrenilenleri pekiştirelim o halde.


Aşağıdaki adımları sırayla uygulayınız : 


İlk olarak emptydir volume örneği yaparak başlıyoruz : 


```bash
apiVersion: v1
kind: Pod
metadata:
  name: emptydir
spec:
  containers:
  - name: frontend
    image: enespekdas/volume_image:v1
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthcheck
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: cache-vol
      mountPath: /cache
  - name: sidecar
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "sleep 3600"]
    volumeMounts:
    - name: cache-vol
      mountPath: /tmp/log
  volumes:
  - name: cache-vol
    emptyDir: {} 
```


Pod dosyasını incelediğimizde iki adet pod olduğunu görmekteyiz. Bu podlardan ilki frontend bir uygulamayken diğeri busybox uygulaması.

Frontend uygulamasına baktığımızda standart pod oluştururken verdiğimiz adımlar olan name , image ve port alanları bulunmakta.

sonrasında ise livenessProbe eklenmiş durumda . Livenesprobb ile ilgili bilgi eksikliğiniz var ise bu repo içerisinde bulunan Livenesprob alanından öğrenebilirsiniz.

daha sonra ise volumeMounts: diye bir alan görmekteyiz. Burada pod içerisinde oluşturacağımız container'a 
volume mount ediyoruz. 
- name: cache-vol --> mount ettiğimiz  volume adını göstermekte.
  mountPath: /cache --> Mount edilen volume'e container'ın hangi folder'ındaki datayı göndereceğimizi belirtiyoruz.

/cache derken , containerın cache klasöründe oluşturulacak olan datanın cache-vol volume'une atanacağını , volume ile sync olacağını söylüyoruz.


Sidecar container'ı için de aynı şekilde volume mount işlemi yapılıyor fakat sidecar container'da cache klasörüne değil de 
" /tmp/log " klasörüne mount işlemi yapılmakta. 

Daha sonra volumes: başlığını görmekteyiz. 

- name: cache-vol --> oluşturulacak olan volume'ün ismini veriyoruz. Container'larda volumeMounts alanında -name ile bir parametre giriyorduk
bu girdiğimiz parametre oluşturulmuş olan volume'ün adıdır.
  emptyDir:{} ---> alanı ise , oluşturulan volume'ün emptyDir tipinde olacağını belirtiyor. Bu volume worker node üzerinde random bir yerde bir klasör oluşturuyor ve
  container'lar bu random oluşturulan folder'a datalarını gönderiyor. 



Yaml incelememizi tamamladığımıza göre adım adım yapmaya devam edelim . 

```bash
  kubectl apply -f emptydirVolumePod.yaml

```
```bash
  kubectl get pods 

```


![App Screenshot](https://user-images.githubusercontent.com/38957716/148549921-0d7d73a2-3d8b-4a61-8bd2-03a09c87b548.png)

Pod'umuz ayağa kalkmış ve oluşturmuş olduğumuz gibi iki container içinde çalışmakta.

kubectl describe pod emptydir


Describe komutu ile baktığımızda container alanında her iki pod'unda özelliklerini ayrı ayrı görmekteyiz.

Frontend container'ının  " Mounts: " başlığının altına baktığımızda "/cache from cache-vol" yazdığını görmekteyiz.
Frontend container'ının cache klasörünü cache-vol volume'une mount edildiğini görüyoruz.

Sidecar container'ının " Mounts: " başlığının altına baktığımızda ise " /tmp/log from cache-vol" yazdığını görmekteyiz. 
sidecar  container'ının /tmp/log klasörünü cache-vol volume'une mount edildiğini görüyoruz.


Peki bunların yapıldığını görüyoruz gerçekten öyle mi peki kontrol edelim. Önce Frontend container'ınınn
içine bağlanalım ve bir kaç işlem yapalım : 

![App Screenshot](https://user-images.githubusercontent.com/38957716/148551098-30645c5c-faf8-480e-aadf-ca226e23999d.png)

```bash
  kubectl exec -it emptydir -c frontend -- bash 

```


Komutu ile emptydir pod'unun frontend container'ına bağlanıyoruz. 

daha sonda ls yaparak neler var bakıyoruz,  burası container'dan gelen dosyalar.

```bash
  cd /

```


Komutu ile root dizine gidiyoruz 

```bash
  ls

```

komutu ile root dizinindeki dosyaları görüntülüyoruz. Burada mount ettiğimiz klasör olan " cache " klasörünü görmekteyiz.

```bash
  cd cache/

```


Komutu ile cache dizinine giriyoruz. 


```bash
  ls
```


komutu ile cache dizinini kontrol ediyoruz ve daha önce data koymadığımız için dizin şuanda boş durumda

```bash
  touch data1.txt

```
```bash
  touch data2.txt

```


cache dosyasına iki adet dosya oluşturuyoruz , bu dosyaları oluşturmamızın sebebi bakalım gerçekten container silindiğinde yerine yeni container geldiğinde
dosyalar duruyor mu sorusunun cevabını alabilmek.

```bash
  cd ..

```


komutu ile bir üst dizine çıkıyoruz


```bash
  ls
```


komutu ile buradaki dosyaları listeliyoruz

```bash
  mkdir testFolder

```

Komutu ile bir başka Folder oluşturuyoruz , bu dosyayı oluşturmamızın sebebi de gerçekten yeni container'da cache haricindeki 
dosyalar siliniyor mu kontrol etmek.

```bash
  ls
```


komutu ile folder'ın oluştuğunu doğruluyoruz.

```bash
  cd testFolder

```


komutu ile testFolder dizinine giriyoruz 

 
```bash
  ls
```


komutu ile dizinin boş olup olmadığını kontrol ediyoruz.


```bash
  touch data3.txt 

```

Komutu ile oluşturmuş olduğumuz dizine yeni bir dosya ekliyoruz.


```bash
 ls 
```


komutu ile kontrol ediyoruz ve dosyanın oluştuğunu doğruluyoruz.

 
```bash
  exit
```


Frontend pod'u içerisinde yapacaklarımızı tamamladık , bu yüzden exit komutu ile Frontend pod'undan çıkış yapıyoruz.

exit komutu ile 

![App Screenshot](https://user-images.githubusercontent.com/38957716/148573848-aec27fc4-2e5e-41de-82d5-59b3de6398d3.png)


Frontend container'ında volume mount edilen cache dosya içerisinde iki adet dosya oluşturduk ayrıca testFolder diye başka bir folder oluşturduk bakalım sidecar container da volume mount edilen 
folder'a girdiğimizde neler olacak :

```bash
  kubectl exec -it emptydir -c sidecar -- /bin/sh

```

komutu ile sidecar container'a bağlanıyoruz.

 
```bash
  ls
```

komutu ile dosyaları listeliyoruz ve volume mount edilen " tmp " folder'ını burada görüyoruz.

```bash
  cd tmp/

```

Komutu ile tmp folder'ına giriyoruz

```bash
 ls 
```


Komutu ile tmp altındaki dosyaları listeliyoruz ve volume mount edilen log folder'ını görüyoruz.

```bash
  cd log/
```

Komutu ile log folder'ına giriyoruz.


```bash
 ls 
```


komutu ile log folder'ındaki dosyaları listelediğimizde görüyoruz ki bizim frontend uygulamasında 
oluşturmuş olduğumuz data1.txt ve data2.txt dosyaları bu container'da da görünüyor.

 

```bash
 exit 
```

komutu ile sidecar container ' dan çıkıyoruz. 

Görmek istediğimiz özelliklerden birini gördük. Aynı volume mount olmuş container'lar kendi içlerinde belirtilen folder'lar ne olursa olsun
sonuç olarak worker node üzerinde bir kaynağa mount olduğundan dolayı her ikisi de aynı yere işlem yapacaktır.


Şimdi gerçekten volume mount container silindiğinde duruyor mu onu kontrol edelim o halde :

frontend container'da livenessprob bulunmakta ve healthcheck folder'ını kontrol etmekteydi , biz 
healthcheck folder'ını silersek eğer livenessprob fail edecek ve pod restart olacaktır. Yani içindeki frontend container silinip tekrar yeniden sıfırdan oluşturulacaktır.

Bu senaryoda neler çıkıyor kaşımıza inceleyelim 



```bash
  kubectl exec emptydir -c frontend -- rm -rf healthcheck

```


komutu ile frontend container'ı içindeki healthcheck folder'ını siliyoruz. Böylece livenessprob fail olacak ve pod restart olacak.


```bash
  kubectl get posd -w 

```


komutu ile emptydir pod'unun restart kolonunu kontrol ediyoruz. Bir kaç saniye sonra pod'un restart olduğunu göreceğiz. 

Tamam pod'umuz restart oldu , yani frontend container yeniden oluşturuldu. Sidecar container ise olduğu gibi kaldı.



```bash
 kubectl exec -it emptydir -c frontend -- bash
 
```

komutu ile frontend pod'una bağlanıyoruz ve daha önceden frontend container'ındaki yaptığımız işlemleri kontrol ediyoruz.

 
```bash
 ls 
```

komutu ile dosyaları listeliyoruz ve silmiş olduğumuz healthcheck folder'ının tekrar oluşturulduğunu görüyoruz

```bash
  cd /

```

komutu ile root dizine gidiyoruz

 
```bash
  ls
```


komutu ile dosyaları listeliyoruz ve daha önce oluşturmuş olduğumuz testFolder folder'ının gelmediğini görüyoruz.

```bash
cd cache/

```
komutu ile cache folder'ına giriyoruz.

```bash
ls
``` 

komutu ile listeliyoruz ve daha önce oluşturmuş olduğumuz data1.txt ve data2.txt dosyalarının hala olduğunu görüyoruz.

testFolder folder'ı default olarak container'ın içinde olmadığından ve volume mount edilmediğinden dolayı yeni oluşturmuş olduğumuz container
ile birlikte gelmedi. 



