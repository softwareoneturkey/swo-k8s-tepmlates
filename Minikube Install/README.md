<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136299673-a61a32ba-cccf-44d2-9f2c-86d569cf0952.jpg"/>
</p>

#### **MINIKUBE INSTALL**




```bash
https://minikube.sigs.k8s.io/docs/start/
```
Adresine gidilir .  


Minikube kullanabilmek için 2 CPU , 2 GB RAM , 20 GB Disk alanı , internet bağlantısı ve sanallaştırma katmanını sağlayacak olan " Docker , Hyperkit , Hyper-v , KVM , Parallels , Podman , VirtualBox veya VMWare " programlarından birine ihtiyaç vardır. Minikube bu programlar aracılığı ile bir sanal ortam ve bu ortamın üzerinde kubernetes ayağa kaldırır.


<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136297450-6125d984-410d-4516-8396-313756def37b.png"/>
</p>

-  Sol taraftaki documentation alanından get started bölümüne tıklanır.
- Operating system alanında " windows " seçilir
- Release Type alanından " stable " seçilir.
- Installer Type alanında ".exe download " seçilir. 
- "1. Download the latest release " alanından minikube installation exe dosyası indirilir.
- Indirilen dosya yüklenir.
- Yükleme işlemi tamamlandıktan sonra powershell run as admin olarak açılır.

Minikube programını kullanıma hazır hale getirelim.
```bash
minikube status
```
```bash
minikube profile list
```
Komutlarıyla  minikube programını kontrol edelim.

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136298113-ad363edb-4bf9-42d1-bff0-4e6990cfc187.png"/>
</p>

Minikube yukledik fakat herhangi bir profil oluşturmadık . 
Minikube sanallaştırma programlarını kullanarak onlar üzerinde sanal bir ortam ayağa kaldırır. Kubernetes bu sanal ortamda kurulur.  Hyperv sanallaştırma programını kullanarak devam ediyoruz.

Profil oluşturabilmek için aşağıdaki komutu çalıştırıyoruz.

```bash
minikube start --driver=hyperv
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136298985-8d1645a7-f0b1-48d8-8036-caefdcaac040.png"/>
</p>

Kurulum 5 ila 10 dakika arasında tamamlanmakta.  
```bash
minikube status
```
Tamamlandıktan sonra tekrar " minikube status " komutunu çalıştırdığımızda profilin oluştuğunu görmekteyiz.

```bash
kubectl get nodes
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136299242-7f18aab6-2d4b-4cae-9261-63cff28f8fe3.png"/>
</p>

Komutnu çalıştırdığımızda , minikube'un bizim için bir node ayağa kaldırdığını görmekteyiz. Artık Cluster'ımız hazır , POD bölümüyle devam edebiliriz.

* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Pod) 
