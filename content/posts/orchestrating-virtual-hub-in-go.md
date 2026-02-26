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

**[📊 Sunum slaytları](/talks/orchestrator.html)** · **[💻 Kaynak kod](https://github.com/mrtrkmn/orchestrator)**

---

## Kaşıntı

Eğer bir gün tek bir makinede hem container'lar hem de VM'ler içeren çok katmanlı bir demo ortamı kurmak zorunda kaldıysanız, hikayeyi biliyorsunuzdur. Önce `docker network create` çağıran bir shell script yazarsınız. Sonra `dhcpd.conf` üreten bir tane daha. Sonra elle `virsh` XML tanımı hazırlarsınız. Sonra VM'in IP alamadığını fark edersiniz — çünkü bridge promiscuous modda değildir. Bir `ip link set` komutu eklersiniz. Sonra birisi bunu başka bir makinede tekrarlamanızı ister ve yarısının dizüstü bilgisayarınıza hardcoded olduğunu keşfedersiniz.

İş yerinde tam olarak bu sorunla karşılaştım. İzole hibrit ortamlara ihtiyacımız vardı — bazı servisler container'da koşuyordu, ama bazı iş yükleri gerçekten kendi kernel'ine sahip bir VM gerektiriyordu. Manuel kurulum 15-20 dakika sürüyordu ve neredeyse hiçbir zaman aynı şekilde çalışmıyordu. Ortamı kaldırmak daha da kötüydü. Bazı container'lar geride kalıyordu, bridge ağı yeniden başlatmalara dayanıyordu ve artık VM tanımları bir sonraki çalıştırmayla çakışıyordu.

Bu yüzden tek seferde hallettiren bir araç yazdım:

```bash
orchestrator up -c config.yaml    # her şey ayağa kalkar
orchestrator status               # nelerin çalıştığını gör
orchestrator ssh demo-vm          # VM'e SSH (IP otomatik algılanır)
orchestrator down                 # temiz kaldırma
```

Tamamı ~15 MB'lık statik bir binary'ye derleniyor, çalışma zamanı bağımlılığı yok — sunucuya `scp` ile atıp çalıştırıyorsunuz.

---

## Gerçekte Ne Yapıyor?

Mimari şu şekilde. Bir sunucu, bir binary, bir config dosyası:

<svg viewBox="0 0 760 480" xmlns="http://www.w3.org/2000/svg" style="max-width:720px;margin:1.5em auto;display:block;font-family:'Segoe UI','Helvetica Neue',Arial,sans-serif;">
  <defs>
    <marker id="arrowD" markerWidth="10" markerHeight="7" refX="5" refY="7" orient="auto"><path d="M0,0 L5,7 L10,0" fill="#06b6d4"/></marker>
    <linearGradient id="bg" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="#0f172a"/><stop offset="100%" stop-color="#1e293b"/></linearGradient>
    <linearGradient id="bridgeGrad" x1="0" y1="0" x2="1" y2="0"><stop offset="0%" stop-color="rgba(6,182,212,0.08)"/><stop offset="50%" stop-color="rgba(6,182,212,0.22)"/><stop offset="100%" stop-color="rgba(6,182,212,0.08)"/></linearGradient>
    <filter id="glow"><feGaussianBlur stdDeviation="2" result="g"/><feMerge><feMergeNode in="g"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
    <filter id="shadow"><feDropShadow dx="0" dy="2" stdDeviation="3" flood-color="#000" flood-opacity="0.35"/></filter>
  </defs>
  <rect width="760" height="480" rx="16" fill="url(#bg)"/>
  <rect x="16" y="16" width="728" height="350" rx="14" fill="none" stroke="rgba(148,163,184,0.25)" stroke-width="1.5" stroke-dasharray="6 4"/>
  <text x="32" y="38" fill="#94a3b8" font-size="11" font-weight="600" letter-spacing="0.5">SERVER</text>
  <rect x="40" y="60" width="138" height="80" rx="10" fill="rgba(6,182,212,0.07)" stroke="#06b6d4" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="109" y="85" text-anchor="middle" fill="#06b6d4" font-size="13" font-weight="700">nginx:alpine</text>
  <text x="109" y="103" text-anchor="middle" fill="#94a3b8" font-size="10">container</text>
  <text x="109" y="128" text-anchor="middle" fill="#22d3ee" font-size="14" font-weight="700">.5.10</text>
  <rect x="198" y="60" width="138" height="80" rx="10" fill="rgba(6,182,212,0.07)" stroke="#06b6d4" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="267" y="85" text-anchor="middle" fill="#06b6d4" font-size="13" font-weight="700">whoami</text>
  <text x="267" y="103" text-anchor="middle" fill="#94a3b8" font-size="10">container</text>
  <text x="267" y="128" text-anchor="middle" fill="#22d3ee" font-size="14" font-weight="700">.5.11</text>
  <rect x="356" y="60" width="138" height="80" rx="10" fill="rgba(6,182,212,0.07)" stroke="#06b6d4" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="425" y="82" text-anchor="middle" fill="#06b6d4" font-size="13" font-weight="700">DHCP / DNS</text>
  <text x="425" y="100" text-anchor="middle" fill="#94a3b8" font-size="10">container</text>
  <text x="425" y="128" text-anchor="middle" fill="#22d3ee" font-size="14" font-weight="700">.5.2 / .5.3</text>
  <rect x="530" y="54" width="194" height="92" rx="12" fill="rgba(244,63,94,0.06)" stroke="#f43f5e" stroke-width="1.5" stroke-dasharray="7 3" filter="url(#shadow)"/>
  <text x="627" y="80" text-anchor="middle" fill="#fb7185" font-size="13" font-weight="700">Debian VM</text>
  <text x="627" y="98" text-anchor="middle" fill="#94a3b8" font-size="10">cloud-init · 1024 MB · 2 vCPU</text>
  <text x="627" y="118" text-anchor="middle" fill="#94a3b8" font-size="10">curl, wget, vim, htop, nmap</text>
  <text x="627" y="136" text-anchor="middle" fill="#fb7185" font-size="10">DHCP assigned IP</text>
  <line x1="109" y1="140" x2="109" y2="170" stroke="#06b6d4" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="267" y1="140" x2="267" y2="170" stroke="#06b6d4" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="425" y1="140" x2="425" y2="170" stroke="#06b6d4" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="627" y1="146" x2="627" y2="170" stroke="#f43f5e" stroke-width="1.2" stroke-dasharray="4 3" marker-end="url(#arrowD)"/>
  <rect x="30" y="174" width="700" height="40" rx="8" fill="url(#bridgeGrad)" stroke="#06b6d4" stroke-width="1.5"/>
  <text x="380" y="200" text-anchor="middle" fill="#e2e8f0" font-size="14" font-weight="700" filter="url(#glow)">Docker Bridge — 172.19.5.0/24</text>
  <text x="380" y="238" text-anchor="middle" fill="#64748b" font-size="10">gateway .5.1 · mask 255.255.255.0 · DHCP range .5.4 – .5.254</text>
  <line x1="380" y1="214" x2="380" y2="260" stroke="#22d3ee" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <rect x="190" y="264" width="380" height="56" rx="12" fill="rgba(34,211,238,0.06)" stroke="#22d3ee" stroke-width="1.5" filter="url(#shadow)"/>
  <rect x="212" y="282" width="14" height="11" rx="2.5" fill="none" stroke="#22d3ee" stroke-width="1.5"/>
  <path d="M215,282 L215,277 a4,4.5 0 0,1 8,0 L223,282" fill="none" stroke="#22d3ee" stroke-width="1.5"/>
  <text x="240" y="293" fill="#22d3ee" font-size="13" font-weight="700">WireGuard VPN Tunnel</text>
  <text x="240" y="310" fill="#94a3b8" font-size="10">10.10.0.0/24 · X25519 key pair · wg-quick compatible</text>
  <line x1="380" y1="320" x2="380" y2="366" stroke="#22d3ee" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="380" y1="366" x2="380" y2="410" stroke="#f43f5e" stroke-width="1.2" stroke-dasharray="5 3" marker-end="url(#arrowD)"/>
  <rect x="305" y="416" width="150" height="50" rx="12" fill="rgba(244,63,94,0.08)" stroke="#f43f5e" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="380" y="440" text-anchor="middle" fill="#fb7185" font-size="13" font-weight="700">Remote Client</text>
  <text x="380" y="456" text-anchor="middle" fill="#94a3b8" font-size="10">wg-client.conf</text>
  <rect x="600" y="410" width="12" height="12" rx="3" fill="rgba(6,182,212,0.15)" stroke="#06b6d4" stroke-width="1"/>
  <text x="618" y="420" fill="#94a3b8" font-size="10">Container</text>
  <rect x="600" y="430" width="12" height="12" rx="3" fill="rgba(244,63,94,0.12)" stroke="#f43f5e" stroke-width="1" stroke-dasharray="3 2"/>
  <text x="618" y="440" fill="#94a3b8" font-size="10">VM (libvirt/KVM)</text>
  <rect x="600" y="450" width="12" height="12" rx="3" fill="rgba(34,211,238,0.12)" stroke="#22d3ee" stroke-width="1"/>
  <text x="618" y="460" fill="#94a3b8" font-size="10">VPN Tunnel</text>
</svg>

`orchestrator up` çalıştırdığınızda sırasıyla bir Docker bridge oluşturur, DHCP ve DNS container'larını başlatır, uygulama container'larınızı ve VM'lerinizi eşzamanlı olarak ayağa kaldırır (goroutine'ler sayesinde her şey paralel koşar), WireGuard istemci yapılandırması üretir ve `orchestrator down`'ın her şeyi temizce geri alabilmesi için bir durum dosyası yazar.

