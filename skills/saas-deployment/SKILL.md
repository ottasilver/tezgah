---
name: saas-deployment
description: >
  SaaS uygulamasını production'a taşı. Dokploy, Railway veya Fly.io ile
  deployment, domain yapılandırması, SSL, ortam değişkenleri, CI/CD,
  izleme ve operasyonel hazırlık. Bu skill'i kullanıcı deploy, yayınlama,
  production, hosting, domain, SSL, CI/CD, monitoring veya "siteyi canlıya
  al" ile ilgili bir şey istediğinde kullan. "Deploy et", "yayınla",
  "canlıya al", "Dokploy'a koy", "domain bağla" gibi ifadeler tetikler.
---

# SaaS Deployment — Production'a Taşıma ve Operasyon

Bu skill, bir SaaS uygulamasını development ortamından production'a taşır ve operasyonel hazırlığını sağlar. Deployment "kodu sunucuya yüklemek" değildir — tüm dış servisleri production moduna geçirmek, güvenliği doğrulamak ve izleme kurmaktır.

**Bağımlılık:** Bu skill **saas-launcher** orkestratör skill'inin Faz 8'idir. Bağımsız olarak da kullanılabilir. İdeal olarak tüm diğer fazlar tamamlandıktan sonra uygulanır.

**Bağlı skill'ler:** Tüm diğer skill'lerin production geçiş adımları bu skill'de toplanır.
- **saas-auth** — Production OAuth callback URL'leri, production secret
- **saas-payments** — Live Stripe/LS anahtarları, production webhook URL
- **saas-email** — DNS doğrulaması tamamlanmış olmalı
- **saas-landing-seo** — SEO kontrol listesi geçirilmiş olmalı
- **saas-api-security** — Güvenlik kontrol listesi geçirilmiş olmalı

---

## Hosting Platformu Seçimi

### Dokploy — Varsayılan Önerimiz

**Ne zaman seç:** Kendi sunucunu kontrol etmek isteyen, maliyeti düşük tutmak isteyen her proje için.

Neden: Açık kaynak, self-hosted PaaS platformu. Kendi VPS'ine (Hetzner, DigitalOcean, Contabo vb.) kurulur — Docker tabanlı deployment, otomatik SSL (Let's Encrypt), GitHub entegrasyonu ile otomatik deploy, preview deployment desteği, dahili monitoring ve log yönetimi. Platform ücretsiz, sadece VPS maliyeti var.

Maliyet: Hetzner'de aylık ~€4-5'lık bir VPS ile başlanabilir. Çoğu erken SaaS için fazlasıyla yeterli. Traffic artınca VPS'i büyüt — platform maliyeti hâlâ $0.

Ne zaman yetmez: Sunucu yönetiminden tamamen kaçınmak isteyenler (managed platform tercih edenler), global edge network gerektiren uygulamalar.

### Railway

**Ne zaman seç:** Veritabanı + uygulama tek platformda olsun isteyenler için.

Neden: PostgreSQL, Redis, MySQL gibi servisleri aynı platformda barındırır. Cron job ve background worker desteği. Docker container çalıştırma imkânı.

Ücretsiz katman: Aylık $5 kredi. Küçük projeler için yeterli ama traffic artınca hızla ücretli plana geçiş gerekir.

### Fly.io

**Ne zaman seç:** Global dağıtım, container bazlı deployment, WebSocket veya uzun süreli bağlantılar gerektiren projeler için.

Neden: Container'ları birden fazla bölgede çalıştırır. Persistent volume (kalıcı disk) desteği. WebSocket ve long-polling doğal desteklenir.

Ek yük: Dockerfile yazmak ve yönetmek gerekir. Dokploy veya Railway kadar kolay dashboard yönetimi yok.

---

## Production'a Geçiş Sırası

Deployment sadece "kodu yükle" değildir. Şu sırayı takip et:

### Adım 1: Production Ortam Değişkenleri

Development'ta kullandığın her değişkenin production karşılığını oluştur. Bunlar farklı olmalıdır:

**Auth:**
- `NEXTAUTH_SECRET` — Production için yeni değer üret (openssl rand -base64 32). Development ile aynı olmamalı.
- `NEXTAUTH_URL` — Production domain'i: `https://uygulamam.com`
- Google OAuth Client: Aynı client kullanılabilir ama callback URL'lerine production domain eklenmiş olmalı.

