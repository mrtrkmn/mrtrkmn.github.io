---
title: "Nedir bu Claude turşusu?"
date: 2026-08-18T00:00:00+00:00
tags: ["claude", "claude-code", "yapay zeka", "llm", "token", "context-window"]
author: "mrturkmen"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Claude Code'da token'ların neden hızlı tükendiği, context window ve uzun oturumları yönetme rehberi."
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: ""
    alt: "Claude Code ve context window"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Yapay zeka hayatımıza girdi gireli teknik konularda yazı yazmak, bir şeyleri anlatmak veya çizmek epey kolaylaştı. Fakat bu kadar kolaylaşması öğrenme eğrisini de düzleştirdi. Araştırma olmadan tüm bilgilerin önümüze rahat şekilde gelmesi de o bilgilerin değerini ister istemez düşürebiliyor. 

Neyse, uzun lafın kısası: bunlar araçlar ve kullanılması şart araçlar, fakat burada önemli olan işinizde kullanmayı öğrenmek en mantıklı seçenek. Zaten öğrenmediğinizde o veya bu şekilde öğrenenler sizin yerinizi er ya da geç alacaktır.

## Token sorunu

Claude kullananlardansanız belki farketmişsinizdir, son aylarda hem yavaş hem de eskisine göre çok daha fazla token kullanmakta. Hadi yavaşlamasını geçtim, bir şey sorduğumuzda, bu basit bir soru olsa bile, "haftalık limitinin %2'sini kullandın" diyebiliyor. Yani hafta dolmadan, 3 gün içinde %100'e dayanıp limitlerin resetlenmesini bekliyoruz.

Peki bu token nereye gidiyor? İnsanın ilk aklına gelen cevap şu: demek ki her soruda bütün projeyi okuyor. 

Fakat **Claude Code her soruda projenizin tamamını okumuyor.** Öyle bir şey yapsa 50 dosyalık bir repoda ilk soruda patlardı. Onun yerine ajan gibi davranıyor, grep atıyor, dosya listeliyor, sadece gerekli gördüğü dosyayı açıyor. Anthropic bunun adına "just-in-time retrieval" diyor: veriyi baştan yüklemek yerine çalışma anında çekmek.

Sorun okumanın kendisinde değil, **birikmesinde**. Her açtığı dosya, her grep sonucu, her tool çıktısı konuşmanın içinde kalıyor. 20. turda, ilk turda okuduğu o 800 satırlık dosya hâlâ orada duruyor ve her yeni istekte tekrar faturaya giriyor. Token sayacının sürekli artmasının sebebi tek bir dev okuma değil, biriken yüzlerce küçük okuma. Bu birikimin nerede durduğunu anlamak için önce context window'a bakalım.

## Context window tam olarak ne?

Context window, modelin bir cevap üretirken bakabildiği metnin tamamı, giriş de çıkış da dahil. Modelin eğitildiği o devasa veri setiyle karıştırmamak lazım; context window daha çok modelin çalışma belleği gibi.