Docker size namespace ile izole edilmiş container'lar verir. Libvirt ise *ayrı bir kernel'e* sahip sanal makineler. İkisini aynı Layer 2 bridge üzerinde, aynı DHCP sunucusuyla buluşturmak — işte aracın asıl değer önerisi bu. Uzak istemci WireGuard üzerinden bağlanır ve container ya da VM fark etmeksizin her servise şeffaf şekilde erişir.

---

## Gerçekten Nerelerde İşe Yarar?

Bir Kubernetes alternatifi yapmadım. Bu araç bilinçli olarak küçük ve fikirli tasarlandı ve dar bir senaryo kümesinde gerçekten işe yarıyor:

### Eğitim ve Workshop Ortamları

İki günlük bir güvenlik workshop'u düzenlediğinizi düşünün. Her katılımcının izole bir ağa, içinde bir web uygulaması (container), üzerinde pratik yapılacak savunmasız bir VM'e ve dahili isimleri çözen DNS'e ihtiyacı var. Bugün ya bir bulut sağlayıcı kullanıp öğrenci başına VPC açarsınız (pahalı), ya da herkese bir Vagrant kurulumu verip dizüstü bilgisayarlarının kaldırmasını dua edersiniz (güvenilmez). Güçlü paylaşımlı bir sunucuda her öğrenci kendi YAML config'ini ve bir WireGuard tünelini alır.

