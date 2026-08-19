---
title: "Headroom'u kurdum, denedim, kaldırdım"
date: 2026-08-19T00:00:00+00:00
tags: ["headroom", "claude", "claude-code", "yapay zeka", "llm", "token", "context-compression", "mcp"]
author: "mrturkmen"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Headroom gerçekten token tasarrufu sağlıyor mu? Kurulum, mimari, kendi benchmark'ları ile bağımsız ölçümler arasındaki fark ve prompt caching tuzağı."
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: ""
    alt: "Headroom context compression mimarisi"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

*(Serinin ikinci yazısı. [İlk yazı](../nedir-bu-claude-tursusu/) token'ın nereye gittiğiyle ilgiliydi.)*

Token derdine çözüm arayan bir sürü repo çıktı son aylarda. En dikkat çekeni **Headroom** ([headroomlabs-ai/headroom](https://github.com/headroomlabs-ai/headroom)). Ocak 2026'da açılmış, yazdığım sırada 66 bin civarı yıldızı var, Apache-2.0, Python. Trendshift'in kaydına göre 2 Haziran 2026'da GitHub Trending'de #1 olmuş. (Repo'nun kendisi "biz #1 olduk" diye bir şey yazmıyor, sadece badge koymuş — o yüzden bu bilgiyi Trendshift'e dayandırıyorum.)

Ben kurup denedim ve beklediğim tasarrufu göremedim. İlk başta "ya ben kurulumu beceremedim ya Claude API'de bir şey var" diye düşündüm. Sonra oturup repo'yu ve issue'ları düzgünce okuyunca anladım ki aslında **tam da olması gerektiği gibi olmuş**. Şimdi onu anlatacağım, çünkü bu yazının asıl faydalı kısmı orası.

## Headroom aslında ne yapıyor?

GitHub açıklaması net: *"Compress tool outputs, logs, files, and RAG chunks before they reach the LLM."* README'deki slogan da *"The context compression layer for AI agents."*

Yani model değiştirmiyor, prompt'unuzu yeniden yazmıyor. **Modele gitmeden önce araya girip içeriği sıkıştırıyor.** Dört ayrı şekilde kullanılabiliyor:

- Kütüphane olarak (`from headroom import compress`)
- Lokal HTTP proxy olarak (`headroom proxy --port 8787`)
- Ajanı sarmalayarak (`headroom wrap claude`, geri almak için `headroom unwrap`)
- MCP server olarak (`headroom_compress`, `headroom_retrieve`, `headroom_stats` tool'larıyla)

İçeride bir router var, içeriğin tipini tespit edip uygun sıkıştırıcıya yolluyor: JSON için SmartCrusher, kod için CodeCompressor, arama sonuçları için SearchCompressor, log'lar için LogCompressor, diff, HTML, tablo, config için ayrı ayrı sıkıştırıcılar. Düz metinde bunlar tutmazsa Kompress-v2-base diye küçük bir HF modeline düşüyor.

Yazının kalanında tek tek gireceğim parçaların tamamı tek resimde şöyle duruyor — istek soldan giriyor, `CacheAligner → ContentRouter → sıkıştırıcılar → CCR` hattından geçip sağdaki sağlayıcıya çıkıyor; kutuların içindeki kırmızılar da benim yaşadığım yerler:

![Headroom mimarisi: ajanlar, dört entegrasyon modu, CacheAligner → ContentRouter → sıkıştırıcılar → CCR boru hattı, lokal cache, cross-agent memory deposu ve manşet rakamlar](headroom-genel-bakis.drawio.png)


Burada bir yanlış anlaşılmayı baştan keselim, ben de bu tuzağa düşmüştüm: **bu semantik/embedding tabanlı bir sıkıştırma değil.** Bir LLM oturup metninizi özetlemiyor. Yapılan iş yapısal ve istatistiksel: fonksiyon gövdelerini atmak, yorumları atmak, uzun string değerleri kırpmak, boşlukları temizlemek, tekrar eden yapıyı maskelemek. UUID ve hash gibi şeyleri korumak için ayrı bir "entropy preservation" ayarı var (`use_entropy_preservation=True`), yoksa onları da çöp sanıp atabiliyor.

Bir de **CCR** var, "Compress-Cache-Retrieve" açılımıyla. Orijinal içerik lokalde saklanıyor ve modele `headroom_retrieve` diye bir tool veriliyor, ihtiyaç duyarsa aslını geri çekebiliyor. Repo bunu "aggressive but reversible" diye tanıtıyor. Buradaki "reversible" kelimesini dikkatli okumak lazım: sıkıştırma **kayıplı**, geri dönüş ancak TTL süresi içinde cache'te durursa mümkün. Nitekim açık bir issue (#2604) tam bunu anlatıyor — 30 dakikalık duvar saati TTL'i uzun bir multi-agent oturumundan kısa kaldığı için `headroom_retrieve` "kayıt yok" diyor. Yani geri alınabilirlik sessizce başarısız olabiliyor.

## "Memory sharing" meselesi — burada yanlış biliyordum

Ben bunu şöyle anlamıştım: birden fazla ajan kurduysan, Claude'da token'ın bitince Codex'e, orada bitince Copilot'a geçiyorsun ve aynı hafızayı paylaştıkları için hiçbir şey kaybetmiyorsun.

İkisi de yanlıştı.

Birincisi, **paylaşılan hafıza size token vermiyor.** O bir kota havuzu değil. Paylaşılan olan şey konuşmalardan çıkarılmış bilgiler (kullanıcının tercihleri, projeye dair kararlar vs.), lokalde bir SQLite + HNSW + FTS5 yığınında tutuluyor. Claude'un haftalık limiti bittiğinde Codex'e geçip devam edebilirsiniz tabii, ama bunu Headroom olmadan da yapabilirsiniz; Headroom'un kattığı şey sadece "aynı notları görürler" kısmı.

İkincisi, **Copilot bu listede yok.** README'nin ifadesi şu: *"Cross-agent memory — shared store across Claude, Codex, Gemini, Grok, auto-dedup."* Copilot ve Cursor repo'da geçiyor ama proxy/wrap entegrasyonu olarak, hafıza paylaşan taraflar olarak değil (Cursor için kurulum zaten manuel). Üstelik dökümantasyonun Memory sayfası hiç ajan ismi vermiyor, sadece "Memory works with any OpenAI-compatible client" diyor ve scope'lardan birinin adı doğrudan **Agent** — yani "o ajana özel" bir kapsam da var.

Özetle: ortak bir lokal veritabanı üzerinden mümkün, ama benim anlattığım gibi first-class bir "havuzdan devam et" özelliği değil. Yukarıdaki şemada bu kısım 5 numaralı blok: hafıza deposu boru hattının **yanında** duruyor, içinde değil — istek yolunda oturup token kesen bir bileşen değil yani.

## Rakamlar: 92% mi, 15% mi?

Ben "bunların benchmark'ı yok" diye yazmıştım. Yanlış — var, hem de repo'nun içinde. Sadece hepsi **kendi koştuğu** benchmark'lar (`python -m headroom.evals suite --tier 1` ile tekrar edilebiliyor, her biri N=100):

- GSM8K: 0.870 → 0.870 (fark yok)
- TruthfulQA: 0.530 → 0.560 (hafif artmış)
- SQuAD v2: %19 sıkıştırmada %97 doğruluk
- BFCL: %32 sıkıştırmada %97 doğruluk

Tasarruf tarafında da senaryo bazlı tablo var: kod araması 17.765 → 1.408 token (%92), SRE incident debug 65.694 → 5.118 (%92), GitHub issue triage %73, codebase exploration %47.

**İşte benim hatam buradaydı.** O %92'leri görüp genel beklenti sanmışım. Halbuki README'nin kendi manşeti şunu diyor: *"60–95% fewer tokens (for JSON data), **15-20% fewer tokens (for coding agents)**."* Ben kodlama ajanı için test ettim. Yani repo bana zaten 15-20 diyordu, ben 90 bekliyordum. Görmediğim tasarruf aslında sözü verilmemiş tasarrufmuş.

Bunu daha da netleştiren bir şey var: repo'da **#645 numaralı açık bir issue** var, üçüncü bir tarafın proxy v0.23.0'ı gerçek bir agentic iş yükünde ölçtüğü bağımsız bir benchmark. Sonuç: **istek başına ortalama %10,2 sıkıştırma**, en iyi tek vaka %25,1. Aynı issue, Headroom'un kendi yayınladığı **production fleet median'ının %4,8** olduğunu söylüyor ve gözlenen değerlerin "manşet benchmark rakamlarından çok uzak, kendi yayınladığınız fleet median'a çok yakın" olduğunu not ediyor. (O %4,8'i README'de bulamadım, o yüzden bu sayıyı issue'ya dayandırıyorum, doğrulanmış kabul etmiyorum.)

Bu arada aynı issue Headroom'a bir konuda tam not veriyor: test edilen üç üründen **tek gerçekten kablo üstünde ölçülebilir tasarruf sağlayanı** Headroom olmuş ve **sayacı sağlayıcının faturasıyla tutan tek ürün** yine o. Yani proje dürüst, sadece manşeti fazla iyimser.

PyPI'ye bakarsanız iş daha da karışıyor, çünkü orada eski bir pitch duruyor: *"Cut costs by 50-90%."* Nereden alıntı yaptığınıza göre aynı proje için üç farklı rakam söyleyebiliyorsunuz.

## Yavaşlık hissim uydurma değildi

"İnanılmaz bir yavaşlık hissettim, o yüzden kaldırdım" diye yazmıştım. Bunu doğrulayan somut şeyler var.

Yukarıdaki bağımsız ölçüm CPU üzerinde **tur başına yaklaşık +0,9 saniye** ek gecikme buluyor. README'de gecikme veya ek yük hakkında tek bir rakam yok, yani bu maliyet hiçbir yerde ilan edilmiyor.

Açık ve kapalı issue'larda "hang" başlıklı bir düzine kayıt var. Birkaçı:

- #1810 — proxy, Claude Code'da WebSearch tool call'larını askıda bırakıyor, elle iptal gerekiyor
- #810 — tiktoken indirmesi engellendiğinde her istekte sessiz 30 saniye bekleme; issue'nun kendi ifadesiyle "the proxy becomes pure latency"
- #946 — 29+ mesajlık uzun Claude Code konuşmalarında ilk aşama 30 saniyede takılıyor
- #980 (açık) — Docker bridge network'te senkron DNS asyncio loop'unu bloklayınca ilk istek donuyor
- #2820 — macOS'ta uzun ömürlü proxy'nin bellek kullanımı ~11 GB'a tırmanıyor

Bir de en can alıcısı, doğrudan tasarruf iddiasının karşısına düşen bir kayıt: **#2438** — *"Proxy defeats Anthropic prompt caching in both --mode token and --mode cache (measured 2-7x cost increase)."* Yani araya girip içeriği değiştirmek, Anthropic'in prompt cache prefix'ini bozduğu için maliyeti **7 kata kadar artırabiliyor**. Kapatılmış bir issue ama mantığı önemli: caching, prefix'in birebir aynı olmasına bakıyor. Her istekte içeriği yeniden yazan bir katman, cache hit'i kaçırmanıza sebep olursa %10'luk cache okuma fiyatı yerine tam fiyat ödersiniz. Sıkıştırmadan kazandığınız %15, cache'ten kaybettiğiniz %90'ı kurtarmaz.

Bu bence araç bazlı en önemli ders: **sıkıştırma her zaman tasarruf demek değil.**

## Doğruluk tarafı

Sıkıştırma kayıplı olduğu için beklenen bir sonuç, ama somut örnekler var:

- #3099 (açık) — sıkıştırma, shell komut sınırlarını aşarak metinleri birleştiriyor ve *"well-formed lines that encode a false fact"* üretiyor. Yani düzgün görünen ama yanlış bilgi taşıyan satırlar.
- #3098 (açık) — CCR'nin bıraktığı işaretler ajan tarafından "veri bozulmuş" gibi okunuyor; bir vakada uydurma bir "dosya bozuk" iddiası hem alt prompt'a hem compaction özetine sızmış.
- #2110 (kapalı) — cache_aligner'ın dinamik içerik dedektörü false positive verip cache'lenmiş system prompt'ları bozuyordu.

Bunlar "araç kötü" demek değil, "kayıplı sıkıştırmanın faturası bu" demek. Karar verirken bilmek lazım.

## Kurulumda takılabileceğiniz yerler

Ben kurulumu beceremediğimi düşünmüştüm; meğer README'nin kendisi bir sürü ortam tuzağını sıralıyor:

- Python 3.10+ gerekiyor, dolar bazlı tasarruf raporu için 3.13 öneriliyor
- CLI **sadece PyPI'de**. `npm install headroom-ai` yalnızca TypeScript SDK'sını veriyor, CLI vermiyor
- x86'da **AVX2 zorunlu**; Intel macOS'ta hazır ONNX Runtime yok (`ORT_STRATEGY=system`)
- Kurumsal SSL inspection varsa önce Rust kurmanız veya hazır wheel kullanmanız gerekiyor
- `[vector]` extra'sı C++ toolchain istiyor ve **`[all]` içine dahil değil**

Ve en önemlisi: **birkaç kilit özellik varsayılan olarak kapalı.** Kaynak kodu sıkıştırma opt-in. Çıktı token'ı azaltma `HEADROOM_OUTPUT_SHAPER=1` ile açılıyor. CacheAligner sadece dedektör, prompt'u hiç yeniden yazmıyor ve varsayılanda kapalı. Yani `headroom wrap claude` deyip "hadi bakalım" diyorsanız muhtemelen özelliklerin bir kısmı hiç devrede değil. #1869'da (açık) tam bunu yaşayan biri var: dashboard baştan sona sıfır gösteriyor.

Son bir uyarı: bundle'lanmış Serena MCP kaydı sorun çıkarabiliyor. #2787'yi maintainer'ın kendisi açmış ve "High" etiketi koymuş: *"one broken MCP server degrades every Claude Code session on the machine."* Yani araç, çözmeye çalıştığı problemi büyütebilecek bir yere dokunuyor.

## Peki kurmaya değer mi?

Bana kalırsa şu ayrımı yapmak lazım.

**JSON, log, arama sonucu, HTML kazıma gibi yapısal ve gürültülü veriyle çalışıyorsanız** — evet, denemeye değer. Repo'nun en güçlü olduğu yer burası ve %70-90'lık rakamlar bu senaryolardan geliyor. Mantıklı da: bir API cevabındaki tekrar eden anahtarları atmak bilgi kaybı yaratmaz.

**Ben gibi kodlama ajanı için bekliyorsanız** — beklentinizi %15-20'ye çekin, çünkü README öyle diyor. Bağımsız ölçüm %10 diyor. Buna karşılık tur başına ~1 saniye gecikme, prompt cache'i bozma riski ve kayıplı sıkıştırmanın doğruluk maliyeti var. Bu takas benim işime gelmedi, o yüzden kaldırdım.

Ölçmeden karar vermeyin ama, `headroom doctor` ve `headroom dashboard` var, `headroom perf` var. Ve şuna özellikle bakın: `usage` alanındaki `cache_read_input_tokens` düşüyor mu? Düşüyorsa sıkıştırma size para kazandırmıyor, kaybettiriyor.

Açık issue listesi de gerçekten okumaya değer: [github.com/headroomlabs-ai/headroom/issues](https://github.com/headroomlabs-ai/headroom/issues). Şu an 244 açık issue var (API 484 diyor ama onun yarısı açık PR). Proje çok hızlı hareket ediyor, yani bu yazıdaki rakamların bir kısmı siz okurken değişmiş olabilir.

---

Sıradaki ve son yazıda araç kurmadan, Anthropic'in kendi sunduğu kollarla ne kadar yol alınabileceğine bakıyorum. Prompt caching'in gerçek fiyat matematiği, MCP tool şişkinliği (ki bu bence en büyük kazanç), compaction, ve Claude Code'da günlük olarak işe yarayan şeyler.
