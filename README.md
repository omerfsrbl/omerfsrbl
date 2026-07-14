# Sunum Şablonu — Log Viewer Projesi

> 22 slayt, ~25-35 dakika (demo süresine göre esnek). PowerPoint + Web + VS Code
> üçlü format için tasarlandı. Her slaytta bir **aksiyon etiketi** var:
> **🎤 SADECE ANLAT** — hiç ekran değiştirme
> **🌐 WEB'E GEÇ** — çalışan uygulamaya geç
> **💻 KODA GEÇ** — VS Code'a geç
> Bu etiketi slaytın sağ alt köşesine (küçük, soluk renkte) koy — hem sana hem
> izleyiciye "şimdi canlıya geçiyoruz" sinyali verir, şeffaf bir sunum tekniği.

---

## Genel Tasarım Sistemi (tüm slaytlara uygulanacak kurallar)

**Tema önerisi:** Uygulamanın kendi kimliğiyle (koyu zemin + yeşil aksan) eşleşen bir PowerPoint teması kullanmak, "ürün ve sunum tutarlı, özenli" hissi verir:
- Zemin: neredeyse siyah (`#0B0C0E`)
- Başlık/metin: açık gri-beyaz (`#E7E8EA`)
- Aksan (başlık altı çizgi, ikonlar, vurgular): yeşil (`#34D399`)
- Bu renkleri PowerPoint'te birebir uygulamak zahmetliyse, herhangi bir **standart koyu kurumsal şablon** yeterli — önemli olan tutarlılık, birebir hex eşleşmesi değil.

**Tipografi:**
- Başlık: 36-40pt, kalın
- Küçük "kicker" etiketi (başlığın hemen üstünde, opsiyonel): 14-16pt, BÜYÜK HARF, harf aralığı geniş, soluk renk — uygulamandaki `TIMESTAMP`, `LEVEL` gibi kolon başlıklarının stiliyle bilinçli bir görsel bağ kurar
- Alt madde metinleri: 20-24pt, **kısa öbekler** (3-6 kelime), tam cümle değil
- Slayt başına en fazla 3-5 madde, bol satır arası boşluk

**Yerleşim:** Çoğu slaytta sol/üstte başlık + altında maddeler; sağ alt köşede aksiyon etiketi. Kod/web'e geçilen slaytlarda madde sayısını 3'ün altında tut — o slaytın asıl "içeriği" senin canlı gösterimin, slayt sadece bir çapa.

---

## BÖLÜM 0 — Açılış (Slayt 1-2)

### Slayt 1 — Başlık
🎤 **SADECE ANLAT**
- **Başlık:** Log Viewer — Yüksek Performanslı Log İzleme Aracı
- **Alt satır (16-18pt):** ASELSAN Yaz Stajı, 2026
- **En altta küçük:** adın, mentörünün adı (opsiyonel)
- **Sunum notu:** 20-30 saniye. Hiçbir şeye dokunma, sadece "20+ mikroservisten gelen logları, tek bir arayüzde, tarayıcıyı hiç zorlamadan görüntüleyen bir araç geliştirdim" gibi tek cümlelik bir açılış yap.

### Slayt 2 — Sunum Planı
🎤 **SADECE ANLAT**
- **Başlık:** Sunum Planı
- **Maddeler:** "Görev ve Kısıtlar" / "Mimari Kararlar" / "Teknik Derinlik (canlı demo)" / "Dayanıklılık Testleri" / "Özet"
- **Sunum notu:** İzleyiciye "nereye gideceğimizi" baştan söylemek, 20+ dakikalık bir sunumda çok işe yarar — dikkat dağılmasını önler. 15 saniye, hızlı geç.

---

## BÖLÜM 1 — Görev ve Kısıtlar (Slayt 3-4)

### Slayt 3 — Görev
🎤 **SADECE ANLAT**
- **Başlık:** Görev
- **Maddeler:** "20+ mikroservisten gelen loglar" / "Onlarca GB'a ulaşabilen veri" / "Tek arayüzde inceleme ve filtreleme"
- **Sunum notu:** Mentörünün sana verdiği orijinal talebi anlat — console_1/2/3 klasör yapısı, serviceLogs ayrımı. Bu slayt, sonrasında göreceğin her teknik kararın **neden** var olduğunun temelini atıyor.

