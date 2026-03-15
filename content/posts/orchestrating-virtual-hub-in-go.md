---
title: "Go ile Sanal Ortam Orkestrasyonu"
date: 2026-02-26T15:00:00+01:00
tags: ["go", "infrastructure", "docker", "libvirt", "wireguard", "conference", "goroutine", "networking"]
author: "mrturkmen"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Tek bir Go binary'si ile container, VM, ağ ve VPN'i ayağa kaldırmak — neden var, nerede işe yarar, neleri eksik."
disableHLJS: true
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

Geçtiğimiz günlerde **Gophers İstanbul 2026**'da bir sunum yaptım. Konu: tek bir Go binary'si ile Docker container'ları, libvirt/KVM sanal makineleri, DHCP+DNS altyapısını ve WireGuard VPN tünelini — hepsini bir YAML dosyasından ayağa kaldıran bir araç. Sunumda bol bol kod parçacıkları ve Go'nun iç yapısı var. Bu yazı *o değil*. Burada daha çok aracın arkasındaki dağınık gerçek-dünya probleminden, nerelerde gerçekten işe yaradığını düşündüğümden, nelerin hâlâ eksik olduğundan ve vakit bulursam denemek istediğim fikirlerden bahsetmek istiyorum.

**[📊 Sunum slaytları](/talks/orchestrator.html)** · **[📊 Slides (English)](/talks/orchestrator_en.html)** · **[💻 Kaynak kod](https://github.com/mrtrkmn/orchestrator)**

---

## Kaşıntı

Bir noktada çoğu backend ya da platform ekibi benzer bir duvara tosluyor: aynı makine üzerinde hem container hem de VM çalıştırmanız gereken bir demo ya da test ortamı ihtiyacı çıkıyor. İlk başta kulağa çok büyük bir mesele gibi gelmiyor. “Bir iki script yazarız, ayağa kalkar” diye düşünüyorsunuz. Sonra işin detayları yavaş yavaş ortaya çıkıyor.

Önce Docker ağı için küçük bir script yazılıyor. Ardından DHCP tarafı için bir konfigürasyon üreten başka bir script ekleniyor. Sonra virsh için XML tanımları geliyor. Her şeyin hazır olduğunu düşündüğünüz noktada bu kez VM’in IP almadığını fark ediyorsunuz. Biraz kurcalayınca sorunun bridge arayüzünde olduğunu görüyorsunuz; promiscuous mode açık değil. Bu defa zincire bir ip link set daha ekleniyor.

Asıl sorun ise genelde daha sonra ortaya çıkıyor. Birisi sizden aynı ortamı başka bir makinede tekrar kurmanızı istediğinde, yazdığınız şeyin önemli bir kısmının fark etmeden kendi geliştirme makinenize bağlandığını anlıyorsunuz. Interface isimleri, dosya yolları, varsayımlar, geçici çözümler… Her şey ilk kurulduğu anda mantıklı görünse de tekrar üretilebilirlik tarafında tablo pek iç açıcı olmuyor.

Ben yıllar önce iş yerinde tam olarak bu problemle karşılaştım. İhtiyacımız olan şey izole hibrit ortamlardı. Bazı servisler container içinde sorunsuz çalışıyordu, ama bazı iş yükleri gerçekten sanal makine gerektiriyordu. Özellikle kendi kernel’ine ihtiyaç duyan ya da container içinde simüle edilmesi anlamsız olan bileşenlerde VM kaçınılmazdı.

Sorun şu ki manuel kurulum hem yavaştı hem de güvenilmezdi. Ortamı baştan kurmak genelde 15-20 dakika sürüyordu ve neredeyse hiçbir zaman tamamen aynı sonucu vermiyordu. Bazen bir container geride kalıyor, bazen bridge ağı yeniden başlatmalardan sonra yaşamaya devam ediyor, bazen de eski VM tanımları yeni çalıştırmalarla çakışıyordu. Ortamı ayağa kaldırmak can sıkıcıydı ama kaldırmak çoğu zaman daha da kötüydü.

Bir süre sonra bunun script koleksiyonuyla çözülecek bir problem olmadığını fark ettim. Parça parça otomasyon eklemek yerine, ortamın tamamını yöneten tek bir araç olması çok daha mantıklıydı. Bunun üzerine her şeyi tek yerden yöneten küçük bir orchestrator yazdım.

Kullanımı mümkün olduğunca basit tuttum:

```bash 
orchestrator up -c config.yaml    # ortamı ayağa kaldır
orchestrator status               # ne çalışıyor kontrol et
orchestrator ssh demo-vm          # VM'e SSH ile bağlan
orchestrator down                 # ortamı temiz şekilde kaldır

```
Amaç yalnızca “çalıştırmak” değildi. Aynı zamanda tekrar üretilebilir, taşınabilir ve temiz kaldırılabilir bir yapı kurmaktı. Bir ortam ayağa kalktığında hangi ağların oluşturulduğunu, hangi VM’lerin tanımlandığını, hangi container’ların bağlı olduğunu sistematik biçimde bilmek; iş bittiğinde de bunların hepsini geride iz bırakmadan temizleyebilmek gerekiyordu.

Araçla ilgili sevdiğim detaylardan biri de dağıtım tarafının çok sade olması oldu. Uygulama yaklaşık 15 MB’lık statik bir binary olarak derleniyor. Herhangi bir çalışma zamanı bağımlılığı gerektirmiyor. Dosyayı sunucuya scp ile kopyalayıp doğrudan çalıştırabiliyorsunuz. Özellikle demo ortamları, geçici lab kurulumları ya da farklı makinelerde hızlı tekrar kurulum gereken senaryolarda bu sadelik ciddi fark yaratıyor.

Bu araç ortaya çıkarken asıl hedefim “yeni bir platform” yazmak değildi. Daha çok, dağınık bash script’leri, unutulan ağ ayarlarını ve her makinede farklı davranan el yapımı kurulumları tek bir akışta toplamak istedim. Sonuçta ortaya çıkan şey, bizim ekip için hem kurulum süresini düşüren hem de hata ayıklamayı ciddi biçimde kolaylaştıran bir çözüm oldu.

Bazen en faydalı araçlar en iddialı olanlar değil, her gün tekrar edilen can sıkıcı işleri ortadan kaldıranlar oluyor. Bu da benim için tam olarak öyle bir projeydi.
---

## Gerçekte Ne Yapıyor?

Aracın yaptığı işi en basit haliyle şöyle özetleyebilirim: tek bir sunucu üzerinde, tek bir binary ve tek bir config dosyasıyla hibrit bir ortam kuruyor.

`orchestrator up` çalıştığında arka planda birkaç adımı sırayla yürütüyor. Önce Docker tarafında gerekli bridge ağı oluşturuyor. Ardından DHCP ve DNS işini üstlenecek container’ları başlatıyor. Sonrasında uygulama container’larıyla birlikte VM’leri ayağa kaldırıyor. Bu aşamada işleri mümkün olduğunca paralel yürüttüğü için, her şey tek tek beklenerek değil eşzamanlı biçimde başlıyor. Kurulum tamamlanırken uzak erişim için WireGuard istemci yapılandırmasını üretiyor ve en son, daha sonra `orchestrator down` çağrıldığında neyin nasıl geri alınacağını bilmek için bir durum dosyası yazıyor.

Aslında burada çözmeye çalıştığı problem oldukça net. Docker size namespace’lerle izole edilmiş container’lar veriyor. Libvirt ise kendi kernel’ine sahip, tam anlamıyla ayrı makineler gibi davranan sanal makineler sunuyor. Tek başına bakınca iki dünya da tanıdık. Zor olan kısım, bunları aynı ağ düzleminde düzgün şekilde bir araya getirmek.

Bu aracın esas işi de tam olarak burada başlıyor: container’ları ve VM’leri aynı Layer 2 bridge üzerinde buluşturmak, üstelik ikisinin de aynı DHCP akışından IP almasını sağlamak. Böylece ortam dışarıdan bakıldığında iki ayrı altyapının yamalı birleşimi gibi değil, tek parça bir sistem gibi davranıyor.

Bunun pratikteki karşılığı önemli. Uzak bir istemci WireGuard üzerinden ortama bağlandığında, karşısında “bu container ağı, şu VM ağı” gibi ayrımlar görmüyor. Servislere, arkada container mı var VM mi diye düşünmeden, şeffaf biçimde erişebiliyor. Bence aracın gerçek değeri de burada: farklı çalışma modellerini tek bir ağ ve yaşam döngüsü altında toplamak.


---

## Gerçekten Nerelerde İşe Yarar?

Baştan net söyleyeyim: bu araç bir Kubernetes alternatifi değil. Zaten öyle olmak için de tasarlanmadı. Bilinçli olarak küçük tutulmuş, ne yaptığını bilen ve belirli senaryolarda gerçekten iş gören bir araç.

Her yere uysun diye şişirilmiş bir platform olmaktan çok, “tek makinede container + VM karışık bir ortamı hızlı, tekrar üretilebilir ve temiz şekilde ayağa kaldırayım” problemine odaklanıyor. Bu yüzden en çok şu tip durumlarda anlamlı hale geliyor:

### Eğitim ve Workshop Ortamları

İki günlük bir güvenlik workshop’u düzenlediğinizi düşünün. Her katılımcının kendi izole ağı var. O ağın içinde bir web uygulaması container olarak çalışıyor, yanında kurcalanacak savunmasız bir VM bulunuyor, bir de içerideki servis isimlerini çözen DNS gerekiyor.

Bunu klasik yöntemlerle yapmak genelde iki yere çıkıyor: ya bulut tarafında kişi başı VPC benzeri yapılar açıyorsunuz ve maliyet hızla büyüyor, ya da herkese Vagrant benzeri bir kurulum verip “umarım laptop kaldırır” noktasına geliyorsunuz. Teoride güzel, pratikte sabahın ilk yarısı kurulum sorunlarıyla geçiyor.

Bu araçta yaklaşım daha sade. Güçlü tek bir paylaşımlı sunucu üzerinde her öğrenciye kendi YAML tanımı ve kendi WireGuard tüneli veriliyor. Herkes aynı fiziksel kaynak üzerinde, ama birbirinden izole ortamlarda çalışıyor. Eğitmen tarafında da yönetmesi çok daha kolay oluyor.

### Edge / IoT Prototipleme

Edge ya da IoT tarafında prototip geliştirirken ihtiyaçlar genelde biraz garip oluyor. Bazı parçalar container içinde rahatça çalışıyor, ama firmware testi ya da daha düşük seviye sistem davranışları için tam bir Linux VM gerekiyor. Böyle bir durumda kimsenin ilk refleksi “buraya bir Kubernetes kuralım” olmuyor.

İstenen şey genelde çok daha basit: binary’yi cihaza at, config dosyasını bırak, tek komutla ortamı ayağa kaldır.

```bash 
GOOS=linux GOARCH=arm64 go build -o orchestrator .
```

Cross-compile tarafının kolay olması da burada ciddi avantaj. Özellikle ARM tabanlı edge cihazlarda, ekstra paket yöneticisi ya da karmaşık runtime bağımlılıklarıyla uğraşmadan doğrudan çalıştırabilmek büyük rahatlık.

### VM Gerektiren CI Entegrasyon Testleri

Bazı şeyler gerçekten container içinde doğru dürüst test edilemiyor. Kernel modülleri, özel ağ davranışları, gerçek boot sırası bekleyen servisler ya da sistemin gerçekten “makine gibi” davranmasını gerektiren senaryolar bunların başında geliyor.

Bugün birçok CI pipeline’ında bu testler ya tamamen pas geçiliyor ya da pahalı ve kırılgan nested virtualization çözümleriyle yürütülüyor. İkisi de çok iç açıcı değil.

Bu tür bir araç ise pipeline’ın başında hibrit ortamı kurup iş bitince düzgünce kaldırmak için uygun bir model sunuyor. Özellikle up ve down tarafının idempotent olacak şekilde düşünülmesi burada önemli; çünkü CI dünyasında aynı şeyi tekrar tekrar, temiz biçimde yapabilmek her şeyden daha değerli.

### Hızlı Demo Ortamları

Satış mühendisleri, çözüm mimarları, destek ekipleri… Kısacası bir ürünü ya da sistemi “çalışır halde” göstermek zorunda olan herkes benzer bir dert yaşıyor. Demo ortamı genelde bir noktada kırılgan bir kar topuna dönüşüyor. Bir VM imajı var, onun üstünde elle yapılmış ayarlar var, kimsenin tam hatırlamadığı bir network workaround’u var; dokununca bozuluyor ama dokunmadan da yaşlanıyor.

Bunun yerine ortamı versiyon kontrollü bir YAML dosyasında tarif etmek çok daha temiz bir yaklaşım. Gerektiğinde birkaç saniye içinde yeniden kurabiliyorsunuz. Demo makinesine “aman buna dokunmayın” muamelesi yapmak yerine, gerektiğinde sıfırdan üretilebilen bir yapıya geçmiş oluyorsunuz.

### Homelab Orkestrasyonu

Bir de işin homelab tarafı var. Evde bare-metal sunucu ya da küçük bir Proxmox kutusu çalıştıranlar bu fikre hemen yakın hissedebilir. Bazen tek ihtiyacınız, tek node üzerinde tekrar kurulabilir container + VM yığınlarıdır. Ama bunun için tam boy bir hyperconverged platformun bütün ağırlığını da sırtlanmak istemezsiniz.

Bu durumda daha hafif, doğrudan ve anlaşılır bir araç daha mantıklı olabiliyor. Özellikle “tek makine, birkaç servis, bir iki VM, düzgün ağ izolasyonu ve temiz lifecycle yönetimi” çizgisinde kalıyorsanız, küçük bir çözüm çoğu zaman büyük platformlardan daha kullanışlı oluyor.

---

## Nasıl Kullanılır?

Bu aracın bütün mantığı config dosyasında yaşıyor. İşin güzel tarafı da burada: ortamı kuran şey dağınık script’ler değil, tek bir yerde duran açık bir tanım.

```yaml
network_name: demo-net
subnet: 172.19.5.0/24
network_type: bridge

containers:
  - name: web-demo
    image: nginx:alpine
    ip: 172.19.5.10

  - name: whoami
    image: containous/whoami:latest
    ip: 172.19.5.11

vms:
  - name: demo-vm
    image: ./images/debian-12.qcow2
    memory_mb: 1024
    vcpus: 2
    packages: [curl, wget, vim, htop, net-tools, nmap]

wireguard:
  enabled: true
  peer_name: demo-client
  address: 10.10.0.2/24
```


Buradaki fikir basit: ortamın neye benzediğini tek tek komutlarla değil, deklaratif bir dosyayla tarif ediyorsunuz. Ağ adı ne, subnet ne, hangi container’lar çalışacak, hangi VM açılacak, içine hangi paketler yüklenecek, WireGuard olacak mı — hepsi burada.

Bu dosya bir bakıma “kurulum notu” değil, doğrudan ortamın kendisi.

En iyi şekilde kullanmak için birkaç pratik not:

**Config’i versiyonlayın, ortamı değil.**

Bence işin en kritik noktası bu. Ortamı kafada ya da bir wiki sayfasında tutmaya çalışmak yerine config dosyasını Git’e koyuyorsunuz. Sonra biri gelip “geçen çeyrekte müşteriye gösterdiğimiz demo tam olarak nasıldı?” diye sorduğunda, kimsenin hafızasına güvenmiyorsunuz.


```bash 
git log
git show <commit>
```
**Adres referanslanan servislerde statik IP kullanın.**

Başka servislerin bağımlı olduğu bileşenler — mesela web frontend, API gateway ya da dahili DNS gibi şeyler — sabit IP ile tanımlanmalı. Böylece “o servis bugün hangi IP’yi aldı?” diye dolaşmanız gerekmiyor. Ama gerçekten önemi olmayan, kimsenin doğrudan adreslemediği worker benzeri işler için dinamik tahsis gayet yeterli.

**VM tarafında paketleri baştan hazırlamak ciddi zaman kazandırıyor.** 

`packages` alanı boşuna yok. Araç bunu görüp ilk boot sırasında gerekli paketleri cloud-init üzerinden yüklüyor. Yani VM açıldığında curl, wget, nmap, vim gibi temel araçlar zaten hazır oluyor. Özellikle internet erişimi sınırlı ortamlarda ya da “bu makine dışarı çıkamayacak” senaryolarında bu küçük detay ciddi fark yaratıyor.

**WireGuard yapılandırması doğrudan kullanılabilir dosya olarak üretiliyor.**

orchestrator up tamamlandıktan sonra elinizde istemciye verebileceğiniz hazır bir konfigürasyon oluyor. İnsanların tek tek anahtar üretmesi, peer eklemesi, “şu public key’i bana atsana” diye mesajlaşması gerekmiyor.

```bash 
orchestrator up -c config.yaml
ls wg-client-*.conf
```
Dosyayı import ediyorlar ve ortama bağlanıyorlar. En sevdiğim kullanım türlerinden biri bu, çünkü gerçekten sürtünmeyi azaltıyor.


---

### Neleri Eksik? (Dürüst Olmak Gerekirse)

Konferans sunumunda tabii ki daha parlak taraflarını gösterdim. Ama işin dürüst kısmı şu: araç kullanışlı olsa da bazı yerleri hâlâ köşeli. Hatta bazı eksikleri özellikle not etmek istiyorum, çünkü bunlar gerçek dünyada bir noktada karşınıza çıkıyor.

### Düzgün Kapanış Zinciri Yok

Şu anda orchestrator down çağrıldığında ortam kapanıyor, kaynaklar temizleniyor, işler toparlanıyor. Ama bütün bunlar bilinçli bir bağımlılık sırasıyla yapılmıyor.

```bash 
orchestrator down
```
Bu şu anlama geliyor: eğer bir VM’in düzgün kapanmadan önce ağdaki başka bir servise ulaşması gerekiyorsa ya da veri flush etmesi için belli bir bileşenin biraz daha ayakta kalması şartsa, sistem bunu anlamıyor. Kısacası “önce bunu kapat, sonra şunu indir” gibi akıllı bir kapanış grafiği henüz yok.

Yapı taşları aslında var. Context iptali var, WaitGroup mantığı var, eşzamanlı işlemler kontrol altında. Ama bağımlılık sırasını yöneten gerçek bir shutdown orchestration katmanı henüz eklenmiş değil. Muhtemelen doğru yaklaşım, başlatma sırasının tersini uygulayan bir DAG mantığı olurdu.


### Yalnızca Tek Sunucu

Bu araç en başından beri tek sunucu varsayımıyla yazıldı. Zaten tasarım hedefi de buydu: bir makine, bir binary, bir config, tek komut.

Ama bunun doğal sınırı şu: VM’leri farklı sunuculara dağıtmak, ağı federasyon yapmak, işleri cluster mantığıyla yaymak gibi kavramlar yok. Eğitim ortamı, hızlı demo, küçük CI senaryoları ve homelab için bu çoğu zaman sorun değil. Hatta sadelik sağlıyor. Ama ölçeği büyütmek istediğiniz anda sınır çizgisi gayet netleşiyor.

### "Çalışıyor mu?" Ötesinde Sağlık Kontrolü Yok

Bugünkü sağlık kontrolü temelde şuna bakıyor: container ya da VM ayağa kalktı mı, süreç running durumda mı?

Yani sistem “çalışıyor” ile “işini doğru yapıyor” arasındaki farkı henüz bilmiyor.

Mesela nginx container’ı açılmış olabilir ama yanlış config yüzünden 502 dönüyordur. Ya da servis portu açılmıştır ama uygulama içeride kendine gelmemiştir. Bu durumda orchestrator status size kötü bir haber vermez; dışarıdan bakınca her şey yolundaymış gibi görünür.

```bash
orchestrator status
```
HTTP probe, TCP check, readiness gate ya da uygulama seviyesinde health validation gibi şeyler henüz yok. Açık söylemek gerekirse, bu eklendiği anda araç bir üst seviyeye çıkmış olacak.

**VM Ağ Yapılandırması Kırılgan**

Container’ları aynı bridge’e almak kolay kısım. VM’i aynı yapıya düzgün bağlamak ise hâlâ biraz “burası Linux networking, burada kimse masum değil” kategorisinde.

Bridge’in promiscuous modda olması gerekiyor, bağlantının gerçekten oluştuğunu doğrulamak gerekiyor, bridge helper doğru yapılandırılmış olmalı. Araç bunları hallediyor ama işin altında yine biraz hassas sistem davranışı yatıyor. Bir kernel güncellemesi, host tarafında ufak bir fark ya da dağıtıma özel bir değişiklik bu zinciri bozabilir.

```bash 
ip link set <bridge> promisc on
brctl show
```

Daha sağlam bir yaklaşım belki macvlan ya da ayrılmış bir OVS bridge olurdu. Şu anki çözüm çalışıyor, ama “bunu sonsuza kadar düşünmeden bırakırım” türden değil.


### Çok Kiracılık (Multi-Tenancy) Yok

Şu an aynı sunucuda iki farklı kullanıcının birbirinden bağımsız biçimde orchestrator up çalıştırdığını düşünün. Eğer isimlendirme ve ağ tarafı dikkatli tasarlanmamışsa, çakışmalar hemen başlıyor.

```bash
orchestrator up -c ata.yaml
orchestrator up -c turkmen.yaml
```
Container adları çakışabilir, network isimleri üst üste binebilir, port binding’ler birbirine girebilir. Yani mevcut haliyle bu araç “çok kullanıcılı lab platformu” olmaktan çok, kontrollü biçimde kullanılan tek operatörlü bir araç.

Kullanıcı ya da oturum bazlı namespace mantığı eklense — isim önekleri, izole alt ağlar, session ID tabanlı kaynak adları gibi — özellikle workshop tarafında çok daha rahat kullanılabilir hale gelirdi.


### Durum Yönetimi İlkel

Durum dosyası var, ama şu anda oldukça düz bir model. Temelde JSON halinde “ne oluşturuldu” bilgisini yazıyor ve down sırasında buna güveniyor.

Bu basitlik belli bir noktaya kadar güzel. Ama binary provisioning sırasında çökerse ya da host üstünde dışarıdan bir değişiklik olursa, durum dosyasıyla gerçek dünya birbirinden kopabiliyor.

Yani şu iki soru arasında henüz otomatik bir uzlaşma döngüsü yok:
	•	Gerçekte ne çalışıyor?
	•	Sisteme göre ne çalışıyor olması gerekiyor?

Bugünkü model daha çok “bir kez çalıştır, sonucu kaydet, sonra oraya güven” yaklaşımı. Terraform benzeri bir reconciliation mantığıyla kıyaslayınca, burada geliştirilecek ciddi alan olduğu açık.

---

## İlginç Teknik Detaylar

Her slaydı tekrarlamak istemiyorum (bunun için [sunum](/talks/orchestrator.html) var), ama birkaç tasarım kararı beni şaşırttığı için bahsetmeye değer:

**IPv4 adreslerini `uint32` olarak temsil etmek her şeyi değiştiriyor.** `172.19.5.10`'u tek bir 32-bit tam sayı olarak ifade ettiğinizde, IP havuzu yönetimi aritmetiğe dönüşür: aralık kontrolleri karşılaştırmadır, broadcast hesaplaması bitwise OR'dur ve bir alt ağı dolaşmak sadece artırmaktır. Go'nun `encoding/binary` ve `net` paketleri bunu kolaylaştırır — üçüncü parti IP manipülasyon kütüphanesine gerek yok.

**IP havuzunun çift stratejisi "verini tanı" ilkesinin güzel bir örneği.** Bir `/24` alt ağ (254 host) için tüm mevcut IP'lerin bir haritasını önceden oluşturmayı göze alabilirsiniz. Bir `/8` (16 milyon host) için bu çılgınlık — sadece başlatmak 5 saniye ve ~1 GB bellek alır. Havuz büyük alt ağlarda şeffaf şekilde tembel stratejiye geçer: yalnızca *kullanılmış* IP'leri takip et, rastgele adaylar üret ve dene. Sonuç: `/8` başlatma 5,1 saniyeden 0,004 saniyeye düştü.

**CGo kullanmamak kesin bir kısıttı.** Ana Go libvirt binding'i (`libvirt-go`) C kütüphanesini sarar, yani `libvirt-dev` yüklü olması gerekir ve statik binary'den vazgeçersiniz. `virsh` CLI'ı `os/exec` üzerinden kullanmak kuşkusuz daha kırılgan (metin çıktısını parse ediyorsunuz), ama binary'yi taşınabilir tutar. WireGuard anahtar üretimi için aynı mantık — `golang.org/x/crypto/curve25519`, OpenSSL veya `wg genkey` ihtiyacını ortadan kaldırır.

**Goroutine-ID log'lama, bir debug hack'iydi ve özelliğe dönüştü.** 6 goroutine aynı anda log yazarken, bir korelasyon olmadan neler olduğunu anlamak imkânsız. `runtime.Stack` çıktısından goroutine ID'sini çıkarmak hack'çe (bir debug string parse ediyorsunuz), ama işe yarıyor ve paralel yürütmeyi log çıktısında görünür kılıyor. Farklı ID'ler + örtüşen zaman damgaları = işlerin sadece eşzamanlı *görünmediğinin*, gerçekten eşzamanlı olduğunun kanıtı.

---

## Henüz Keşfetmediğim Fikirler

Bunlar daha fazla zamanım olsa yapacağım şeyler:

**Bir `plan` komutu (Terraform gibi).** `up`'tan önce tam olarak nelerin oluşturulacağını göster — "X ağı oluşturulacak, A, B, C container'ları başlatılacak, D VM'i tanımlanacak." Kullanıcı inceleyip onaylasın. Özellikle prodüksiyona yakın senaryolarda kullanışlı.

**Config değişikliğinde canlı yeniden yükleme.** YAML dosyasını izle ve çalışan durumu uzlaştır. Config'e yeni bir container mi eklendi? Geri kalanı yıkmadan ayağa kaldır. Bir VM mi kaldırıldı? Kapat. Bu özünde bir kontrol döngüsü ve Go'nun `fsnotify` + goroutine'leri bunu doğal olarak inşa etmeye uygun.

**İş yükü türleri için plugin sistemi.** Şu anda araç tam olarak iki tür iş yükü biliyor: Docker container'ları ve libvirt VM'leri. Ama kalıp genelleştirilebilir — Podman, Firecracker veya hatta Kata Containers için plugin'ler düşünülebilir. `Start()`, `Stop()`, `Status()` metodları olan bir `WorkloadDriver` arayüzü temiz bir soyutlama olurdu.

**Prometheus metrik endpoint'i.** Mevcut container sayısı, VM sayısı, IP havuzu kullanım oranı, provisioning gecikmesi bilgilerini dışa aç. Uzun süre çalışan bir lab sunucusu için kapasite planlaması açısından gerçekten yararlı olurdu.

**VM'ler için snapshot ve rollback.** Bir checkpoint tanımla, öğrencilerin/test edenlerin her şeyi bozmasına izin ver, sonra bilinen iyi duruma geri dön. Libvirt snapshot'ları doğal olarak destekler — tesisat basit, sadece bağlanmamış.

**gRPC API.** CLI interaktif kullanım için iyi, ama bunu daha büyük bir sisteme (bir eğitim platformuna, bir CI kontrolcüsüne) entegre etmek istiyorsanız, gRPC API uzak istemcilerin ortamları programatik olarak orkestre etmesini sağlar. Cobra CLI ve `Orchestrator` struct'ı zaten temiz bir şekilde ayrılmış durumda, dolayısıyla `Up()`, `Down()` ve `Status()`'u gRPC handler'larına sarmak mekanik bir iş.

**Çoklu sunucu için WireGuard mesh.** Tek yıldız topolojisi yerine, birden fazla orchestrator örneğinin mesh oluşturmasına izin ver. Her biri anahtar üretir, public key'lerini ve endpoint'lerini değiştirir ve sunucular arasında düz bir ağ elde edersiniz. Eğer bir gün mantıklı olursa, dağıtık versiyona giden yol bu.

---

## Son Düşünceler

Bu araç, Kubernetes'in tek sunucu için fazla olması ve shell script'lerinin bakımsız kalması yüzünden var. Tatlı nokta, tek bir işi iyi yapan sıkıcı bir Go binary'si: tekrarlanabilir bir hibrit ortamı ayağa kaldırmak ve temizce kapatmak.

Go bu iş için doğru dil çıktı — tek bir özellik yüzünden değil, altyapı araçlarının statik binary, ucuz eşzamanlılık, birinci sınıf ağ desteği ve güçlü standart kütüphane *kombinasyonunu* istemesi yüzünden. Ekosistemdeki her büyük altyapı projesi (Docker, Kubernetes, Terraform, Prometheus) aynı bahsi yaptı.

Bunlardan herhangi biri iş akışınız için kullanışlı geliyorsa, kod açık ve binary kendi kendine yeterli. Deneyin, kırın ve nelerin eksik olduğunu söyleyin.

**[📊 Sunum slaytları](/talks/orchestrator.html)** · **[📊 Slides (English)](/talks/orchestrator_en.html)** · **[💻 GitHub](https://github.com/mrtrkmn/orchestrator)** · **[LinkedIn](https://www.linkedin.com/in/mrturkmen/)**
