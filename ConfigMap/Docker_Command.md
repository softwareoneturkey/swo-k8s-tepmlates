


Uygulamalar bazı noktalarda environment variable'lardan parametre alarak çalışmaktadır. Bunlar user bilgisi db bilgisi gibi bilgiler olabilir . Container dünyasında uygulamanın paketlenip imaj haline getirildiğini ve bu imajdan container runtime engine üzerinde çalıştırılacak bir container üretildiğini biliyoruz. Peki bizim uygulamamız parametre alıyor ve bu parametrelere göre ayağa kalkıyor olsun. Biz uygulamamıza bu noktada nasıl parametre göndereceğiz ? 


Docker file da gerekli ayarlamaları yaptıktan sonra imajdan docker run komutu ile konteyner ayağa kaldırırken bu parametreleri vereibiyoruz. Gelin aşağıda bu işlemin örneğine bakalım . 

docker exec -it condainer_name /bin/bash

enespekdas/ubuntu-sleeper --> imajı kullanılacaktır. 

ubuntu sleeper imajı default olarak 120 saniye sleep komutunu çalıştırmakta , peki biz 120 saniye değil de 30 saniye çalıştırmak istersek bunu nasıl yapacağız ?


docker run -it -d enespekdas/ubuntu-sleeper 30  , diyerek ubuntu imajına 120 saniye olarak belirlenen default parametresini 30 parametresini göndererek ezdirip 30 saniye çalışmasını sağlayabiliriz. 


Veya başka bir uygulamamız olsun , bu uygulama web sitesinin arka planın rengini parametre olarak alabilisin . Biz bu rengi parametre olarak nasıl göndeririz  ? 


enespekdas/simple-webapp-color --> imajı kullanılacaktır. 


docker run -it -d -e APP_COLOR=BLUE enespekdas/simple-webapp-color 

komutunu çalıştırarak environment variable olarak blue rengini parametre olarak verebiliriz. 