### Slayt 4 — Performans Barı & Kısıtlar
🎤 **SADECE ANLAT**
- **Başlık:** Performans Barı & Kısıtlar
- **Maddeler:** "İlk yükleme: 20-30 sn kabul edilebilir" / "Sonrası: sıfır lag" / "Docker YOK (yüksek güvenlik ortamı)" / "Kod: tamamen elle (AI sadece tasarım/CSS)"
- **Sunum notu:** Bu slaytta **iki farklı kısıtı** net ayır — biri sayısal bir performans hedefi, diğeri bir ortam kısıtı (Docker). AI kullanım sınırını burada açıkça söylemek, sunumun geri kalanında "bunu ben yazdım" güvenilirliğini baştan kuruyor — saklanacak bir şey değil, aksine olgunluk göstergesi.

---

## BÖLÜM 2 — Mimari Kararlar (Slayt 5-8)

### Slayt 5 — İlk Plan: ClickHouse + Docker
🎤 **SADECE ANLAT**
- **Başlık:** İlk Plan: ClickHouse + Docker
- **Maddeler:** "Log analytics için sektör standardı" / "Columnar, çok hızlı" / "Docker container olarak planlandı"
- **Sunum notu:** Bunu bir **hikaye başlangıcı** gibi anlat, düz bir liste okur gibi değil — "araştırdım, ClickHouse'a karar verdim, kurulumu yaptım" tonunda. Bir sonraki slaytta bu planın neden çöktüğünü göstereceksin, aradaki kontrast önemli.

### Slayt 6 — Kısıt: Docker Yasak → Pivot
🎤 **SADECE ANLAT**
- **Başlık:** Docker Yasak → Mimari Pivot
- **Maddeler:** "Kurumsal güvenlik politikası" / "Container çalıştırılamıyor" / "Embedded bir alternatif gerekti"
- **Sunum notu:** Burada dramatik bir duraklama işe yarar — "planım hazırdı, ortam beni durdurdu" cümlesi, bunun bir tutorial takip etmek değil **gerçek bir mühendislik problemi** olduğunu gösteriyor. Bunu es geçme, en güçlü anlatım anlarından biri.

### Slayt 7 — Neden DuckDB
💻 **KODA GEÇ** (kısa)
- **Başlık:** Neden DuckDB — Embedded Mimari
- **Maddeler:** "Embedded: ayrı server/container yok" / "Columnar: log analytics'e doğal uyum" / "Disk'e yazan, GB ölçeğini kaldıran"
- **Sunum notu:** VS Code'a geç, `DuckDBInstance.create('logs.db')` satırını göster: "host, port, bağlantı stringi yok — çünkü bağlanacak ayrı bir server yok, kütüphane benim process'imin içine gömülü." "Embedded" kelimesini burada **net tanımla**, sonraki her şey buna dayanıyor.

### Slayt 8 — Stack Özeti
🎤 **SADECE ANLAT**
- **Başlık:** Stack Özeti
- **Maddeler:** "React + TypeScript" / "Node.js + Express" / "DuckDB (embedded)" / "react-window (virtualization)"
- **Sunum notu:** Bu bir "harita" slaytı — her parçayı isim isim say, derinlemesine girme, sonraki bölümde her birine tek tek gireceksin.

---

## BÖLÜM 3 — Teknik Derinlik (Slayt 9-16) — sunumun en uzun ve en kritik bölümü

### Slayt 9 — Ingestion Pipeline
🌐 **WEB'E GEÇ** + 💻 **KODA GEÇ**
- **Başlık:** Ingestion Pipeline
- **Maddeler:** "Klasör sürükle-bırak" / "Her dosya ayrı istek" / "DuckDB dosyayı kendisi okur"
- **Sunum notu:** Önce web'e geç, **küçük** bir klasör sürükle (mock-logs-3 gibi), progress overlay'i göster. Sonra VS Code'a geç, `read_json_auto(...)` satırını göster: "Node kendisi JSON parse etmiyor, DuckDB dosyayı diskten kendisi, streaming şekilde okuyor."

### Slayt 10 — Timestamp Normalizasyonu
💻 **KODA GEÇ**
- **Başlık:** Timestamp Normalizasyonu
- **Maddeler:** "Ham format: saniye+ms bitişik" / "Sondaki sıfırlar kırpık" / "rpad → SS.mmm dönüşümü"
- **Sunum notu:** `rebuilt_ts` satırını (rpad/split_part zinciri) göster. Somut örnek ver: "`123` → aslında 12sn 300ms, ama bunu `rpad` etmeden sıralarsan `12345` (12sn 345ms) ile yanlış sıralanır." Bu, projenin en "zeki" SQL parçası — hakkını ver, aceleye getirme.