### Edge / IoT Prototipleme

Edge'de prototipleme yapıyorsanız — belki bazı servisleri container'da çalıştırması ama firmware test etmek için tam bir Linux VM barındırması gereken bir gateway cihazı — Kubernetes deploy etmek istemezsiniz. Binary'yi `scp` ile atıp, bir YAML dosyası bırakıp, tek komut çalıştırmak istersiniz. Cross-compile hikayesi bunu kolaylaştırır: `GOOS=linux GOARCH=arm64 go build -o orchestrator .`

### VM Gerektiren CI Entegrasyon Testleri

Bazı yazılımlar gerçekten container'da test edilemez. Kernel modülleri, özel ağ yığınları, gerçek bir boot sırası gerektiren her şey. Bugün çoğu CI pipeline'ı bu testleri ya atlar ya da pahalı iç içe sanallaştırma kullanır. Böyle bir araç, pipeline'ın başında hibrit ortamı kurup sonunda yıkabilir. İdempotent `up`/`down` semantiği tam olarak bunun için tasarlandı.

### Hızlı Demo Ortamları

Satış mühendisleri, çözüm mimarları, destek ekipleri — tek bir makinede çoklu servis kurulumunu çalışır halde göstermesi gereken herkes. Kırılgan bir demo VM imajı bakımı yerine, ortamı versiyon kontrollü bir YAML dosyasında tanımlayıp saniyeler içinde ayağa kaldırırsınız.

