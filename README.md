# Tezgah

[![npm version](https://img.shields.io/npm/v/tezgah.svg)](https://www.npmjs.com/package/tezgah)
[![CI](https://github.com/komunite/tezgah/actions/workflows/ci.yml/badge.svg)](https://github.com/komunite/tezgah/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**Claude Code ile production-ready SaaS kur.**

Tezgah, [Claude Code](https://claude.ai/claude-code) için yazılmış bir skill setidir. Sıfırdan çalışan bir SaaS uygulaması kurmanın tüm adımlarını — veritabanı, kimlik doğrulama, ödeme, e-posta, landing page, yasal uyumluluk, API güvenliği, test, analytics ve deployment — kapsar.

Türkiye'deki solopreneur'ler için tasarlandı. Stripe Türkiye kısıtlamaları, Lemon Squeezy MoR avantajı, KVKK uyumluluğu gibi konular özellikle ele alınmıştır.

> **English:** Tezgah is a Claude Code skill kit for building production-ready SaaS apps. Currently in Turkish — contributions for i18n are welcome.

---

## Hızlı Başlangıç

```bash
npx tezgah init
```

Bu komut `skills/` klasörüne 12 skill dosyası kurar. Sonrasında Claude Code'da:

```
/saas-launcher
```

komutuyla orkestratörü başlat. Sana sorular soracak, tech stack kararlarını verecek ve her katmanı adım adım kuracak.

## CLI Komutları

```bash
npx tezgah init                        # Tüm skill'leri kur
npx tezgah add saas-auth               # Tek bir skill ekle
npx tezgah remove saas-storage         # Bir skill'i kaldır
npx tezgah update                      # Kurulu skill'leri güncelle
npx tezgah doctor                      # Kurulumu kontrol et
npx tezgah list                        # Mevcut skill'leri listele

# Seçenekler
npx tezgah init --dir .claude/skills   # Farklı klasöre kur
npx tezgah init --force                # Mevcut dosyaların üstüne yaz
npx tezgah list --no-color             # Renksiz çıktı
```

## Skill'ler

### Orkestratör

| Skill | Açıklama |
|-------|----------|
| `saas-launcher` | Tüm süreci yöneten ana skill. Keşif görüşmesi yapar, tech stack kararlarını verir, her katmanı sırayla ilgili uzman skill'e devreder. |

### Uzman Skill'ler

Her biri bağımsız olarak da kullanılabilir:

| Skill | Faz | Kapsam |
|-------|-----|--------|
| `saas-database` | 2 | Supabase kurulumu, şema tasarımı, RLS, migration, connection pooling |
| `saas-auth` | 3 | Google OAuth, Magic Link, e-posta/şifre, JWT oturum, middleware |
| `saas-payments` | 4 | Stripe / Lemon Squeezy, checkout, webhook, abonelik yönetimi |
| `saas-email` | 5 | Resend / Mailgun, DNS (SPF, DKIM, DMARC), e-posta şablonları |
| `saas-storage` | 6.5 | Supabase Storage, dosya yükleme, RLS güvenliği, görsel optimizasyonu |
| `saas-landing-seo` | 6 | Landing page bileşenleri, SEO, sitemap, Open Graph, blog |
| `saas-legal` | 7 | KVKK/GDPR, gizlilik politikası, kullanım koşulları, çerez onayı |
| `saas-api-security` | 8 | Rate limiting, plan bazlı erişim, input validation, CORS |
| `saas-testing` | 9 | Vitest, Playwright E2E, webhook testi, CI entegrasyonu |
| `saas-analytics` | 10 | PostHog, event tracking, feature flags, session replay |
| `saas-deployment` | 11 | Dokploy / Railway / Fly.io, domain, SSL, CI/CD, izleme |

### Faz Akışı

```
Faz 0   Keşif Görüşmesi         ─── saas-launcher
Faz 1   Tech Stack Kararları     ─── saas-launcher
Faz 2   Altyapı + Veritabanı    ─── saas-database
Faz 3   Kimlik Doğrulama         ─── saas-auth
Faz 4   Ödeme Sistemi            ─── saas-payments
Faz 5   E-posta Altyapısı        ─── saas-email
Faz 6   Landing Page & SEO       ─── saas-landing-seo
Faz 6.5 Dosya Depolama           ─── saas-storage (opsiyonel)
Faz 7   Yasal Uyumluluk          ─── saas-legal
Faz 8   API Güvenliği            ─── saas-api-security
Faz 9   Test                     ─── saas-testing
Faz 10  Analytics                ─── saas-analytics
Faz 11  Deployment               ─── saas-deployment
```

## Varsayılan Tech Stack

| Katman | Teknoloji | Gerekçe |
|--------|-----------|---------|
| Framework | Next.js (App Router) + TypeScript | Server/client rendering, dahili API route'ları |
| UI | Tailwind CSS + shadcn/ui | Hızlı geliştirme, tutarlı tasarım |
| Veritabanı | Supabase (PostgreSQL) | Auth + DB + Storage + Realtime tek çatıda |
| Auth | Google OAuth + Magic Link | Şifresiz, düşük sürtünmeli giriş |
| Ödeme | Stripe veya Lemon Squeezy | Bölgeye ve ihtiyaca göre seçim |
| E-posta | Resend + React Email | Modern API, tip güvenli şablonlar |
| Analytics | PostHog | Ürün analizi, feature flags, session replay |
| Hosting | Dokploy (self-hosted) | Ücretsiz, Docker tabanlı, kendi sunucunda deploy |

Her teknoloji keşif görüşmesinde kullanıcının ihtiyaçlarına göre değiştirilebilir.

## Tezgah vs. Alternatifler

| | Tezgah | create-t3-app | Shipfast / Supastarter |
|---|--------|---------------|----------------------|
| Ne | Claude Code skill seti (rehber) | Boilerplate kod üretici | Hazır SaaS şablonu |
| Nasıl | Claude Code ile adım adım inşa | Proje iskeleti oluşturur | Hazır kodu klonla |
| Dil | Türkçe | İngilizce | İngilizce |
| Fiyat | Ücretsiz (MIT) | Ücretsiz | $200-400 |
| Esneklik | Her karar tartışılır, değiştirilebilir | Sabit tech stack | Sabit mimari |
| Kapsam | DB → Auth → Ödeme → SEO → Yasal → Test → Deploy | Sadece proje iskeleti | Kod + bazı entegrasyonlar |
| Türkiye bağlamı | KVKK, Stripe/LS karşılaştırması, yerel ipuçları | Yok | Yok |

## Gereksinimler

- [Claude Code](https://claude.ai/claude-code) kurulu olmalı
- Node.js 18+

## Nasıl Çalışır

1. `npx tezgah init` ile skill dosyalarını projenize kopyalarsınız
2. Claude Code'da `/saas-launcher` ile orkestratörü başlatırsınız
3. Orkestratör size ürününüz hakkında sorular sorar (Faz 0)
4. Tech stack kararlarını birlikte verirsiniz (Faz 1)
5. Her katman ilgili uzman skill tarafından adım adım inşa edilir (Faz 2-11)

Tek bir skill'i bağımsız kullanmak da mümkündür:

```
/saas-auth          # Sadece auth sistemi kur
/saas-payments      # Sadece ödeme entegrasyonu ekle
/saas-database      # Sadece Supabase altyapısını kur
/saas-legal         # Sadece yasal sayfaları hazırla
```

## Katkıda Bulunma

Katkılarınızı bekliyoruz! [CONTRIBUTING.md](CONTRIBUTING.md) dosyasını okuyun.

Kısa özet:
- Yeni skill ekleyebilirsiniz
- Mevcut skill'leri iyileştirebilirsiniz
- Hata bildirimi veya özellik talebi açabilirsiniz
- Türkçe dışında dil desteği ekleyebilirsiniz

## Güvenlik

Güvenlik açığı bildirimi için [SECURITY.md](SECURITY.md) dosyasını okuyun.

## Lisans

[MIT](LICENSE) — Fatih Guner