### Slayt 11 — Veri Bütünlüğü Savunması
💻 **KODA GEÇ**
- **Başlık:** Veri Bütünlüğü
- **Maddeler:** "ignore_errors=true riski" / "Bozuk satırlar NULL'a düşebilir" / "COALESCE ile güvenli varsayılan"
- **Sunum notu:** Sadece backend kodunu göster (`COALESCE(CAST(level AS VARCHAR), 'unknown')` gibi). Adversarial testing hikayesine **henüz değinme** — o, Bölüm 4'te çok daha güçlü bir sahnede patlayacak, burada onu erken harcamayalım.

### Slayt 12 — SQL Injection Güvenliği
💻 **KODA GEÇ**
- **Başlık:** Parametreli Sorgular
- **Maddeler:** "Kullanıcı girdisi asla metne karışmıyor" / "prepare → bind, iki ayrı aşama" / "$1, $2 yer tutucuları"
- **Sunum notu:** `conditions`/`params` dizisi + `prepared.bind(params)` satırlarını göster. Sorulması muhtemel bir soruya önden cevap ver: "checkbox'tan geldiği için güvenli değil mi diye düşünülebilir, ama kullanıcı benim UI'ımı hiç kullanmadan doğrudan API'ye istek atabilir — asıl koruma burada."

### Slayt 13 — Filtreleme Sistemi
🌐 **WEB'E GEÇ**
- **Başlık:** Filtreleme Sistemi
- **Maddeler:** "Kolon başlığına gömülü popover'lar" / "Level, File, Event, Message, Source, Timestamp" / "Her filtre → dinamik WHERE"
- **Sunum notu:** Web'de birkaç filtreyi canlı, art arda uygula (Level seç → Apply, sonra Message'a bir kelime yaz). Kombinasyon göstermek (iki filtre aynı anda) etkileyici.

### Slayt 14 — Invisible Pagination
🌐 **WEB'E GEÇ** (bu slaytı özenle pratik et)
- **Başlık:** Invisible Pagination
- **Maddeler:** "Kullanıcı hiç sayfa numarası görmüyor" / "Scroll → arka planda sessiz fetch" / "Backend'de LIMIT/OFFSET hâlâ çalışıyor"
- **Sunum notu:** **En kritik demo anlarından biri.** Tarayıcıda DevTools → Network sekmesini aç, kaydırırken her birkaç saniyede bir yeni `/logs?page=...` isteğinin **canlı** düştüğünü göster. Bunu mutlaka önceden birkaç kez prova et — DevTools açıkken doğru sekmeye zamanında geçmek pratik gerektirir.

### Slayt 15 — Filtreleme Nerede Çalışıyor?
🎤 **SADECE ANLAT** (bir önceki slaytın hemen devamı, ekran değişmeden)
- **Başlık:** Filtreleme — Tüm Veri Üzerinde
- **Maddeler:** "Tarayıcıda GÖRÜNEN kısımda değil" / "Her seferinde DuckDB'deki TÜM veri" / "Sonra sadece 3000'lik dilim"
- **Sunum notu:** Bunu **kesinlikle doğru** anlat — bu, projenin en çok yanlış anlaşılabilecek noktası. "1 milyon satırın 90 binini yüklemiş olsam da, filtre uyguladığımda tarayıcı o 90 bini değil, DuckDB'deki 1 milyonun tamamını süzüyor" cümlesini ezbere bil.

### Slayt 16 — react-window / Virtualization
🌐 **WEB'E GEÇ** + 💻 **KODA GEÇ**
- **Başlık:** react-window — Virtualization
- **Maddeler:** "30M satır, ~20 DOM node" / "rowHeight sabit → pozisyon hesabı" / "Haber bandı analojisi"
- **Sunum notu:** Web'de DevTools → Elements/Inspector aç, kaydırırken DOM'daki `<div>` sayısının **sabit kaldığını** canlı göster — çok güçlü bir görsel kanıt. Sonra VS Code'a geç, `<List rowComponent={LogRow} rowHeight={52} .../>` satırını göster.

### Slayt 17 — Performans Kanıtı — Canlı Ölçek Testi
🌐 **WEB'E GEÇ** (sunumun zirve anı, en sona/en vurguluya koy)
- **Başlık:** Performans Kanıtı
- **Maddeler:** "mock-logs-1: 2.000.000 satır/dosya" / "~60 milyon satır toplam" / "20-30 sn yükleme, sonrası akıcı"
- **Sunum notu:** Gerçekten büyük dosyayı (mock-logs-1) canlı yükle, süreyi yüksek sesle say ("...25, 26, 27 saniye, bitti"), sonra akıcı scroll + bir filtre uygula. Bu, mentörünün performans barını **doğrudan gözünün önünde** kanıtladığın an — Bölüm 3'ün son slaytı olarak bırak, izleyicinin hafızasında en taze kalan sahne bu olsun.

