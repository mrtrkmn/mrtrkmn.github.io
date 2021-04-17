---
title: "vim "
date: 2019-01-11T09:04:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["linux", "terminal", "komut","vim", "nftp","wget", "scp", "html"]
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
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---

## Özet

Terminal üzerinden kullanılan en meşhur yazı düzenleme programı VIM hakkında komutlar ve bazı kullanışlı linux komutları



## Script nasıl yazılır.

* Dosya oluşturulur ve dosyanın başına terminalin yolu eklenir `#!/bin/bash`
*  terminal komutlarını bu yolun altına yazınız.
* daha sonra dosyanın iznini ayarlayınız. chmod +x dosya_ismi
* terminal scriptini şu şekilde çalıştırılabilirsiniz. ./dosya_ismi

* örnek bash scripti

```bash 

!/bin/bash

echo "Adınzı giriniz: "
read isim
echo "Sifrenizi giriniz"
read sifre

if [[ ( $isim == "admin" && $sifre == "random" ) ]]; then
echo "Başarılı"
else
echo "Başarısız"
fi
```

## VIM


#### Birden fazla dosya üzerinde çalışmak 

```bash
$ vim *.txt # birden fazla txt ile dosyasını aynı anda açmanızı sağlar (eğer birden fazla dosya mevcut ise)
$ :wall veya :qall # bütün açık dosyalardan yaz veya çık komutudur.
$ vim -o *.txt # birden fazle txt dosyasını açar ve yatay düzlemde gösterir, dikey düzlem için -O parametresi kullanılara
$ :args *.txt # txt ile biten bütün dosyaları argument listesine aktarır.
$ :all # bütün dosyaları yatay düzlemde ayırır
$ CTRL-w # birden fazla pencere arasında gezmenizi sağlar
$ :split # aynı dosyayı iki farklı pencerede gösterir.
$ :split <acılacak_dosya> # dosyayı yeni bir pencerede açar
$ :vsplit # brden fazla pencereyi dikey komunda ayırır, tablolar için çok kullanışlıdır. ":set scrollbind " komutu ile açık dosyalar da aynı anda yukarı aşağı yapabilirsiniz. 
$ :close # bulunduğunuz pencereyi kapatır
$ :only # bulunduğunuz pencere hariç diğerlerinin tamamını kapatır. 
```   

#### Hece Kontrolü & Sözlük 

```bash
$ aspell -c <dosya> # verilen dosyada heceleri kontrol eder, terminal komutudur
$ aspell -l <dosya> # terminal komutu
$ :! dict <cumle> # cümlenin anlamını kontrol etmenizi sağlar
$ :! wn 'cumle' -over # cümlenin eş anlamlılarını gösterir
```   

  
#### Dosyayı yazdırma 
```bash
 $ :ha # bütün dosyayı yazdırır
 $ :#,#ha # (#,#) ile belirtilen alandaki metini yazdırır
```
#### Birleştime / Ekleme Komutu 
```bash
 $ :r <dosya_ismi> # açık olan dosya içerisine,
                   # aynı dizinde olan başka bir dosyayı eklemek için bu komut kullanılabilir,
                   # imlecin hizasından sonra ekleme yapar
```
#### Geri Alma / Yeniden Alma 
 ```bash
 $ u # en son yaptığınız değişikliği geri alır
 $ U # yaptığınız bütün değişiklikleri geri alır
 $ CTRL-R # geri alınmış bir kısmı yeniden getirmenizi sağlar.
 ```

#### Kopyalama & Yapıştırma
 ```bash
 $ yy # imlecin bulunduğu satırı kopyalar, 2 satır kopyalamak için 2yy kullanılabilir.
 $ p # kesilen/kopyalanan içeriği imleçten başlayacak şekilde yapıştırır 
 ```

#### Silme/Kesme (NORMAL modda uygulanır. Yani Vim komut satırında değil. EXE modunda değil. )
 ```bash
 $ x # imlecin üzerinde bulunduğu karakteri siler.
 $ dw # imlecin bulunduğu kelimeyi sonuna kadar siler (Boşluklar dahil )
 $ de # imlecin bulunduğu kelimeyi sonuna kadar siler (Boşluklar hariç )
 $ cw # kelimenin geriye kalan kısmını siler ve sizi ekleme moduna alır, ekleme modundan ESC ile çıkabilirsiniz.
 $ c$ #  bulunduğu satırı tamamen siler ve sizi ekleme moduna alır ekleme modundan ESC ile çıkabilirsiniz. 
 $ d$ # imlecten itibaren satırı siler e
 $ dd # satırı tamamen siler, imlecin nerede olduğunun önemi yoktur
 $ 2dd # ileriki 2 satırı siler, benzer sekilde 3dd : uc satır siler, 4dd: dort satır siler, (imlecten bagımsız)
    Koyma  

 $ p # kesilen/kopyalanan içeriği imleçten başlayacak şekilde yapıştırır 
 ```

