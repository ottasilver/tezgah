# Changelog

Bu dosya [Keep a Changelog](https://keepachangelog.com/tr/1.1.0/) formatını takip eder
ve proje [Semantic Versioning](https://semver.org/lang/tr/) kullanır.

## [1.0.0] - 2026-03-22

### Eklendi

**Skill'ler (12 adet):**
- **saas-launcher** — Orkestratör skill: keşif görüşmesi, tech stack kararları, faz yönetimi
- **saas-database** — Supabase veritabanı: proje kurulumu, şema tasarımı, RLS, migration, connection pooling
- **saas-auth** — Kimlik doğrulama: Google OAuth, Magic Link, JWT, korumalı route'lar
- **saas-payments** — Ödeme sistemi: Stripe/Lemon Squeezy, checkout, webhook, abonelik yönetimi
- **saas-email** — E-posta altyapısı: Resend/Mailgun, DNS (SPF/DKIM/DMARC), şablonlar
- **saas-storage** — Dosya depolama: Supabase Storage, yükleme, RLS güvenliği, görsel optimizasyonu
- **saas-landing-seo** — Landing page ve SEO: bileşenler, metadata, sitemap, Open Graph, blog
- **saas-legal** — Yasal uyumluluk: KVKK/GDPR, gizlilik politikası, çerez onayı
- **saas-api-security** — API güvenliği: rate limiting, plan bazlı erişim, input validation
- **saas-testing** — Test stratejisi: Vitest, Playwright E2E, webhook testi, CI entegrasyonu
- **saas-analytics** — Ürün analizi: PostHog, event tracking, feature flags, session replay
- **saas-deployment** — Deployment: Dokploy/Railway/Fly.io, domain, SSL, CI/CD, izleme

**CLI aracı:**
- `tezgah init` — tüm skill'leri kur
- `tezgah add <skill>` — tek skill ekle
- `tezgah remove <skill>` — skill kaldır
- `tezgah update` — kurulu skill'leri güncelle
- `tezgah doctor` — kurulumu kontrol et
- `tezgah list` — skill'leri listele
- Fuzzy matching ile yazım hatası önerisi
- Terminal renk desteği algılama (NO_COLOR uyumlu)

**Altyapı:**
- GitHub Actions CI (Node 18/20/22)
- Otomatik npm release workflow (tag bazlı)
- ESLint kod kalitesi
- Otomatik CLI testleri
- Issue ve PR şablonları
- Güvenlik politikası (SECURITY.md)

[1.0.0]: https://github.com/komunite/tezgah/releases/tag/v1.0.0