---

## BÖLÜM 4 — Dayanıklılık / Mühendislik Olgunluğu (Slayt 18-20)

### Slayt 18 — Kasıtlı Bozma Testi #1: Bozuk JSON
🌐 **WEB'E GEÇ** + 💻 **KODA GEÇ**
- **Başlık:** Kasıtlı Bozma Testi
- **Maddeler:** "Bir satırdan { çıkarıldı" / "level=NULL olarak insert edildi" / "Uygulama TÜMÜYLE çöktü (düzeltmeden önce)"
- **Sunum notu:** Önceden hazırlanmış bozuk bir dosyayı sürükle — artık **çökmediğini** göster (`UNKNOWN` rozetiyle satır görünür). Sonra VS Code'da üç katmanı sırayla göster: backend `COALESCE`, frontend `levelKey` null-guard, `ErrorBoundary.tsx`.

### Slayt 19 — Kasıtlı Bozma Testi #2: Race Condition
🎤 **SADECE ANLAT** + 💻 **KODA GEÇ** (canlı tekrar riskli, sözle anlat)
- **Başlık:** Race Condition
- **Maddeler:** "Upload sürerken ikinci klasör sürüklendi" / "İki işlem aynı anda çalıştı" / "uploadingRef ile çözüldü"
- **Sunum notu:** Bunu **canlı tekrarlamaya çalışma** — zamanlama gerektiriyor, sunumda başarısız olma riski var. Sözle anlat ("upload sürerken tekrar dosya sürükledim, sayaç 26/30'dan 0/30'a düştü, gerçek bir eşzamanlılık hatası buldum"), sonra VS Code'da `uploadingRef` kilidini göster.

### Slayt 20 — Üç Katmanlı Savunma Felsefesi
🎤 **SADECE ANLAT**
- **Başlık:** Üç Katmanlı Savunma
- **Maddeler:** "Kaynakta önle (backend)" / "Her yerde savun (frontend)" / "Son çare (ErrorBoundary)"
- **Sunum notu:** Bu bir **sentez** slaytı, kod göstermene gerek yok. Uçak güvenlik sistemleri analojisini kullanabilirsin: "tek bir önlem yok, birden fazla bağımsız katman birbirinin başarısız olma ihtimaline karşı sigorta."

---

## BÖLÜM 5 — Kapanış (Slayt 21-22)

### Slayt 21 — Karşılaşılan Zorluklar
🎤 **SADECE ANLAT**
- **Başlık:** Karşılaşılan Zorluklar
- **Maddeler:** "Docker kısıtı → mimari pivot" / "Stale closure bug'ları (3 kez)" / "Cross-machine font/timestamp bug'ları"
- **Sunum notu:** Hızlı, madde madde geç — bunlar zaten önceki slaytlarda detaylandırıldı, burada sadece bir liste olarak hatırlat. Soru gelirse derine inersin.

### Slayt 22 — Özet
🎤 **SADECE ANLAT**
- **Başlık:** Özet
- **Maddeler:** "Kısıt altında bağımsız mimari karar" / "Ölçekte performans (60M satır)" / "Test edilmiş dayanıklılık"
- **En altta:** "Sorularınız?"
- **Sunum notu:** Üç cümlede kapat: "Docker yasağı bizi embedded bir mimariye taşıdı, 60 milyon satırda bile pagination görünmeden çalışıyor, sistemi bilerek iki kez bozdum ve üç katmanlı bir savunma kurdum." Sonra açık kapı bırak.

---

## Genel Prova Notları

- **En az 2 tam prova** yap, kronometreyle — 22 slayt + 6 web/kod geçişi, zamanı kolayca aşabilir.
- **Kısaltma gerekirse** Slayt 8 (Stack Özeti) ve Slayt 21 (Zorluklar) en kolay atlanabilir/kısaltılabilir yerler.
- **Asla kısaltma:** Slayt 14 (Invisible Pagination), Slayt 17 (Performans Kanıtı), Slayt 18 (Bozma Testi) — bunlar en güçlü kanıtların olduğu yerler.
- Web/kod geçişlerini **önceden pencereleri hazır tutarak** (tarayıcı + VS Code + DevTools sekmeleri açık, doğru dosyalar/sekmeler seçili) hızlandır — canlı arama/açma sunumda zaman kaybettirir.
