---
title: "debian: cp/reboot komuları"
date: 2019-01-09T09:04:03+00:00
draft: false
author: "mrturkmen"
description: "Linuxa başlamak ve basit komutları ögrenmek"
tags: ["linux", "terminal", "komut", "nftp", "wget", "scp", "html"]
toc: true
images: ["/images/bash_preview.png"]
---

## Özet:

Bu kısa yazımızda linux bilgisayarlarının terminali üzerinden yapabileceğiniz basit işlemlere dair bilgiler verilecektir. 


#### Linux tabanlı sunucularda/bilgisayarda terminal üzerinden kopyalama

Kopyalama işlemi "cp" komutu ile yapılmaktadır, bu komuta ait format aşağıdaki gibi özetlenebilir.

```bash

 cp [parametreler] [kopyalanacak-dosya] [kopyalanmasi-hedeflenen-yer] 

```

Bu komutun kullanımına örnek verelim,  kopyalanacak dizin ve kopyalanması gereken dosya ;

__Kopyalanacak dizin__ : `/home/geek/Masaustu/`

__Kopyalanacak dosya__ : `/home/geek/Dokumanlar/resim.png`

Bu durumda komut :  (* Dizinlere ve dosyalara erişim hakkına sahip olduğunuzdan emin olunuz)

```bash

cp /home/geek/Dokumanlar/resim.png /home/geek/Masaustu/

```

 Eğer bir dizin içerisindeki bütün dosyalar kopyalanması planlanıyor ise -R parametresi kullanılması gerekmektedir. Diyelim ki bir dizin içerisinde 1.png, 2.png, 3.png ... 12.png gibi dosyalar var ise ve bu dizinin adı "resimler" ise, resimler dizisi istenilen diziye aşağıdaki komut yardımı ile kopyalanabilir.

```bash

 cp -R /home/geek/Dokumanlar/resimler /home/geek/Masaustu/

```

Bu kısımda bütün veri hedeflenen dizine aktarılmaktadır.

cp komutuna ait bazı parametreler ve onların kısa açıklamaları:

`-R :` verilen dizindeki bütün dosyaları hedeflenen dizine kopyalamak için gereklidir.

`-p :` kopyalama yaparken dosyaya ait olan, oluşturulma, değiştirme, sahiplik bilgilerini değiştirmeden
      onlar ile birlikte kopyalamak için gereklidir.


#### Terminal üzerinden yeniden başlatma 

Linux terminali üzerinden bir sunucuyu yeniden başlatmak veya kapatmak için gerekli olan komutlar şu şekilde özetlenebilir. Bu komutların çalışması için sunucu üzerinde yönetici (root) yetkilerine sahip olmalısınız.

Yeniden başlatmak için

```bash

reboot

```

 Bu kısımda `"Permission denied"` veya buna benzer bir izin reddedildi mesajı aldığınızda aynı komutu sudo eki koyarak denemelisiniz, yönetici yetkisi olmadığı durumda bu tür mesajları alırsınız.

```bash

sudo reboot

```

Bu komut bazen her linux dağılımı için geçerli olmayabilir bu durumda  aşağıda verilen komut reboot komutuna alternatif olarak verilebilir.  (* Bu komut MacOS bilgisayarlar içinde kullanılabilir, MacOS işletim sistemleri Hybrid bir yapıya sahip olduğundan dolayı Unix çekirdeği içermektedir.)

```bash
 shutdown -r now 
```

`"shutdown"` komutunda isterseniz bekleme süresi ekleyebilirsiniz böylece sunucu veya bilgisayar, belirlenen bekleme süresi sonunda verdiğiniz komutu uygulayacaktır.

```bash
 shutdown -r +30
```

__30 dk sonrasında sunucu yeniden başlatılacaktır__, benzer şekilde dk belirlemektense, istediğiniz saatte bu işlemi yapmak isterseniz, istediğiniz saati aşağıdaki formatta ayarlanabilir.

```bash
 shutdown -r 19:30
```

 Sunucu, `saat 19:30` da yeniden başlatılmaya ayarlanmıştır.

`"shutdown"` komutuna ait bazı parametreler ve açıklamaları şu şekilde özetlenebilir.

-r : sunucuya yeniden başlatma sinyalini vermesi için gereklidir.

-k : sunucuya bağlı olan fakat yönetici yetkisinde erişmemiş kişileri sunucudan atar.