### Homelab Orkestrasyonu

Evde Proxmox veya bare-metal sunucu çalıştıran insanlar için — tek bir node olduğunda ve tam bir hyperconverged kurulumun yükü olmadan tekrarlanabilir container+VM yığınları istediğinizde daha hafif bir alternatif.

---

## Nasıl Kullanılır?

Config dosyası tüm düşünmenin yapıldığı yer:

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

En iyi şekilde nasıl kullanılacağına dair birkaç not:

**Config'i versiyonlayın, ortamı değil.** Tüm mesele config dosyasının ortamın *kendisi* olması. Git'e commit'leyin. Birisi "geçen çeyrekteki demo kurulumu nasıldı?" diye sorduğunda, hangi container'ları çalıştırdığınızı hatırlamaya çalışmak yerine `git log` bakarsınız.

**Referans etmeniz gereken servisler için statik IP kullanın.** Diğer servislerin bağımlı olduğu container'lar (web frontend'iniz, API gateway'iniz) sabit IP almalı. Araç hem statik atama hem de havuzdan rastgele tahsis destekler — kimsenin doğrudan adresleme yapmadığı worker node'lar gibi şeyler için rastgeleliği kullanın.

**VM'lere paketler önceden yüklenir.** VM config'indeki `packages` listesi özel bir imaj derlemesi tetikler — araç cloud-init ile bu paketleri ilk boot'ta yükler. Yani VM'iniz `curl`, `nmap` vb. zaten yüklü şekilde başlar; bu özellikle çevrimdışı veya air-gapped ortamlar için büyük avantaj.

**WireGuard config'leri istemciye hazır dosya olarak üretilir.** `orchestrator up` sonrasında, birine doğrudan verebileceğiniz bir `wg-client-<isim>.conf` dosyası bulursunuz. WireGuard istemcisine import ederler ve içerdeler — elle anahtar değişimi yok.

---

## Neleri Eksik? (Dürüst Olmak Gerekirse)

Bir konferansta sundum, dolayısıyla parlak kısımları gösterdim. İşte pürüzlü taraflar:

### Düzgün Kapanış Zinciri Yok

Şu anda `orchestrator down` her şeyi durdurur ve kaldırır, ancak sıralı kapanış yok. VM'inizin DHCP container'ı kaybolmadan önce veri flush etmesi gerekiyorsa, şansınız yok. Düzgün bir uygulama bağımlılık sıralaması gerektirir — belki başlatma DAG'ının tersi. Yapı taşları mevcut (context iptali, WaitGroup'lar), ama sıralama mantığı yok.

### Yalnızca Tek Sunucu

Araç her şeyin tek bir sunucuda çalıştığını varsayar. VM'leri birden fazla sunucuya dağıtma veya ağları federasyon yapma kavramı yok. Birçok kullanım senaryosu için bu sorun değil — *zaten* tek sunucu aracı olması amaçlanıyor — ama ölçekleme ihtiyacı olan CI/CD ve eğitim senaryolarını sınırlıyor.

### "Çalışıyor mu?" Ötesinde Sağlık Kontrolü Yok

`select` tabanlı polling döngüsü container'ın "running" durumuna ulaşıp ulaşmadığını kontrol eder, o kadar. Uygulama seviyesinde sağlık kontrolü yok — HTTP probe yok, TCP kontrolü yok, readiness gate yok. nginx container'ınız çalışıyor ama yanlış yapılandırılmış ve 502 döndürüyorsa, `orchestrator status` mutlu bir şekilde her şeyin yolunda olduğunu rapor eder.

### VM Ağ Yapılandırması Kırılgan

Bir VM'i Docker bridge'e bağlamak, aşikâr olmayan bir dizi adım gerektirir — bridge'i promiscuous moda almak, bağlantıyı doğrulamak için `brctl` kullanmak, bridge helper'ın yapılandırıldığından emin olmak. Araç bunu halleder, ama bir kötü kernel güncellemesiyle bozulabilir. Daha sağlam bir yaklaşım macvlan ya da ayrılmış bir OVS bridge kullanabilir.