#### Dosya içerisinde arama (Vim) (bu kısımda genelde düzenli ifadeler kullanılır )
 ```bash
 $ /aramak_istediğiniz_düzen # yazdığınız ifadeyi açık olan belge içerisinde arar ve hepsini işaretler
 $ ?aramak_istediğiniz_düzen # yazdığınız ifadeyi açık olan belge içerisinde arar ama işaretlemez, n ile ileriki kelimeyi görebilirsiniz. 
 $ :set ic # kelimelerin büyük/küçük harf ayrımını ortadan kaldırır
 $ :set hls # aranan ve bulunan kelimeleri vurgulu şekilde gösterir.
```

#### Düzenli ifadeler ile metin yönetimi 
 ```bash
 $ :s/harf1/harf2/ # harf1, harf2 ile değiştirilir fakat sadece ilk karşılaşmada yapılır
 $ :s/harf1/harf2/g # bütün dosya içerisindeki harf1, harf2 ile değiştirilir.
 $ :s/harf1/harf2/gc # yukarıdaki işlemin aynısını onay alarak yapmak için "c" eklenir
 $ :#,#s/harf1/harf2/g #  (#,#) arasındaki satırlarda bulunan harf1, harf2 ile değiştirilir.
 $ :%s/harf1/harf2/g # tüm dosyadaki harf1 ifadesi harf2 ile değiştirilir.
 $ :%s/\(harf1\)\(.*\)/\1/g # harf1 sonrakisindeki bütün satırları siler.
 $ :%s/\(SL\dm\d\d\d\d\d\.\d\)\(.*\)/\1\t\2/g #  SL1m12345.1 ve tanımı arasına TAB boşluğu ekler 
 $ :%s/\n/ifade/g #Satır verilen ifade ile değiştirilir.
 $ :%s/\(^SL\dm\d\d\d\d\d.\d\t.\{-}\t.\{-}\t.\{-}\t.\{-}\t\).\{-}\t/\1/g  #  5 ve 6.ncı TAB taki (5. Kolondaki), içeriği "{-}" ile degiştirir. 
 $ :#,#s/\( \{-} \|\.\|\n\)/\1/g # (#,#) verilen aralıkta ne kadar cümle olduğunu hesaplar
 $ :%s/\(E\{6,\}\)/<font color="green">\1<\/font>/g # 6 dan fazla E geçen kısımları, HTML renkleri ile vurgular.
 $ :%s/\([A-Z]\)/\l\1/g # Büyük harfleri, küçük harfler ile değiştirir, '%s/\([A-Z]\)/\u\1/g' , bu ise küçük harfleri büyük harfler ile değiştirir.
 $ :g/ifade/ s/\([A-Z]\)/\l\1/g | copy $ # ifade yeni oluşturulan ifade ile değiştirilir eşdeğer olanlar copy $ ile yazdırılır. 
 ```
#### HTML Düzenleme 
 ```bash
 -metini HTML formatına cevirme
 $ :runtime! syntax/2html.vim # vim içerisinde bu komutu çalıştırınız.
 ```
#### Vim içerisinden terminal komutu çalıştırma 
 ```bash
 $ :!<terminal_komutu> <ENTER> # terminal komutunu vim içerisinden çalıştırır
 $ :sh terminal ile vim arasında gezmenizi sağlar
 ```
#### Tablo düzenleyicisi olarak Vim' i kullanmak 
 ```bash 
 $ v # karakterleri seçmek için görsel mod başlatılır.
 $ V # satırları seçmek için görsel mod başlatılır.
 $ CTRL-V # blok görsel seçim yapmanızı sağlar.
 $ :set scrollbind # aynı ayna ayrılan iki ayrı dosyada gezinti yapmanızı sağlar. 
  ```

####  Vim ayarlarını değiştirmek 
 
   __- .vimrc dosyası içerisindeki parametreler isteğinize göre değiştirilebilir.__


## Kullanışlı terminal komutları
```bash
 $ cat <dosya1> <dosya2> > <sonuc> # dosya1 ve dosya2 yi sonuc dosyasina kopyalar ve sonuc dosyasini olusturur.
 $ paste <dosya1> <dosya2> > <p_sonuc> # iki farklı kaynaktan gelen girdiyi, aralarında TAB boşluğu olacak şekilde aynı dosya (p_sonuc) içerisine yapıştırır.
 $ cmp <dosya1> <dosya2> # iki dosyanın aynı olup olmadıgını size bildirir.
 $ diff <dosya1> <dosya2> # iki dosya arasındaki farklılıkları gösterir
 $ head -<numara> <dosya> # verdiğiniz numara kadar ilk X satırı yazdırır.
 $ tail -<numara> <dosya> # verdiğiniz numara kadar son X satırı yazdırır. 
 $ split -l <numara> <dosya> # dosyanın satırılarını ayırır.
 $ csplit -f out dosya_ismi "%^>%" "/^>/" "{*}" # dosya_ismini > den itibaren birçok farklı küçük dosyalar oluşturur.
 $ sort <dosya_ismi> # dosya içerisindekileri sıralar -b argument kullanılırsa boşlukları yok sayar.
 $ sort -k 2,2 -k 3,3n girdi_dosyası > cıktı # -k argument i kolon için, -n sayısal olarak sıralar ve tablo şeklinde kaydeder. 
 $ sort girdi_dosyası | uniq > cıktı # uniq komutu aynı olan verileri dahil etmez.
 $ join -1 1 -2 1 <tablo1> <tablo2> #  tablo1 ve tablo2 yi birleştirir, -1 dosya1, 1:kolon1; -2dosya2, col2. 
 $ sort tablo1 > tablo1a; sort tablo2 > tablo2a; join -a 1 -t "`echo -e '\t'`" tablo1a tablo2a > tablo3 # '-a <tablo>' : verilen tablonun bütün kayıtlarını yazdırır.  Normalde yazdırma işlemi iki tabloda ortak olan kısımları yazdırır.  '-t "`echo -e '\t'`" ->' 
 : TAB boşluğu kullanarak tabloları çıktı dosyasına yazdırır. 
 $ cat tablom | cut -d , -f1-3 # cut komutu : tablonun belirlenen kısımları alır, -d alanların nasıl ayrılacağını belirtilsiniz. -d : burada , olarak belirlenmiştir, normalde TAB boşluk, -f tablonun kolonlarını belirtir, kolon 1 den 3 e. 
```


