---
title: "Kendimize özel VPN kurulumu "
date: 2020-07-01T10:00:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["security", "linux", "macosx","vpn","vps","tunneling","wireguard"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Decent open source, lightweight VPN solution."
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "../../images/wireguard.svg" # image path/url
    alt: "" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


# VPN Kuralım


Bugün sizlere kendinize ait VPN sistemi nasıl kurulur, onu anlatmak istiyorum, daha önce İngilizce olarak, [yayınladım](https://mrturkmen.com/setup-free-vpn/) fakat  Türkçe bir kaynağın da faydalı olabileceğini düşündüm. Burada anlatılanlar, ubuntu ailesine (16.04,18.04) ait sunucular üzerinde test edilmiştir. 

İlk olarak bulut hizmeti sağlayan bir şirketten bu DigitalOcean, Google Cloud, Microsoft Azure veya Amazon olabilir, sunucu kiralıyorsunuz, en ucuzu ve makul olanı DigitalOcean tarafından sunulan [aylık 5 dolar](https://www.digitalocean.com/pricing/) olan sunucu diyebilirim. Sunucuyu kiraladıktan ve ssh bağlantısını sağladıktan sonra VPN kurulumuna geçebiliriz. 

VPN hakkında tam bilgisi olmayan arkadaşlar için şu şekilde özetlenebilir, sizin için oluşturulmuş sanal bir bağlantı noktası gibi düşünebilirsiniz. Yani VPN'e bağlandıktan sonra bilgisayarınızdan çıkan ve bilgisayarınıza gelen ağ trafik şifrelenmiş olarak işlenir. Üçüncü parti yazılımların veya MITM gibi saldırıların önüne geçmiş olursunuz. 

## Neden kendi VPN sistemi kurmalıyız ? 

Çünkü şu anda var olan bütün VPN sistemleri, ücretsiz olarak hizmet sağlasa dahi, sizin bilgilerinizin satılması, arşivlenmesi ve gerektiğinde ilgili birimlere aktarılması amacıyla kaydedilmektedir. Bunun ne gibi zararları olabilir gelin birlikte şöyle bir sıralayalım: 

- Oltalama saldırılarına sadece sizin bilebileceğiniz bilgiler ile maruz kalma. 
- Ziyaret ettiğiniz siteler tarafından reklam bombardımanına maruz kalma.
- Kişisel bilgilerinizin reklam veren ajanslara satılması, bu durum birçok kişi tarafından tam olarak anlaşılamıyor, yani şu şekilde anlaşılamıyor, internet üzerinden alışveriş yapan A kişisi, kendine ait bilgilerin, onun bilgilerini satacak kişiler tarafından değersiz olduğuna inanıyor ve hiçbir gizlilik sağlamadan internet kullanımına devam ediyor. Bu sonunda o kişiye zarar vermese bile o kişinin konuştuğu, görüştüğü veya birlikte çalıştığı arkadaşlara zarar verebiliyor. 

Burada sıralananlar sadece buzdağının görünen ucu bile diyemeyiz, günümüzde veri işleme teknikleri ve yaklaşımları öyle gelişmiştir ki siz bile kendinize ait olan bir şeyin varlığına farkında olmadan onlar işlemleri tamamlamış oluyor :). 

Bu ve bunlardan çok daha fazla nedenden dolayı VPN kullanımı şart diyebilirim. Peki bunu nasıl yapacağız, bu kısımdan sonra sizin bir bulut sağlayıcısı tarafından sunucunu kiraladığınızı ve ssh bağlantısını sağladığınızı varsayıyorum. 

Bu gönderide WireGuard VPN uygulaması kullanılacaktır. WireGuard VPN uygulaması açık kaynaklı bir uygulama olup, sağladığı imkanlar sayesinde diğer VPN uygulamalarına (OpenVPN ve diğerleri) kıyasla çok daha hızlı ve güvenilirdir. 


## Sunucu Ayarları

**VPN uygulamasını kiraladığımız sunucu üzerine kuralım.**

```bash 
$ sudo apt-get update && sudo apt-get upgrade -y 
$ sudo add-apt-repository ppa:wireguard/wireguard
$ sudo apt-get update 
$ sudo apt-get install wireguard
```


**Uygulamayı çekirdek güncellemeleri ile birlikte güncellemek için gerekli komutu girelim.** 

  ```bash 
  $ sudo modprobe wireguard
  ```

**Aşağıda verilen komut girildiğinde beklenen sonuç.**

```bash 
$ lsmod | grep wireguard

wireguard             217088  0
ip6_udp_tunnel         16384  1 wireguard
udp_tunnel             16384  1 wireguard
```

**Anahtarları üretelim**

```bash 
$ cd /etc/wireguard
$ umask 077
$ wg genkey | sudo tee privatekey | wg pubkey | sudo tee publickey
```

**VPN Konfigurasyon dosyasını `/etc/wireguard/wg0.conf` ayarlayalım.**

```conf 

[Interface]
PrivateKey = <daha-öncesinde-üretilen-gizli-anahtar>
Address = 10.120.120.2/24
Address = fd86:ea04:1111::1/64
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
ListenPort = 51820
```
Burada önemli nokta `ens3`, ip tables komutu içerisinde yer alan `en3`, sunucudan sunucuya farklılık gösterebilir, bundan dolayı sizin sunucunuzda ne ise ağ kartının ismi onu girmelisiniz. `ifconfig ` komutu sayesinde öğrenilebilir. 

Bir diğer önemli nokta ise daha öncesinde 4. adımda üretilen `privatekey`, içeriğinin `PrivateKey` alanına girilmesidir. 

**Ağ trafiğini yönlendirme**

`/etc/sysctl.conf` dosyası içerisine aşağıda verilen bilgileri girerek kaydediniz. 

```conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

**Bilgiler gerekli dosyaya kaydedildikten sonra aşağıdaki komutlar sırası ile girilmelidir.**


```bash 
$ sysctl -p
$ wg-quick up wg0
```

**Komutların girilmesi ve herhangi bir sorun görülmemesi durumda ve `wg` komutu terminale girildikten sonra aşağıda verilen çıktıya benzer bir çıktı göreceksiniz.**

```bash 
$ wg
interface: wg0
  public key: loZviZQpT5Sy4gFKEbk6Vc/rcJ3bH84L7TUj4qMB918=
  private key: (hidden)
  listening port: 51820
```

Eğer herhangi bir sorun ile karşılaşmazsanız bu adıma kadar, bu demek oluyor ki, sunucu tarafında işiniz şimdilik tamamlandı geriye sadece kendi bilgisayarımızı, telefonumuzu vs VPN sunucusuna bağlamak kaldı. 

## Kullanıcı Ayarları

Kullanıcıların kendi bilgisayar ortamlarında, telefonlarında, tabletlerinde veya diğer sunucularında kullanabileceği uygulamaları [buradan](https://www.wireguard.com/install/) indirebilirsiniz. 

Gerekli uygulamayı kendi ortamınıza indirdikten sonra tek yapmanız gereken, VPN sunucusu tarafında ayarladığımız VPN'e bağlanmak, bunun için gerekli olan sadece konfigurasyonları doğru girmek olacaktır. 

Kullanıcı tarafında, uygulama üzerinden aşağıda verilen konfigurasyona benzer bir ayarı (kendi kurduğunuz VPN ayarlarına göre privatekey ve ip adressi değişiklik gösterecektir.) ayarlamanız gerekmektedir. 

```conf
[Interface]
Address = 10.120.120.2/32
Address = fd86:ea04:1111::2/128
# note that privatekey value is just a place holder 
PrivateKey = KIaLGPDJo6C1g891+swzfy4LkwQofR2q82pFR6BW9VM=
DNS = 1.1.1.1

[Peer]
PublicKey = <sunucunuza-ait-public-anahtar>
Endpoint = <sunucunuzun-dış-ip-adresi>:51820
AllowedIPs = 0.0.0.0/0, ::/0

```

Gerekli işlemler kullanıcı tarafında da sağlandıktan sonra, sunucu tarafında bu kullanıcıya bağlantı izni vermek kalıyor, onuda aşağıda verilen komut ile sağlayabilirsiniz. 

```bash 
$ wg set wg0 peer <kullanici-public-anahtari> allowed-ips 10.120.120.2/32,fd86:ea04:1111::2/128
```

Sunucu tarafindan kullanicinin VPN baglantisi sağladığını aşağıda verilen komut ile teyit edebilirsiniz. 

```bash 
$ wg

interface: wg0
  public key: loZviZQpT5Sy4gFKEbk6Vc/rcJ3bH84L7TUj4qMB918=
  private key: (hidden)
  listening port: 51820

peer: Ta9esbl7yvQJA/rMt5NqS25I/oeuTKbFHJu7oV5dbA4=
  allowed ips: 10.120.120.2/32, fd86:ea04:1111::2/128

```

Daha sonrasinda, wireguard tarafından oluşturulan ağ kartını aktivate edelim. 

```bash 
$ wg-quick up wg0
```

## Güvenlik Duvarı ayarları

Bazen sunucu tarafında yapmanız gereken bazı güvenlik duvarı ayarları bulunmakta, bunlar VPN bağlantısını başarılı bir şekilde sağlamanız için kritik öneme sahiptir. 



```bash 
$ ufw enable
```

**VPN uygulamasına bağlanmamızı sağlayacak portu açıyoruz.**

```bash 

$ ufw allow 51820/udp
```

**IP tabloları ile 51820  portu için bazı ayarlamalar yapıyoruz.**

```bash 
$ iptables -A INPUT -p udp -m udp --dport 51820 -j ACCEPT
$ iptables -A OUTPUT -p udp -m udp --sport 51820 -j ACCEPT
```

Burada önemli olan kısımlardan biriside bütün komutlar ROOT, yani yönetici yetkisi ile yapılmalı, aksi takdirde hata verecektir. 


Bu noktadan sonra, bilgisayarınıza, tabletinize veya telefonunuza kurduğunuz WireGuard uygulaması sayesinde sorunsuz ve güvenlikli bir şekilde internetinizi kullanabilirsiniz. 