### Çok Kiracılık (Multi-Tenancy) Yok

Her çalıştırma aynı namespace'i paylaşır. İki kullanıcı aynı sunucuda farklı config'lerle `orchestrator up` çalıştırırsa, container adları, ağ adları ve port bağlamaları çakışır. Kullanıcı veya oturum başına namespace'leme (önek eklenmiş container adları, izole alt ağlar) eklemek, eğitim/workshop kullanım senaryosunu çok daha pratik hale getirirdi.

### Durum Yönetimi İlkel

Durum dosyası düz bir JSON blob. Binary provisioning ortasında çökerse, durum dosyası gerçeği yansıtmayabilir. "Gerçekte ne çalışıyor?" ile "Ne çalışıyor olmalı?" arasını kontrol eden bir uzlaşma döngüsü yok — tek seferlik ateşle-unut yaklaşımı. Bunu Terraform'un durum yönetimiyle karşılaştırırsanız, burada ne kadar çok şey yapılabileceğini görürsünüz.

---

## İlginç Teknik Detaylar

Her slaydı tekrarlamak istemiyorum (bunun için [sunum](/talks/orchestrator.html) var), ama birkaç tasarım kararı beni şaşırttığı için bahsetmeye değer:

**IPv4 adreslerini `uint32` olarak temsil etmek her şeyi değiştiriyor.** `172.19.5.10`'u tek bir 32-bit tam sayı olarak ifade ettiğinizde, IP havuzu yönetimi aritmetiğe dönüşür: aralık kontrolleri karşılaştırmadır, broadcast hesaplaması bitwise OR'dur ve bir alt ağı dolaşmak sadece artırmaktır. Go'nun `encoding/binary` ve `net` paketleri bunu kolaylaştırır — üçüncü parti IP manipülasyon kütüphanesine gerek yok.

**IP havuzunun çift stratejisi "verini tanı" ilkesinin güzel bir örneği.** Bir `/24` alt ağ (254 host) için tüm mevcut IP'lerin bir haritasını önceden oluşturmayı göze alabilirsiniz. Bir `/8` (16 milyon host) için bu çılgınlık — sadece başlatmak 5 saniye ve ~1 GB bellek alır. Havuz büyük alt ağlarda şeffaf şekilde tembel stratejiye geçer: yalnızca *kullanılmış* IP'leri takip et, rastgele adaylar üret ve dene. Sonuç: `/8` başlatma 5,1 saniyeden 0,004 saniyeye düştü.

**CGo kullanmamak kesin bir kısıttı.** Ana Go libvirt binding'i (`libvirt-go`) C kütüphanesini sarar, yani `libvirt-dev` yüklü olması gerekir ve statik binary'den vazgeçersiniz. `virsh` CLI'ı `os/exec` üzerinden kullanmak kuşkusuz daha kırılgan (metin çıktısını parse ediyorsunuz), ama binary'yi taşınabilir tutar. WireGuard anahtar üretimi için aynı mantık — `golang.org/x/crypto/curve25519`, OpenSSL veya `wg genkey` ihtiyacını ortadan kaldırır.

**Goroutine-ID log'lama, bir debug hack'iydi ve özelliğe dönüştü.** 6 goroutine aynı anda log yazarken, bir korelasyon olmadan neler olduğunu anlamak imkânsız. `runtime.Stack` çıktısından goroutine ID'sini çıkarmak hack'çe (bir debug string parse ediyorsunuz), ama işe yarıyor ve paralel yürütmeyi log çıktısında görünür kılıyor. Farklı ID'ler + örtüşen zaman damgaları = işlerin sadece eşzamanlı *görünmediğinin*, gerçekten eşzamanlı olduğunun kanıtı.

---

## Henüz Keşfetmediğim Fikirler

Bunlar daha fazla zamanım olsa veya birisi ihtiyaç duyduğunu söylese yapacağım şeyler:

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

**[📊 Sunum slaytları](/talks/orchestrator.html)** · **[💻 GitHub](https://github.com/mrtrkmn/orchestrator)** · **[LinkedIn](https://www.linkedin.com/in/mrturkmen/)**
