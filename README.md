# BitKurus Cüzdan

Tek dosyalık (`cuzdan.html`), kurulum gerektirmeyen, **tarayıcı tabanlı** bir BitKurus (BK) cüzdanıdır. Tüm kriptografi tarayıcının içinde çalışır; **private key cihazından asla çıkmaz**. Ağ tarafına yalnızca public key ve imzalanmış işlem zarfı gönderilir.

> ⚠️ **Uyarı:** Bu yazılım eğitim/deneysel amaçlıdır, denetlenmemiş (unaudited) bir cüzdandır. Gerçek değerli varlıklarınızı saklamadan önce riskleri okuyun. Bkz. [Güvenlik ve sorumluluk reddi](#güvenlik-ve-sorumluluk-reddi).

---

## İçindekiler

- [Özellikler](#özellikler)
- [Hızlı başlangıç](#hızlı-başlangıç)
- [Arayüz ve cüzdan durumları](#arayüz-ve-cüzdan-durumları)
- [Adım adım kullanım](#adım-adım-kullanım)
- [Güvenlik mimarisi](#güvenlik-mimarisi)
- [Ağ / API](#ağ--api)
- [İşlem (transaction) formatı](#i̇şlem-transaction-formatı)
- [Değer ve ondalık işleme](#değer-ve-ondalık-i̇şleme)
- [Yedek dosyası formatı](#yedek-dosyası-formatı)
- [Sorun giderme](#sorun-giderme)
- [Teknik detaylar](#teknik-detaylar)
- [GitHub Pages ile yayınlama](#github-pages-ile-yayınlama)
- [Geliştirme ve katkı](#geliştirme-ve-katkı)
- [Güvenlik ve sorumluluk reddi](#güvenlik-ve-sorumluluk-reddi)
- [Lisans](#lisans)

---

## Özellikler

- 🔐 **İstemci taraflı anahtarlar** — Ed25519 anahtar çifti tarayıcıda üretilir; private key sunucuya gönderilmez.
- 🆕 **Yeni cüzdan oluşturma** — tek tıkla rastgele güvenli anahtar üretimi.
- 📥 **Private key ile içe aktarma** — mevcut bir cüzdanı 64 karakterlik hex private key ile açma.
- 💾 **Bu cihazda şifreli saklama** — private key, parolayla şifrelenip (`PBKDF2 + AES-GCM`) tarayıcının `localStorage`'ında tutulur; sonra tek parolayla açılır.
- 🗄️ **Şifreli yedek indir / yedekten aç** — cüzdanı başka cihaza taşımak veya güvenle saklamak için `.json` yedek dosyası.
- 💸 **Transfer** — Ed25519 imzalı işlem zarfı hazırlama ve ağa gönderme.
- 📊 **Bakiye ve token görüntüleme** — aktif token (UTXO) listesi ve toplam bakiye.
- 🔎 **İşlem sorgu** — `tx_id` ile işlem durumu sorgulama.
- 🔒 **Durum bazlı arayüz** — cüzdan açılana kadar sol panel kilit overlay'i ile kaplanır.
- 🌐 **Sıfır kurulum** — bağımlılık `@noble/ed25519` CDN'den çekilir; ayrı `npm install` yok.

---

## Hızlı başlangıç

Bu, tek bir HTML dosyasıdır ancak ES modülleri ve CDN/ağ istekleri nedeniyle **`file://` ile değil, bir HTTP sunucusu üzerinden** açılmalıdır.

### Yerelde çalıştırma

```bash
# Proje klasöründe:
python3 -m http.server 4173
```

Ardından tarayıcıdan:

```
http://localhost:4173/cuzdan.html
```

Alternatif sunucular:

```bash
npx serve .          # Node.js
php -S localhost:4173 # PHP
```

> Dosyayı doğrudan çift tıklayıp `file://` ile açarsanız sayfa sizi uyarır; tarayıcı güvenliği nedeniyle modül/CDN/API istekleri çalışmayabilir.

---

## Arayüz ve cüzdan durumları

Arayüz, cüzdanın durumuna göre kendini uyarlar. Sağ üstteki rozet anlık durumu gösterir:

| Durum | Rozet | Sol panel | Sağ panel |
|------|-------|-----------|-----------|
| **Cüzdan yok** | `Cüzdan yok` (gri) | Kilit overlay'i ile kaplı | "Yeni cüzdan oluştur" + "Başka bir cüzdan aç" |
| **Kilitli** (public key var, private key oturumda yok) | `Kilitli` (turuncu) | Kilit overlay'i ile kaplı | Cihaz parolasıyla kilidi aç / yeniden içe aktar |
| **Açık** (private key oturumda) | `Açık` (yeşil) | Tam erişilebilir (bakiye, tokenlar, transfer) | Kilitle, cihazda sakla, yedek indir |

- **Kilit overlay'i:** Cüzdan açılana kadar sol kolon (bakiye + tokenlar + transfer) bulanıklaştırılmış bir kilit katmanıyla kaplanır ve tıklamaları engeller. Kilit açılınca otomatik kaybolur.
- **İlerlemeli gösterim:** Cihazda saklama, yedek, API adresi ve işlem sorgu gibi nadir kullanılan işlemler açılır‑kapanır bölümlerde toplanmıştır.

---

## Adım adım kullanım

### 1) Yeni cüzdan oluşturma
Sağdaki **Cüzdan** panelinden **"Yeni cüzdan oluştur"**a basın. Anahtar çifti üretilir, cüzdan anında **açık** duruma geçer. Public adresi hero alanında görürsünüz; **"Adresi kopyala"** ile kopyalayabilirsiniz.

> Yeni cüzdan oluşturmak yalnızca rastgele bir anahtar üretir; bakiyeniz başlangıçta 0'dır.

### 2) Mevcut cüzdanı açma
**"Başka bir cüzdan aç"** bölümünden:
- **Private key ile aç:** 64 karakterlik hex private key'i yapıştırıp **"Private key ile aç"**.
- **Yedek dosyasından aç:** `.json` yedeği seçip yedek parolasını girin, **"Yedekten aç"**.

### 3) Bu cihazda şifreli saklama
Cüzdan **açıkken** → **"Bu cihazda sakla"** bölümünde bir cihaz parolası belirleyip **"Bu cihazda şifreli sakla"**ya basın. Private key, parolayla şifrelenip tarayıcıya kaydedilir. Sonraki gelişlerde **Kilitli** durumda cihaz parolasıyla tek tıkla açarsınız.

> Parola en az **8 karakter** olmalıdır. Parolayı unutursanız bu kayıt **kurtarılamaz** — mutlaka yedek de alın.

### 4) Şifreli yedek alma / geri yükleme
- **Yedek indir:** Cüzdan açıkken **"Şifreli yedek indir"** bölümünde yedek parolası girip indirin. `bitkurus-wallet-XXXXXXXXXX.json` dosyası kaydedilir.
- **Geri yükleme:** "Başka bir cüzdan aç" → yedek dosyası + parola → **"Yedekten aç"**.

### 5) Transfer (gönderim)
1. **Alıcı public key** (64 karakter hex) ve **tutar** girin.
2. **"İmzalı zarf hazırla"** — ağ durumu tazelenir, uygun token'lar girdi olarak seçilir, zarf imzalanır ve metin kutusunda görünür.
3. **"Ağ'a gönder"** — imzalı zarf ağa iletilir; ardından bakiye otomatik yenilenir.

> İşlem imzalamak için cüzdanın **açık** olması (private key oturumda) gerekir.

### 6) İşlem sorgu
**Gelişmiş ayarlar** → **Tx id** girip **"Durum sorgula"** ile işlemin ağdaki durumunu görüntüleyin.

### 7) Kilitleme
**"Cüzdanı kilitle"** private key'i oturumdan siler (public key kalır, bakiye okunmaya devam eder). Cihazda şifreli kaydınız varsa kilidi tekrar parolayla açarsınız.

---

## Güvenlik mimarisi

- **Anahtar üretimi/imzalama:** [`@noble/ed25519`](https://github.com/paulmillr/noble-ed25519) (sürüm `2.2.3`, jsDelivr CDN üzerinden ES modülü).
- **Private key nerede durur?**
  - Bellekte yalnızca cüzdan **açıkken** (`oturum` boyunca).
  - İsteğe bağlı olarak şifrelenmiş şekilde `localStorage`'da.
  - **Public key** kolaylık için `localStorage`'da düz tutulur.
- **`localStorage` anahtarları:**
  - `bk.publicKey` — public key (hex, düz metin).
  - `bk.encryptedPrivateKey` — şifreli private key kasası (JSON).
- **Şifreleme (yerel kasa ve yedek için aynı şema):**
  - Anahtar türetme: **PBKDF2‑SHA256**, **250.000** iterasyon, 16 baytlık rastgele `salt`.
  - Simetrik şifreleme: **AES‑GCM 256‑bit**, 12 baytlık rastgele `iv`.
  - Parola minimum 8 karakter.
- **Ağ'a ne gider?** Yalnızca public key (bakiye okuma için) ve imzalı işlem zarfı. Private key **hiçbir zaman** ağa gönderilmez.

> Bu bir tarayıcı cüzdanıdır: güvenliğiniz, cihazınızın ve tarayıcınızın güvenliği kadardır. Ortak/halka açık bilgisayarlarda "bu cihazda sakla" özelliğini kullanmayın.

---

## Ağ / API

Varsayılan API adresi: **`https://bitkurush.org`** (arayüzdeki **Gelişmiş ayarlar → API adresi** alanından değiştirilebilir).

`GET /api` çıktısındaki uç noktalar:

| Uç nokta | Açıklama |
|----------|----------|
| `GET /api` | Servis bilgisi ve uç nokta listesi |
| `GET /api/wallet/{public_key}` | Cüzdan bakiyesi ve token (UTXO) listesi |
| `POST /api/tx/submit` | İmzalı işlem zarfını gönderme |
| `GET /api/tx/{id}` | İşlem durumu sorgulama |
| `GET /api/token/{id}` | Token detayı |
| `GET /api/validators/{token_id}` | Token doğrulayıcıları |
| `GET /api/network` | Düğüm, eşler (peers) ve konsensüs bilgisi |
| `GET /api/state` | Ağ durumu |

### Örnek: cüzdan yanıtı

```json
{
  "result": "ok",
  "state_hash": "....",
  "data": {
    "public_key": "....",
    "balance": "777.997073991854272442",
    "is_commission_wallet": false,
    "validator_commission_rate_ppm": 200,
    "tokens": [
      {
        "token_id": "commission-token-....",
        "value": "0.051297982406494487",
        "status": "active",
        "version": 1,
        "created_at": "2026-06-03T21:53:25.000000Z",
        "origin": "validator_commission",
        "origin_tx_id": "commission-...."
      }
    ]
  },
  "node_signature": "...."
}
```

> Değerler **18 ondalıklı string** olarak gelir. Doğrulayıcı komisyonu `validator_commission_rate_ppm` (örn. `200` = %0,02) ile ifade edilir.

---

## İşlem (transaction) formatı

İşlemler bir **UTXO / token** modeli üzerinde çalışır: girdiler harcanan token'lar, çıktılar yeni token'lardır (alıcı + para üstü).

İmzalanan payload:

```json
{
  "tx_id": "wallet-tx-<base36-zaman>-<8bayt-hex>",
  "type": "transfer",
  "sender": "<gönderen public key>",
  "nonce": "<tx_id>-nonce",
  "inputs": [
    { "token_id": "....", "version": 1 }
  ],
  "outputs": [
    { "token_id": "<tx_id>-out-recv",   "value": "1.000000000000000000", "owner": "<alıcı public key>" },
    { "token_id": "<tx_id>-out-change", "value": "4.500000000000000000", "owner": "<gönderen public key>" }
  ]
}
```

- **`type`:** Tek girdi ve para üstü yoksa `"transfer"`, aksi halde `"split"`.
- **İmza:** Payload **kanonik JSON**'a çevrilir (tüm anahtarlar özyinelemeli olarak alfabetik sıralanır), Ed25519 ile imzalanır ve sonuç `signature` alanı olarak zarfa eklenir:

```json
{ ...payload, "signature": "<hex imza>" }
```

- Bu zarf `POST /api/tx/submit` ile gönderilir.

### Kanonik JSON kuralı

Anahtarlar **alfabetik** sıralanır; diziler sırasını korur; değerler `JSON.stringify` ile serileştirilir. Aynı payload her zaman aynı baytları üretir — bu sayede imza doğrulanabilir.

---

## Değer ve ondalık işleme

Token değerleri ağda **18 ondalıklı** ondalık string'lerdir (`"0.051297982406494487"`). Bu değerler **asla** JavaScript `Number` (kayan nokta / float) tipine çevrilmez; çünkü float yuvarlama kalıntısı (dust) çıktı toplamını girdinin üzerine çıkarıp ağ tarafından

```
Output value exceeds input value.
```

hatasına yol açar.

Bunun yerine cüzdan tüm aritmetiği **18 ondalığa ölçeklenmiş BigInt tamsayılar** üzerinde **tam (exact)** yapar:

- `parseUnits("0.05")` → ham string'i kayan noktaya uğramadan `BigInt`'e çevirir.
- `change = totalUnits − amountUnits` → tamsayı çıkarması.
- Sonuç: **`recv + change` her zaman seçilen girdilerin toplamına birebir eşittir**; çıktılar matematiksel olarak girdiyi geçemez.

Ayrıca her **"İmzalı zarf hazırla"** öncesi cüzdan otomatik yenilenir; böylece bir önceki gönderimde harcanmış token tekrar girdi olarak seçilmez.

---

## Yedek dosyası formatı

İndirilen şifreli yedek (ve `localStorage`'daki kasa) şu yapıdadır:

```json
{
  "version": 1,
  "public_key": "<public key hex>",
  "kdf": "PBKDF2-SHA256",
  "iterations": 250000,
  "salt": "<16 bayt hex>",
  "iv": "<12 bayt hex>",
  "ciphertext": "<AES-GCM şifreli private key, hex>"
}
```

- Geri yüklemede dosya bu şemaya uymalıdır; aksi halde "Bu dosya şifreli cüzdan yedeği değil" uyarısı verilir.
- Çözülen private key'den türetilen public key, yedekteki `public_key` ile karşılaştırılır; uyuşmazsa içe aktarma reddedilir.

---

## Sorun giderme

| Belirti | Sebep | Çözüm |
|--------|-------|-------|
| `file://` uyarısı, CDN/modül hatası | Dosya doğrudan açıldı | Yerel HTTP sunucusuyla açın (bkz. [Hızlı başlangıç](#hızlı-başlangıç)) |
| `Output value exceeds input value.` | (Giderildi) Float yuvarlama veya bayat token | Güncel sürüm BigInt + otomatik yenileme ile bunu önler |
| `Yetersiz bakiye` | Seçilen tutar mevcut aktif token toplamından büyük | Tutarı düşürün veya bakiyeyi yenileyin |
| `Private key oturumda değil` | Cüzdan kilitli | Cihaz parolasıyla açın, private key içe aktarın veya yedekten açın |
| `Parola veya şifreli kayıt hatalı` | Yanlış parola ya da bozuk kasa | Doğru parolayı girin; gerekirse yedekten geri yükleyin |
| Bakiye 0 / token yok | Ağdan okunmadı veya gerçekten boş | **Yenile**'ye basın; API adresini kontrol edin |

---

## Teknik detaylar

- **Tek dosya:** Tüm HTML, CSS ve JavaScript `cuzdan.html` içinde (harici build/araç yok).
- **Bağımlılık:** `@noble/ed25519@2.2.3` (jsDelivr ESM). Çevrimdışı kullanım için bu modülü yerelleştirmeniz gerekir.
- **Tarayıcı API'leri:** Web Crypto (`crypto.subtle` — PBKDF2, AES‑GCM, SHA‑256), `BigInt`, `localStorage`, `fetch`, `clipboard`.
- **Gereksinim:** Güncel bir masaüstü/mobil tarayıcı (Chrome, Firefox, Edge, Safari).
- **Saklanan veriler:** Yalnızca tarayıcının `localStorage`'ı; sunucuda hesap/oturum yoktur.

### Depo içeriği

| Dosya | Açıklama |
|-------|----------|
| `cuzdan.html` | Cüzdan uygulamasının tamamı |
| `README.md` | Bu doküman |
| `LICENSE` | Lisans (MIT) |
| `.gitignore` | `commands.json`, `subdomains.json`, `.claude/` hariç tutulur |

> `commands.json` ve `subdomains.json` yalnızca yerel hosting/altyapı yapılandırması içindir ve depoya **dahil edilmez**.

---

## GitHub Pages ile yayınlama

Uygulamayı `https://` üzerinden ücretsiz yayınlayabilirsiniz (bu, `file://` ve CDN sorunlarını da çözer):

1. GitHub'da depo **Settings → Pages**.
2. **Source:** `Deploy from a branch` → **Branch:** `main` → klasör `/ (root)` → **Save**.
3. Birkaç dakika içinde yayınlanır:
   `https://sencerhan.github.io/cuzdan/cuzdan.html`

> İsterseniz `cuzdan.html` dosyasını `index.html` olarak da kopyalayın; o zaman adres kök dizinden (`.../cuzdan/`) açılır.

---

## Geliştirme ve katkı

- Derleme adımı yoktur; `cuzdan.html`'i düzenleyip kaydedin ve tarayıcıda yenileyin.
- Kod tek dosyada düzenli bölümlere ayrılmıştır: stil (`<style>`), arayüz (`<body>`) ve mantık (`<script type="module">`).
- Katkı için: bir fork açın, değişikliğinizi yapın, açıklayıcı bir PR gönderin.
- Hata/öneri için GitHub **Issues** kullanın.

---

## Güvenlik ve sorumluluk reddi

- Bu yazılım **"olduğu gibi" (as is)**, herhangi bir garanti olmaksızın sunulur.
- **Denetlenmemiştir (unaudited).** Kriptografik uygulama ve ağ protokolü bağımsız olarak doğrulanmamıştır.
- Private key'inizi kaybetmeniz veya parolanızı unutmanız hâlinde varlıklarınız **geri getirilemez**.
- Anahtarlarınızı/yedeklerinizi güvenli saklayın; parolanızı kimseyle paylaşmayın; phishing'e dikkat edin.
- Kullanım tamamen **sizin sorumluluğunuzdadır**. Geliştiriciler oluşabilecek kayıplardan sorumlu değildir.

---

## Lisans

[MIT](LICENSE) — Dilediğiniz lisansla değiştirebilirsiniz.