**Ödeme (bkz. saas-payments):**
- Stripe/LS anahtarları: Test anahtarlarından live anahtarlara geç. `sk_test_` → `sk_live_`, `pk_test_` → `pk_live_`.
- Price ID'ler: Live modda oluşturulan ürünlerin ID'leri test modundakilerden farklıdır. Güncelle.
- Webhook secret: Production webhook endpoint'i için yeni secret oluşturulur.

**Veritabanı:**
- Production veritabanı instance'ı kullan. Development DB'sini production'da kullanma.
- Bağlantı string'inde connection pooling parametrelerini ekle (serverless ortam için).

**E-posta:**
- API anahtarı aynı kalabilir ama domain doğrulamasının tamamlanmış olduğunu kontrol et.

**Genel:**
- `NEXT_PUBLIC_SITE_URL` — Production domain'i: `https://uygulamam.com`

### Adım 2: Platform Kurulumu

Seçilen hosting platformunda:
1. Yeni proje oluştur ve GitHub repo'sunu bağla
2. Ortam değişkenlerini ekle (Adım 1'deki production değerleri)
3. İlk deploy'u tetikle
4. Build'in başarılı olduğunu doğrula

### Adım 3: Domain Yapılandırması

**DNS kayıtları:**
- Hosting platformunun verdiği IP veya CNAME kaydını domain sağlayıcında ekle
- www → non-www yönlendirmesi ayarla (veya tersi — birini seç, tutarlı ol)
- SSL sertifikası otomatik oluşturulmasını bekle (genellikle birkaç dakika)

**DNS yayılımı:** 15 dakika ile 48 saat arasında sürebilir. `dig uygulamam.com A` komutu ile yayılımı kontrol et. Sabırlı ol.

### Adım 4: Ödeme Sistemi Production Geçişi

Bu adım kritiktir ve dikkatli yapılmalı (detaylar **saas-payments** skill'inde):

1. Ödeme sağlayıcı dashboard'unda live mode'a geç
2. Live modda ürünler ve fiyatlar oluştur
3. Production webhook endpoint'i ekle: `https://uygulamam.com/api/stripe/webhook` (veya Lemon Squeezy karşılığı)
4. Dinlenecek event'leri seç
5. Webhook signing secret'ı ortam değişkenlerine ekle
6. Müşteri portalını yapılandır

### Adım 5: Auth Provider Production Geçişi

Google OAuth (detaylar **saas-auth** skill'inde):
1. Google Cloud Console'da OAuth Consent Screen'i "Published" yap
2. Callback URL'lerine production domain'i ekle
3. App review gerekiyorsa birkaç gün önceden başvur

### Adım 6: E-posta DNS Doğrulaması

E-posta servisinin dashboard'unda domain doğrulamasının tamamlandığını kontrol et (detaylar **saas-email** skill'inde). SPF, DKIM ve DMARC kayıtlarının üçü de "verified" olmalı.

### Adım 7: Son Doğrulama

Deploy tamamlandıktan sonra şu testleri yap:

**Fonksiyonel testler:**
- Kayıt/giriş akışı çalışıyor mu?
- Ödeme akışı çalışıyor mu? (Stripe live modda test kartı ile gerçek ödeme yap, sonra iade et)
- E-postalar teslim ediliyor mu? (spam'a düşmüyor mu?)
- Webhook'lar çalışıyor mu? (ödeme sonrası plan aktifleşiyor mu?)
- Korumalı sayfalar giriş yapmadan erişilemez mi?

**SEO testleri (bkz. saas-landing-seo):**
- sitemap.xml erişilebilir mi?
- robots.txt erişilebilir mi?
- OG görselleri doğru çalışıyor mu?

**Performans testi:**
- Lighthouse skoru 90+ mı?
- İlk yükleme süresi kabul edilebilir mi?

---

## İzleme ve Operasyon

Uygulama production'a çıktıktan sonra izleme hayati önem taşır. Görmediğin hatayı düzeltemezsin, bilmediğin kesintiye müdahale edemezsin.

### Uptime Monitoring

Uygulamanın erişilebilir olup olmadığını düzenli kontrol eden dış servis. Health check endpoint'ini (/api/health) dakikada bir kontrol ettirir.

Önerilen servisler: BetterStack (eski adıyla Better Uptime), UptimeRobot. İkisinin de ücretsiz katmanı yeterli.

Kesinti olduğunda e-posta veya SMS ile bildirim alırsın. Müşterilerinden önce sen öğrenmiş olursun.

### Error Tracking

Uygulamada oluşan hataları toplayan, gruplayan ve alert gönderen servis. Kullanıcılar hata raporlamaz — sessizce giderler.

Önerilen: Sentry. Ücretsiz katmanı çoğu erken SaaS için yeterli. Entegrasyonu basit — birkaç satır konfigürasyon.

Sentry'nin sağladıkları:
- Hata mesajı ve stack trace
- Hangi kullanıcıda olduğu
- Hangi tarayıcı/cihazda olduğu
- Hata sıklığı ve trend
- Yeni hata algılandığında alert

### Analytics

Kullanıcıların uygulamayı nasıl kullandığını anlamak için.

Başlangıç için: Plausible (gizlilik odaklı, basit) veya PostHog (zaten analytics skill'inde kurulu). Google Analytics da seçenek ama GDPR/KVKK açısından cookie consent gerektirir.

İzlenmesi gereken temel metrikler:
- Aylık ziyaretçi sayısı
- Kayıt oranı (ziyaretçi → kayıt)
- Dönüşüm oranı (kayıt → ödeme)
- Aktif kullanıcı sayısı (günlük/haftalık/aylık)
- Churn oranı (iptal eden / toplam abone)

### Log Yönetimi

Dokploy, Railway ve Fly.io dahili log görüntüleme sunar. Başlangıç için yeterli. Büyüdükçe Axiom, Datadog veya LogTail gibi bir log toplama servisine geç.

---

## CI/CD (Sürekli Entegrasyon / Sürekli Deployment)

### Basit Yaklaşım (Önerilen Başlangıç)

Dokploy veya Railway GitHub repo'na bağlandığında zaten CI/CD yapar:
- `main` branch'e push → otomatik production deploy
- PR açıldığında → otomatik preview deploy (Dokploy)

Bu çoğu erken SaaS için yeterlidir.

### GitHub Actions ile Ek Kontroller

Büyüdükçe deploy öncesi otomatik kontroller ekle:
- Lint (kod stili kontrolü)
- Build test (build başarılı mı?)
- Type check (TypeScript hataları var mı?)
- Opsiyonel: test suite (birim ve entegrasyon testleri)

Bu kontroller PR'da başarısız olursa merge engellenebilir — production'a bozuk kod gitmesini önler.

---

## Yedekleme Stratejisi

### Veritabanı Yedekleme

- MongoDB Atlas: Otomatik daily backup (ücretsiz katmanda dahil)
- Supabase: Point-in-time recovery Pro planda dahil, ücretsiz planda haftalık backup
- Railway PostgreSQL: pg_dump ile manuel backup veya otomatik backup (ücretli plan)

### Kod Yedekleme

Git zaten tüm kodu versiyonlar. GitHub/GitLab repo'su silinmedikçe kod güvende. Ek önlem: lokal clone'u güncel tut.

### Ortam Değişkenleri Yedekleme

Platform (Dokploy, Railway) çökerse veya VPS'in arızalanırsa ortam değişkenlerin kaybolabilir. Güvenli bir yerde (1Password, Bitwarden gibi şifre yöneticisi) bir kopyasını tut.

---

## Ölçeklendirme Düşünceleri (Büyüdükçe)

Erken aşamada ölçeklendirme düşünme — premature optimization'dan kaçın. Ama şu noktaları aklında tut:

**Dokploy'da büyüdükçe:**
- VPS kaynakları (CPU, RAM) yetmezse → daha güçlü VPS'e geç veya birden fazla sunucu kur
- Ağır işleri background job'lara taşı (Inngest, Trigger.dev, BullMQ)
- Yüksek traffic için Cloudflare CDN ekle (global düşük latency)
- Dokploy birden fazla sunucuya deployment destekler — horizontal scaling mümkün

**Veritabanında büyüdükçe:**
- Connection pooling: serverless'ta bağlantı sayısı çabuk tükenir → pooler kullan
- Index'ler: sorgu yavaşlayınca oluştur, baştan gerekmez
- Read replica: okuma ağırlıklı uygulamalarda veritabanı yükünü dağıtır

**Genel ilke:** "Optimize etmeden önce ölç." Bir darboğaz yaşamadığın sürece mevcut yapı yeterlidir.

---

## Production Geçiş Ana Kontrol Listesi

### Güvenlik
- [ ] `NEXTAUTH_SECRET` production için benzersiz üretildi
- [ ] Ödeme anahtarları live mode'a geçirildi
- [ ] Webhook URL'leri production domain'e güncellendi
- [ ] Webhook imza doğrulama aktif
- [ ] `.env.local` ve hassas dosyalar `.gitignore`'da
- [ ] Rate limiting en azından login ve webhook endpoint'lerinde aktif
- [ ] Error tracking (Sentry) kuruldu

### Altyapı
- [ ] Production veritabanı instance'ı oluşturuldu ve bağlandı
- [ ] Connection pooling yapılandırıldı (serverless ortam)
- [ ] Domain DNS kayıtları yapılandırıldı
- [ ] SSL sertifikası aktif (HTTPS çalışıyor)
- [ ] www → non-www yönlendirmesi ayarlandı

### E-posta
- [ ] E-posta DNS kayıtları (SPF, DKIM, DMARC) doğrulandı
- [ ] E-posta domain'i servis tarafında verify edildi
- [ ] Test e-postası gönderildi ve inbox'a düştüğü doğrulandı

### Ödeme
- [ ] Live modda ürünler ve fiyatlar oluşturuldu
- [ ] Production webhook endpoint eklendi ve test edildi
- [ ] Müşteri portalı yapılandırıldı
- [ ] Test ödeme yapılıp plan aktifleştiği doğrulandı (sonra iade et)

### SEO
- [ ] sitemap.xml erişilebilir
- [ ] robots.txt erişilebilir
- [ ] OG görselleri test edildi
- [ ] Google Search Console'a site eklendi
- [ ] Lighthouse skoru 90+

### İzleme
- [ ] Uptime monitoring kuruldu (BetterStack / UptimeRobot)
- [ ] Error tracking kuruldu (Sentry)
- [ ] Analytics aktif

### Son Fonksiyonel Test
- [ ] Kayıt/giriş akışı çalışıyor
- [ ] Ödeme akışı çalışıyor
- [ ] E-postalar teslim ediliyor
- [ ] Webhook'lar çalışıyor
- [ ] Korumalı sayfalar korunuyor
- [ ] Health check endpoint yanıt veriyor
- [ ] Build hatasız tamamlanıyor

---

## Gotchas

- **İlk deploy'da build hatası.** Local'de çalışan kod Dokploy'da çökebilir — server component'lerdeki client-only kodlar, eksik ortam değişkenleri veya case-sensitive dosya adları (Linux case-sensitive, macOS değil) en sık nedenler.
- **Docker build cache.** Bazen eski build cache'i sorunlara neden olur. Dokploy dashboard'ında yeniden deploy ederken cache temizlemeyi dene.
- **Preview vs. production ortam değişkenleri.** Dokploy'da her uygulama için ayrı ortam değişkenleri tanımlanabilir. Preview deploy'larda test anahtarlarını, production'da live anahtarlarını kullan.
- **DNS propagation sabır gerektirir.** 24-48 saat sürebilir. "Deploy ettim ama site açılmıyor" panikleme — DNS yayılımını bekle.
- **Stripe live modda ilk test.** Live modda gerçek kart ile küçük bir test ödemesi yap, webhook'un çalıştığını doğrula, sonra Stripe dashboard'dan iade et. "Production'da çalışıyor varsayıyorum" deme — doğrula.
- **Gece launch yapma.** Sorun çıkınca müdahale edebileceğin saatlerde launch et. İdeal: hafta içi sabah.
- **Rollback planı.** Bir şeyler ters giderse önceki deploy'a geri dönebilmelisin. Dokploy'da önceki Docker image'a geri dönülebilir. Git'te revert commit yapıp yeniden deploy da alternatif. Platform seçerken bu yeteneği kontrol et.
