
<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136333926-d07bcd11-eb86-4fe7-b572-3aee9233759a.png"/>
</p>


Kubernetes pod objelerinden oluşmaktadır. Kubernetes ilk oluştuğunda bir pod kümesi oluşturur. Bu oluşturulan pod kümesi ayrı bir namespace'de oluşturulur. Bunun sebebi kubernetes'i kullanıcıdan yalıtmak ve yanlışlıkla kubernetes sistem podlarının silinmesini / değiştirilmesini veya müdahele edilmesini engellemektir. Bu namespace'in adı " kube-system " dir. Otomatik olarak oluşturulan ikinci namespace ise "kube-public" dir . Tüm kullanıcıların kullanımına sunulması gereken kaynaklar burada oluşturulur. Üçüncüsü ise default namespace'dir. Developer kubernetes cluster'a bağlandığında default olarak bu namespace'de çalışır. Oluşturduğu tüm objeler , yaptığı tüm işlemler bu namespace üzerinde gerçekleşmektedir. 

Bir kubernetes cluster'ı kurumsal veya üretim amacıyla kullanıldıkça kendi namespace'lerinizi oluşturmak isteyebilirsiniz. Eğer ortam küçükse default namespace'inde devam edebilirsiniz .

<p align="center">
  <img src="https://user-images.githubusercontent.com/38957716/136333982-afb4333e-1496-44eb-a200-d6084dea87ac.png"/>
</p>

Örneğin aynı cluster'ı hem dev hem de prod ortam için kullanmak istiyorsanız , prod ve dev ortamlarının kaynaklarını birbirlerine karşı izole ederek kullanabilirsiniz. Bu kaynak izolasyonunu ise namespace'ler aracılığı ile yapabilirsiniz. Bu sayede dev ortamında çalışırken prod ortamındaki bir kaynağı yanlışlıkla değiştiremezsiniz. 

--> imperative olarak namespace oluşturma : 
```bash
kubectl create namespace dev
```
--> Declerative olarak namespace oluşturma


```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

--> Başka bir namespace'de bir obje oluşturma :
```bash
kubectl apply -f "yaml_file.yaml" --namespace="hedef_namespace"
```
```bash
kubectl apply -f pod_definition.yaml --namespace=dev
```
--> Başka bir namespace'de ki objeleri görüntüleme
```bash
kubectl get "object_type" --namespace="hedef_namespace"
```
```bash
kubectl get pods --namespace=dev
```
--> Namespace Değiştirme
```bash
kubectl config set-context $(kubectl config current-context) --namespace="hedef_namespace"
```
```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

* [<-- Geri](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Label%20and%20Selectors) [/ ileri -->  ](https://github.com/softwareoneturkey/swo-k8s-tepmlates/tree/main/Service) 