[Resmi dökümantasyondaki](https://platform.claude.com/docs/en/build-with-claude/context-windows) güncel tablo şöyle:

| Model | Context Window | Max Output |
|-------|---------------|------------|
| Claude Fable 5, Mythos 5 | 1M token | 128K token |
| Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6 | 1M token | 128K token |
| Claude Sonnet 5, Sonnet 4.6 | 1M token | 128K token |
| Claude Opus 4.5, Sonnet 4.5, Haiku 4.5 | 200K token | 64K token |

Buradaki iki detay çoğu blog yazısında yanlış geçiyor. Birincisi, 1M olan modellerde **1M varsayılan**, beta header istemiyor, ayrı bir tier gerektirmiyor. İkincisi, uzun context için ekstra ücret yok; dökümantasyon bunu açıkça yazıyor, 900 bin token'lık bir istek 9 bin token'lık bir istekle aynı birim fiyattan faturalanıyor. (Bir dönem 200K üstü için premium fiyat vardı, o yüzden internette hâlâ o bilgi dolaşıyor.)

Bir de şu "1M token yaklaşık 750 bin kelime, yani 8-10 roman" hesabı var. Kabaca doğru ama İngilizce için. Türkçe'de aynı metin daha fazla token'a bölünüyor, çünkü tokenizer'lar ağırlıklı olarak İngilizce'ye göre optimize edilmiş. Yani Türkçe bir dökümanı context'e koyduğunuzda cebinizden İngilizce muadilinden fazlası çıkıyor.

Bununla ilgili bilmediğim ve öğrenince şaşırdığım bir şey daha: **Claude 4.7 ve sonrası modeller yeni bir tokenizer kullanıyor ve aynı metin için yaklaşık %30 daha fazla token üretiyor.** Sonnet 4.6 ve öncesi eski tokenizer'da. Yani "şu dosya 5 bin token" diye ezberlediğiniz sayı model değiştirdiğinizde geçersiz oluyor. Zaten pencere büyük olsa bile, içini gelişigüzel doldurmak her zaman iyi sonuç vermiyor.

## Context rot: büyük context her zaman iyi değil

İşte çoğu kişinin atladığı kısım. Anthropic bunu kendi dökümantasyonunda kabul ediyor ve adını da koyuyor:

> "A larger context window allows the model to handle more complex and lengthy prompts, but more context isn't automatically better. As token count grows, accuracy and recall degrade, a phenomenon known as *context rot*."

Yani 1M'lik pencereniz var diye 900K yükleyip "şimdi şu dosyadaki bug'ı bul" demek pratikte iyi bir fikir değil. Model o dosyayı atlayabilir veya yanlış hatırlayabilir. Anthropic bunu sert bir duvar olarak değil, "performans gradyanı" olarak tarif ediyor, yani belli bir eşikte çökmüyor, dolduruşa devam ettikçe azar azar bozuluyor.

Mühendislik blog'undaki [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) yazısı bunun sebebini de açıklıyor: modellerin bir "attention budget"i var ve eklediğiniz her token bu bütçeden yiyor. Transformer'da n token için n² ilişki hesaplanıyor, dolayısıyla dikkat seyreliyor. Yazının özeti tek cümle:

> "finding the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome"

Bu bence olayın esas önemli kısmı. Token tasarrufu sadece para meselesi değil, **modelin doğruluğunu korumak** meselesi. İkisi aynı yöne çekiyor, o yüzden context'i küçük tutmak iki taraflı kazanç sağlıyor. Peki bu alanı tam olarak hangi parçalar dolduruyor?

## Context window'a neler dahil?

Request'te olan her şey, buna aşağıdakiler dahil, resmi dökümasyona göre: 

- System prompt
- `messages` içindeki her mesaj (tool result'lar, image'lar, döküman'lar dahil)
- Tool definition'lar (`tools` array'i)
- Claude'un o tur ürettiği çıktı — thinking dahil

Yani sadece "sorduğunuz soru" değil, **kullandığınız araçların tanımları da yer kaplıyor** `tools` parametresini kullandığınız anda API ayrıca otomatik bir tool-use system prompt'u ekliyor.Opus 5'te 286 token, Opus 4.7'de 675 token, `tool_choice` ayarınıza göre token kullanımı değişiyor. Üç dört MCP server bağladıysanız buranın 50 bin token'ı geçmesi çok normal, daha siz "merhaba" demeden. 

Prompt caching kullanıyorsanız da şunu bilmek gerek: **cache'lenmiş token'lar hâlâ context window'da yer kaplar.** Caching o token'lara *ne kadar ödediğinizi* değiştiriyor, sayılıp sayılmadıklarını değiştirmiyor. 200K cache'lediniz diye pencerenizde 200K ek yer açılmaz. Bazı modeller bu toplamın ne kadarına ulaştığını kendileri de takip edebiliyor.

## Model kendini takip ediyor (bazıları)

Güzel bir özellik var, adı **context awareness**. Model kalan token bütçesini bilerek hareket ediyor, API her request'in system prompt'una şunu enjekte ediyor:

```xml
<budget:token_budget>200000</budget:token_budget>
```

Her tool call'dan sonra da güncelliyor:

```xml
<system_warning>Token usage: 35000/200000; 165000 remaining</system_warning>
```

Bunlar sizin tarafınızdan gönderilmiyor Claude API kendisi ekleme yaparak call yapıyor. Uzun agentic işlerde çok kritik, çünkü model "bende ne kadar yer kaldı" diye kestirmek yerine kesin bilgiyle hareket ediyor, işi ona göre bölüyor.

Ama bu özellik her modelde bulunmuyor. Dökümantasyona göre **Sonnet 5, Sonnet 4.6, Sonnet 4.5 ve Haiku 4.5**'te var. **Opus 4.7 ve sonrası Opus'lar, Fable 5 ve Mythos 5** bu tag'leri almıyor; onlar için "task budgets" denen ayrı bir beta özellik var (`task-budgets-2026-03-13` header'ı, minimum 20 bin token, ve önemli bir nüans: bu bir *soft hint*, `max_tokens` gibi katı bir tavan değil). Bu bilgi modeli daha temkinli yapabilir, ama pencereyi büyütmez. Sınır gerçekten aşıldığında API'nin nasıl tepki verdiği ise ayrı bir konu.


