---
title: "debian: terminal/komut "
date: 2019-01-10T09:04:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["linux", "terminal", "komut", "nftp","wget", "scp", "html"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Linuxa başlamak ve basit komutları ögrenmek"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "../../images/Icon-Vim.svg" # image path/url
    alt: "" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---

## Özet:

Bu yazıda linux ortamına biraz daha giriş yaparak, linux ortamında bulunan komutlar hakkında kısa bilgilendirme yapılması planlanmaktadır. 

## Giriş

#### Neden Linux ?

  * Birden fazla işlemi aynı anda kolay şekilde yapmanızı sağlar 
  * Uzaktan işlemlerinizi halletmede büyük kolaylık sağlar
  * Birden fazla kullanıcı aynı sunucuya erişebilir
  * Terminale, bir sistem üzerinde olan kaynaklara birden fazla erişim mümkündür
  * Arayüz olan sistemlere göre daha performanslı, Bedava , Güncel

## Temeller
   
#### Bu bilgilendirme dosyası için not

   -  __Bütün komutlar büyük ve küçük harfe duyarlıdır.__ 

   -  ` "$" `komutun başlangıcını temsil eder.
   - ` "#" ` komutun sonunu temsil eder.

#### Windows bilgisayar üzerinden giriş için


   - __PuTTY__ : windows bilgisayar üzerinden SSH ile baglantı sağlamak için gereklidir.
   - __PuTTY__ programında SSH ile bağlantı için sunucu IP adresi ve PORT numarasını bilmelisiniz.

#### Mac veya Linux Bilgisayarlardan Erişim

Bu tür bilgisayarlar UNIX tabanlı olduğundan dolayı terminal üzerinden aşağıda verilen komutları yazmanız yeterli olacaktır ekstradan herhangi bir programa gerek duyulmamaktadır. 

```bash 
$ ssh <kullanıcı_adı>@<sunucu_adresi(IP)>
$ kullanıcı_adı: ...
$ şifre: ...
{% endhighlight  %}

#### Giriş yaptıktan sonra şifrenizi değiştirmek isterseniz

```bash 
$ passwd # bu komutu kullanabilirsiniz. 
    # bu komut sayesinde giriş yapılan 
    # kullanıcı için yeni şifre 
    # belirleyebilirsiniz.
{% endhighlight  %}


#### Listeleme

```bash 
$ pwd # şu anda bulunduğunuz konumu çıktı olarak yazdırır
$ ls # bulunduğunuz konumdaki dosyaları ve klasörleri listeler
$ ll # ls komutunun eşdeşi olarak tanımlı bir ifadedir genelde "ls -alF" olarak kayıtlıdır bash profilinde

 $ ll -R # dosyalar/klasörler listelenir, klasörler içerisindeki dosyalarda listelenir
 $ ll -t # listeleme işlemi kronolojik şekilde gerçekleşir.
 $ stat <dosya_adı> # dosyaya ait bilgileri meta bilgileri listeler
 $ whoami # sizin sistem tarafından kim olduğunuzu söyler, yani kullanıcı adınızı listeler
 $ hostname # bağlı olduğunu makinenin URL ni yada IP sini gösterir
```

#### Dosyalar ve Klasörler

 ```bash 
 $ mkdir <klasör_adı> # belirlenen isimde klasör oluşturur
 $ cd <klasör_adı> # klasör adı tanımlanan klasöre gidersini.
 $ cd .. # üst klasöre gitmenizi sağlar
 $ cd ../../ # iki üst klasöre gitmenizi sağlar
 $ cd # ana klasörüne gidersiniz
 $ rmdir <klasör_adı> # klasörü siler
 $ rm <dosya_adı> # dosyayı siler
 $ rm -r <klasör_adı> # klasörü ve içerisindeki bütün dosyaları siler
 $ mv <dosyaadı1> <dosyaadı2> # isim değiştirmenizi sağlar, name
 $ mv <dosyaadı> <taşınacak_yol> # dosyayı belirtilen yere taşır
 $ cp <dosyaadı> <kopyalanacak_yol> # dosyayı belirtilen yere kopyalar, eğer klasör kopyalanacak ise -r parametresi eklenir. 
 ```

## Kısayollar


 ```bash 
 $ . # sadece nokta bulunduğunuz dizini ifade eder
 $ ~/ # kullanıcının ana dizinini ifade eder
 $ history # yazmış olduğunuz komutların kaydını tutar ve bu komut ile erişebilirsiniz
 $ !<komut_sıralaması> # daha önce yazmış oldunuz komutu sıralamasının numarasını vererek calıştırabilirsiniz.
 $ yukarı(asagı)_okları # geçmiş komutlar arasında gezmenizi sağlar
 $ <tamamlanmamış_yol_veya_dosyaadı> TAB # Tab a bastığınızda sistem otomatik tamamlama işlemini gerçekleştirir.
 $ <tamamlanmamış komut> SHIFT&TAB # komutu otomatik tamamlar
 $ Ctrl a # imlecin en başa gitmesini sağlar
 $ Ctrl e # imlecin en sona gitmesini sağlar
 $ Ctrl d # imlec altındaki karakteri siler
 $ Ctrl k # imlecin bulunduğu sağlar
 $ Ctrl y # Ctrl k ile alınan içerik yapıştırılır.
 ```

## Yardım Alma

 ```bash 
 $ man # genel yardım
 $ man wc # wc komutu hakkında yardım almanızı sağlar
 $ wc --help # wc komutu hakkında yardım almanızı sağlar
 $ info wc # wc komutu hakkında detaylı bilgi almanız sağlanır
 $ apropos wc # wc komutuna ait bütün yardım dosyaları güncellenir.
 ```

## Aradığınız Dosyayı Bulma Yöntemleri

#### Arama yapmak


```bash 
 $ find -name "*aramakistediğinizdesen*" # girdiğiniz desene göre bulunduğunuz dizinde arama yapmanızı sağlar.
 $ find /usr/local -name "*klas*" # isminin içerisinde klas geçen dosyaları ve klaksörleri listeler.
 $ find /usr/local -iname "*klas*" # yukarıdaki komutuna benzer şekilde, isminin içerisinde klas geçen dosyaları ve klasörleri listeler fakat bu durumda büyük veya küçük harfte olması dikkate alınmaz

 $ find ~ -type f -mtime -2 # 2 gün içerisinde değiştirilmiş bütün dosyaları listeler
 $ locate <aramakistediğinizdesen> # aradığınız dosyayı veya klasörü sistem genelinde arar.
 $ which <uygulama_adı> # uygulamanın nerede bulunduğunu gösterir
 $ whereis <uygulama_adı> # uygulamanın çalıştırılabilir dosyasının yerini gösterir
 $ dpkg -l | grep aramakistediğinizpaketismi # Debian paketleri içerisinde arama yaparak verilen desende bulunan paketleri listel
```

#### Dosya içerisinde arama yapmak

 ```bash 
 $ grep aranan_kelime dosya # dosya içerisinde aranan kelimenin nerelerde geçtiğini size aktarır
 $ grep -H aranan_kelime # -H çıktı dosyasını aranan kelimenin önüne koyar
 $ grep 'aranan_kelime' dosya | wc # burada iki komut birleştirilmiştir, yani grep komutunun çıktısı wc komutuna girdi olmaktadir yani aranan kelime verilen dosyada 5 kere geçiyor ise bu durumda sonuc 5 olarak dönmektedir. 
 $ find /home/kullanıcı_adı -name '*.txt' | xargs grep -c ^.* # verilen dizinde txt dosyalarını bularak bu dosyalar içerisindeki satır sayısını hesaplayarak çıktı vermektedir. 
 ```


## İzinler & Hak Sahipliği


```bash 
 $ ls -al # bu komut çalıştırıldıgında buna benzer bir çıktı görünebilir : drwxrwxrwx
```

Burada bahsi geçen harflerin anlamları aşağıdaki gibi verilebilir.
 - __d:__ dizin
 - __rwx:__ oku, yaz, çalıştır
 - __ilk üçlü (rwx)__ : kullanıcı izinleri(u)
 - __ikinci üçlü__: grup izinleri (g)
 - __üçüncü üçlü__: diğer izinleri (o) ifade etmektedir. 
__En başta d olursa bu o dosyanın aslında bir klasör olduğunu ifade etmektedir.__

Kullanıcı ve grupa, yazma ve çalıştırma izni vermek:

 ```bash 
 $ chmod ug+rx dosya_ismi 
 ```

#### Kullanıcı haklarının alınması


 ```bash 
 
 $ chmod ugo-rwx dosya_ismi

 '+' izin eklemeyi sağlar
 '-' izin silmenizi sağlar

 $ chmod +rx dosya_ismi/ VEYA $ chmod 755 dosya_ismi/

 ```

#### Hak sahipliğinin değiştirilmesi 

 ```bash 
 $ chown <kullanıcı_adı> <dosya veya klasör> # kullanıcı hak sahibini değiştirir
 $ chgrp <grup> <dosya veya klasör> # grup hak sahibini değiştirir
 $ chown <kullanıcı_adı>:<grup> <dosya veya klasör> # kullanıcı ve grup hak sahipliğini değiştirir
 ```


## Kullanışlı Linux Komutları

 ```bash 
 $ df # sistem diskinin ne kadar dolu ve boş olduğu bilgisini gösterir
 $ free # ne kadar önbellek (RAM) alanının boş/dolu olduğunu gösterir
 $ uname -a # işletim sistemine ait temel bilgileri gösterir
 $ bc # terminal üzerinden hesap makinesi kullanmanızı sağlar
 $ /sbin/ifconfig # sunucunun ağ bilgilerini listeler.
 $ ln -s orjinal_dosyaismi yeni_dosyaismi # orjinal dosyaya link oluşturur
 $ du -sh # bulunduğunuz konumdaki disk kullanım bilgilerini listeler
 $ du -sh * # bulunduğunuz konumdaki dosyaların/klasörlerin kullanım bilgilerini gösterir
 $ du -s * | sort -nr # sıralanmış şekilde dosyaların/klasörlerin kullanımlarını listeler
 ```


## İşlem Yönetimi 

```bash 
 $ who # sisteme kimin girdiğini gösterir
 $ w # sistemde kimlerin olduğunu gösterir
 $ ps # arka planda çalışan işlemler hakkındaki bilgileri listeler.
 $ ps -e # sistemdeki bütün işlemleri listeler
 $ ps aux | grep <kullanıcı_adı> # kullanıcıya ait çalıştırılan işlemleri listeler
 $ top # CPU ve RAM değerlerinin kullanım bilgilerini gösterir
 $ mtop # birden fazla CPU için top komutunun yaptığını yapar
 $ Ctrl z <enter> bg or fg <enter> # çalışan işlemleri durdurur, arka plana atar (bg) veya ön plana getirir (fg)
 $ Ctrl c # yeni başlamış olan işlemi durdurur
 $ kill <işlem_no> # belirlenen işlemi sonlandırır, işlem ID sine göre belirlenir
 $ renice -n <önemlilik_değeri> # işlemin önemlilik değerini değiştirmenizi sağlar 
```

## Text dosyalarını okumak


 ```bash 
 $ less <dosya_ismi> # belirtilen dosyayı terminal üzerinden okumanızı sağlar G :dosyanın sonuna gider, g : dosyanın başına gider. 
 $ more <dosya_ismi> # dosya içeriğini gösterir çıkmak için q ya basılması gereklidir.
 $ cat <dosya_ismi> #dosya içeriğini terminale yazdırır
 ```

## Metin Düzenleyicileri

#### VI ve VIM
Terminal tabanlı güçlü metin düzenleyicidir. Vi genelde linux tabanlı bütün sistemlerde mevcuttur, vim, vi nin gelişmişidir. 

#### EMACS
Grafik tabanlı metin düzenleyicidir. Bu düzenleyiciye ait olan klavye düzeni hakkında bilginiz olmalıdır. Bütün linux ve unix tabanlı sistemlerde mevcuttur. 

#### XEMACS
EMACS in çok daha gelişmişidir, yazım hataları, web ve metini iyi bir şekilde kontrol etmek mümkündür fakat normalde yüklenmiş olmaz. 

#### PICO
Terminal tabanlı basit metin düzenleyicidir, buna ait klavye düzeni bilinmelidir. 

## VIM Temelleri

#### Temeller 

 ```bash 
 $ vim dosya_ismi # dosya_ismin de dosya oluşturur veya yazma modunda açar
 $ i # vim 'in içerisine girdikten sonra i açık olan dosyaya bir şeyler yazmanıza olanak sağlar.
 $ ESC # dosya düzenleme modundan çıkılır
 $ : # vim içerisinde kullanacağınız komutlar : ile başlar
 $ :w # vim içerisinde :w yaptığınızda yazdığınızı kaydeder.
 $ :q # bu komut vim den çıkmanızı sağlar
 $ :q! # hiç birşeyi kaydetmeden çıkmanızı sağlar.
 $ :wq # kaydederek çıkar
 $ R # vim içerisinde özellik değiştirmenizi sağlar
 $ r # imlecin bulunduğu karakteri değiştirmenizi sağlar
 $ q: # vim içerisinde yazdığınız komutların kaydını gösterir
 $ :w yeni_dosyaadı # yeni dosyaya kaydeder.
 $ :#,#w yeni_dosyaadı# belirlenen (#,#) aralıktaki metini yeni dosyaya kaydeder.
 $ :# belirlenen (#) satıra gitmenizi sağlar
 ```


#### Yardım

 ```bash 
 $ vimtutor # vim içerisinde nasıl çalıştığına dair bilgileri içeren tur başlatılır.
 $ :help # vim içerisinde yardım açar, çıkmak için q komutu kullanılır.
 $ :help <konu> # belirlenen konu hakkında yardım açar
 $ :help <konu> CTRL-D # belirlenen konunun geçtiği bütün yardım dökümanını listeler.
 $ :<yukarı-asagı tusları> # daha önceki yaptığınız komutlar arasında gezmenizi sağlar.
 ```


#### Dosya içerisinde gezme (vim içerisinde)

 ```bash 
  $ $ # bulunduğunuz satırın en sonuna gider
  $ A # bulunduğunuz satırın en sonuna yazma modunu açarak gider
  $ 0 (sıfır) # satırın başlangıcına gider
  $ CTRL-g # imlecin nerede olduğu ve o satır hakkında bilgi verir
  $ SHIFT-G # imleci dosyanın en sonuna getirir
 ```

#### Görüntü (vim içerisinde)

 ```bash 
  
  WRAPPING AND LINE NUMBERS

  $ :set nowrap # kelimelerin kaymamalarını sağlar
  $ :set number # satır numaralarını gösterir
 ```

## Arşivleme ve Sıkıştırma 

 ```bash 
 $ tar -cvf dosya_ismi.tar klasör/ # verilen klasör için arşiv oluşturur
 $ tar -czvf dosya_ismi.tgz klasör/ # verilen klasör için arşivlenmiş ve sıkıştırılmış dosya oluşturur. 
 ```

#### Arşivleri görüntüleme 

 ```bash 
 $ tar -tvf dosya_ismi.tar
 $ tar -tzvf dosya_ismi.tgz 
 ```

#### Çıkartma

 ```bash 
 $ tar -xvf dosya_ismi.tar
 $ tar -xzvf dosya_ismi.tgz
 $ gunzip dosya_ismi.tar.gz 
 $ tar zxf blast.linux.tar.Zs
 ```

## Basit yükleme işlemleri 

#### RPM yüklemeleri 

 ```bash 
 $ rpm -i uygulama_ismi.rpm
 $ rpm --query <paket_ismi> ## RPM versiyonunu kontrol etme için
 ```
 
#### Debian paketlerinin yüklenmesi


 ```bash 
 $ apt-cache search nmap # nmap adındaki uygulamayı debian deposundan arama yapar
 $ apt-cache show nmap # nmap hakkında tanımlamayı(bilgi) gösterir
 $ apt-get install nmap # nmap 'i sisteme kurar.
 $ apt-get update # sistemdeki uygulamaları ve servisleri günceller
 $ apt-get upgrade -u # uygulamaların yeni versiyonu var ise yükseltme yapar
 $ dpkg -i dosya.deb # indirilen debian dosyasının yüklenmesini sağlar
 $ aptitude # apt-get ile aynı işlemi görür
 $ aptitude search vim # vim programını debian deposunda arar 
 ```

## Cihazlar

#### Takma /Çıkarma usb/floppy/cdrom


 ```bash 
 $ mount /media/usb
 $ umount /media/usb
 $ mount /media/cdrom
 $ eject /media/cdrom
 $ mount /media/floppy
 ```

## Çevresel Değişkenler 

 ```bash 
 $ xhost user@host # kullanıcı için çalıştırma izini ekler
 $ echo DISPLAY # ekranın ayarlarını gösterir
 $ export (setenv) DISPLAY=<lokal_IP>:0 # görüntü değişkeninin değerini değiştirir
 $ unsetenv DISPLAY # görüntü değişkenini siler
 $ printenv # kullanılan çevresel değişkenleri listeler
 $ $PATH # terminal üzerinde programların çalışmasını sağlayan yolları gösterir.
 ```