#### Kullanışlı tek satır komutlar 

```bash
 $ for i in *.input; do mv $i ${i/isim\.eski/isim\.yeni}; done # isim.eski adındaki dosyanın ismini, isim.yeni olarak değiştirir. Komutu test etmek için,  do mv komutu önüne "echo" konulabilir.  
 $ for i in *.girdi; do ./uygulama $i; done #  bir çok dosya için verilen uygulamayı çalıştırır. 
 $ for i in *.girdi; do komut -d /veri/../veri_tabanı -i $i > $i.out; done #  komut for döngüsü içerisinde *.girdi üzerinde çalışır ve *.out dosyası oluşturur. 
 $ for i girdi *.pep; do hedef -db /usr/../veri_tabanı -seed $i -out $i; done # hedef in üzerinde for döngüsü çalıştırılır ve çıktı dosyası yazdırılır.
 $ for j girdi 0 1 2 3 4 5 6 7 8 9; do grep -iH <ifade> *$j.seq; done # 
 verilen ifadeyi girdi > 10.000 dosyaya kadar arar ve ne kadar o ifade geçtiğini yazdırır.
 $ for i in *.pep; do echo -e "$i\n\n17\n33\n\n\n" | ./program $i > $i.out; done # etkileşimli programı çalıştırır ve girdi/çıktı sorar. 
```


#### Basit Perl Komutları 

```bash
 $ perl -p -i -w -e 's/ifade1/ifade2/g' girdi_dosyası #  girdi dosyası içerisindekileri verilen ifadelere göre değişimini yapar. '-p'  bu komut yedek bir dosya oluşturur 
 $ perl -ne 'print if (/ifade1/ ? ($c=1) : (--$c > 0)) ; print if (/ifade2/ ? ($d = 1) : (--$d > 0))' girdi_dosyası > cıktı_dosyası # ifade1 ve ifade2 içeren satırları ayrıştırır (parse eder.)  
```


#### WGET (terminal üzerinden linki verilen dosya indirimini gerçekleştirir.)

```bash
 $ wget ftp://ftp.itu.edu.tr.... # verilen linkteki dosya wget komutunun çalıştırıldığı dizine iner. 
```


#### SCP (İki makine arasında güvenli kopyalama işlemi sağlar. )

```bash
 Genel Kullanım.
 $ scp kopyalanacak_dosya kopyalanacak_yer #
 Örnekler 
 Sunucudan dosya kopyalamak için (bilgisayarınızın terminalinden)
 $ scp kullanıcı@sunucu_ip:dosya_adı . # '.'  en sona nokta koyulması, sunucu üzerindeki kopyalanacak dosyayı bulunduğunuz yere kopyalamasını sağlar. 
 Bilgisayarınızdan sunucuya kopyalama yapmak için. (Bilgisayar terminalinden)
 $ scp resim.jpg kullanıcı@sunucu_ip:~/belgeler/resimler/
 Sunucu üzerinde bulunan klasörü bilgisayarımıza kopyalamak için. (Bilgisayar terminalinden)
 $ scp -r kullanıcı@sunucu_ip:dizin/ ~/Masaustu
 Bilgisayar üzerinde bulunan klasörü sunucuya kopyalamak için. (Bilgisayar terminalinden)
 $ scp -r klasör/ kullanıcı@sunucu_ip:dizin/
  ```

#### NFTP : (Dosya transfer işlemlerinizi kolay şekilde terminal üzerinden yapmanızı sağlar )

```bash
 $ open ncftp
 $ ncftp> open sunucu_url  # sunucuya baglantı sağlanıyor..
 $ ncftp> cd /root/Masaustu. # masaustune gecildi
 $ ncftp> get resimler.gz    # masaustunde bulunan resimler.gz indirildi.
 $ ncftp> bye    # gule gule mesajı alındı
```