## Pencere (window) dolduğunda ne oluyor?

"Window dolunca 400 alırsınız" demek tam doğru değil. Burada iki farklı sınır var ve sonuçları farklı:

### 1. Input tek başına pencereye sığmıyorsa

Diyelim modelin context window'ı 200K token. Konuşma geçmişi, system prompt, tool sonuçları, açılan dosyalar ve yeni mesajınız zaten toplam 205K token ediyor.

Bu durumda modelin cevap yazmaya başlayacağı yer bile yoktur. API isteği doğrudan reddeder:

```text
400 invalid_request_error
prompt is too long
```

Bu davranış bütün modellerde aynı, burada hiç cevap yok.

### 2. Input sığıyor ama cevap için kalan yer yetmiyorsa

Şimdi yine 200K'lık bir pencere düşünelim. Input 190K token olsun, teknik olarak istek sığıyor; modelin cevap üretmesi için yaklaşık 10K yer kalmış durumda.

Ama `max_tokens: 20000` verdiğinizde bu, modele gerçekten 20K boş alan açıldığı anlamına gelmiyor. `max_tokens` yalnızca "bu turda en fazla bu kadar output üretebilirsin" sınırı. Gerçek sınır hâlâ pencerenin kalan kısmı:

```text
Context window       200K
──────────────────────────────
Mevcut input         190K
Gerçekte kalan alan   10K
max_tokens isteği     20K
```

Claude 4.5 ve daha yeni modellerde API bu isteği kabul ediyor ve model 10K'lık gerçek boşluğa ulaştığında üretim duruyor ve cevapta şunu görüyorsunuz:

```text
stop_reason: "model_context_window_exceeded"
```

Yani bu bir `400` değil. Cevap teknik olarak başarılı dönmüş olabilir, fakat context sınırına geldiği için yarıda kesilmiştir. Kodunuz yalnızca hata/exception bekliyorsa, elinize sessizce eksik bir sonuç geçebilir.

Eski modellerde davranış daha katı: input + `max_tokens` toplamı pencereyi aşıyorsa istek validation hatasıyla reddedilir. İsterseniz eski modellerde de yeni davranışı `model-context-window-exceeded-2025-08-26` beta header'ı ile açabiliyorsunuz.

Claude Code tarafında bu ayrım özellikle uzun oturumlarda önemli. Code; konuşma geçmişini, okuduğu dosyaları, grep/tool çıktılarını ve talimatları aynı pencereye koyuyor. Input tek başına sığmıyorsa yeni bilgi ekleyemez; sığıp da kalan alan azaldıysa ise bir agent adımı, açıklama ya da kod üretimi normal görünüp yarıda kalabilir. Bu yüzden "limitim bitmedi, neden iş yarım kaldı?" sorusunun cevabı çoğu zaman haftalık kullanım limiti değil, o anki context'te kalan gerçek alandır. Uzun işlerde compaction, gereksiz tool çıktısını temizlemek veya temiz bir oturum açmak bu yüzden işe yarar. Üstelik bu alanı yalnızca dosyalar ve tool çıktıları tüketmiyor; extended thinking de aynı bütçeden yiyor.

## Thinking token'ları

Extended thinking kullanıyorsanız thinking token'ları da context window içerisinde sayılıyor, ama modele göre davranış değişiyor:

- **Opus 4.5 ve sonrası, Sonnet 4.6 ve sonrası, Fable 5, Mythos 5:** önceki turların thinking block'ları context'te **kalıyor**. Kaldıkları için de sonraki isteklerde **input token olarak faturalanıyorlar**.
- **Daha eski Opus/Sonnet'ler ve tüm Haiku'lar:** sadece son turun thinking'i kalıyor, eskilerini geri gönderseniz bile API otomatik ayıklıyor, dolayısıyla para da götürmüyor.

Birinci gruptaysanız bu ciddi bir kalem, çünkü thinking binlerce token olabiliyor ve her turda birikiyor. İsterseniz `clear_thinking_20251015` parametresi ile bu davranışı yoksayıp "son N turun thinking'ini tut" diyebiliyorsunuz.

Not: dökümantasyonun token-counting sayfası bu konuda hâlâ eski davranışı ("önceki turların thinking'i sayılmaz") genel kural gibi yazıyor ve kendi örneğiyle çelişiyor.Thinking ve context-windows sayfalarını esas aldığımızda yukarıda anlattığım senaryo geçerli oluyor. 



---

Özet: token'ınız tek bir hamlede bitmiyor, biriken tool çıktıları, tool tanımları ve thinking'le bitiyor. Ve context'i küçük tutmanın karşılığı sadece daha az para değil, daha isabetli cevaplar olarak size dönüyor.
