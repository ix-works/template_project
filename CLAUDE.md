# CLAUDE.md — template_project (ince proje loader'ı)

<!-- KESIN-YASAKLAR:BEGIN — kanonik: core/claude/kesin-yasaklar.canonical.md · ELLE DÜZENLEME (sync_yasaklar.py yeniden-damgalar) -->
```
████████████████████████████████████████████████████████████████
█  ⛔  KESİN YASAKLAR — BYPASS YOK, İSTİSNA YOK (ADR 0005)  ⛔  █
████████████████████████████████████████████████████████████████
```

| Kategori | Yasak |
|---|---|
| **A — Standart SAP objeleri** (Z/Y ile başlamayan) | Hiçbir şekilde yarat/değiştir/sil. Append struct, alan ekleme, FM/BAdI/program değişikliği, message class değişikliği = YASAK. Bunları yapan script çalıştırma da YASAK. **Append field/DTEL adını AI ÖNERMEZ — kullanıcı belirler, sonucu AI'a bildirir.** |
| **B — Standart tablo verileri** | Direkt `INSERT/UPDATE/DELETE/MODIFY` YASAK (Z'li programda yazdığın kod içinde bile). Sıralı arama: BAPI → RFC FM → transaction (BDC) → kullanıcıdan manuel. Asla direkt SQL. |
| **C — Sistem state** | Transport request yaratma, release etme YASAK. Package yaratma YASAK. Enqueue lock silme YASAK. |
| **D — Z'li obje yaratma** | Login dili = projenin **master_language**'idir (`project.yaml`). Tüm 4 field label (short/medium/long/heading) o dilde ve TAM yazılır. Title/description boş bırakılmaz. Activate öncesi REST GET ile doğrulanır. |

**Yapılması gerekiyorsa:** DUR → AÇIKLA → ÖNERİ SUN → KULLANICIDAN İSTE → BEKLE → DEVAM. "Küçük dokunuş" istisnası YOK.

> **🧭 ÇEKİRDEK DAVRANIŞ — lider + TÜM alt-ajanlar:** **TAHMİN YASAK = kanıtlı hareket et.** Yöntem/pattern/syntax/alan-adını mevcut artefakt + playbook/standard'dan doğrula, canlı teyit et; "activated/uploaded/çalıştı" mesajına güvenme; emin değilsen DUR → sor; DTEL/append adı önerme (kullanıcı verir).

📖 Detay: `core/governance/decisions/0005-sap-standart-obje-koruma-ve-sistem-state-yasaklari.md` · Bu blok her projenin kök `CLAUDE.md`'sine FİZİKSEL damgalıdır (junction'dan bağımsız daima yüklü); `check_kesin_yasaklar.py` guard'ı kanonikle eşliğini zorlar. Değişiklik: kanoniği düzenle → `sync_yasaklar.py` tüm projeleri yeniden damgalar.
<!-- KESIN-YASAKLAR:END -->

<!-- KESİN YASAKLAR bloğu init_project tarafından buraya FİZİKSEL damgalanır (junction-
     bağımsız daima yüklü). Aşağıdaki @import metodolojinin GERİ KALANINI yükler. -->

@core/CLAUDE.core.md

> Yukarıdaki import metodoloji çekirdeğini yükler (protokol, SORU 0, gate'ler). **Yasaklar
> yukarıda fiziksel damgalıdır — import'a bağlı değil** (junction kırılsa da anayasa yüklü;
> `check_kesin_yasaklar` guard'ı damganın kanonikle eşliğini zorlar).
> **Bu dosyada YALNIZ proje-özel bilgi durur.** Metodoloji buraya YAZILMAZ (SORU 0 → core).
> Not: Metodoloji dosyaları `core/` junction'ı altındadır; core dokümanlarındaki göreli
> yollar CORE köküne göredir. **Metodoloji araması DAİMA `path=core/` ile** (kök-Grep
> core'u görmez — D29).

> **⚠ BU PROJE BİR REFERANS İSKELETTİR.** `PROJECT_BOOTSTRAP.md` STEP 0–6'nın canlı
> provası olarak açıldı; gerçek bir SAP sistemine **bağlı değildir** (`.conn_adt` yok,
> MCP çağrıları çalışmaz). Yeni bir proje açarken buraya bakabilirsin — ama kopyalama:
> `python core/scripts/init_project.py` ile **üret**. Repo public'tir; buraya müşteri
> adı, SAP host/user, transport numarası **girmez**.

## PROJE KİMLİĞİ

- **Profil:** `project.yaml` → `sap_profile: s4_private` · `release: "2025"` ·
  `cleancore_policy: balanced` · `master_language: TR` · `source_root: SOURCE_CODES`
- **SAP bağlantı:** YOK (iskelet). Gerçek projede: proje kökünde `.conn_adt`
  (şablon: `core/claude/conn_adt.template`; çoklu-tier: `conn/` + `switch_tier.py`, ADR 0010).
  Dosya `.gitignore`'ludur — kimlik bilgisi repoya **hiçbir zaman** girmez.
- **Kaynak kod:** `SOURCE_CODES/<MODULE>/<PKG>/` (L4 kuralları: her pakette `.rules.md`)
- **Repo:** `ix-works/template_project` (`repo_mode: full`, public — şablon olduğu için)

## PROJE-ÖZEL DOSYA İNDEKSİ

| Konu | Dosya |
|---|---|
| Paket listesi (auto-generated) | `governance/package-registry.md` |
| Proje ADR'leri (`<PROJE>-NNN` serisi) | `governance/decisions/` |
| Ertelenmiş iş tetikleri | `governance/deferred-triggers.md` |
| Proje-özel pattern/standart overlay | `playbook-local/` · `standards-local/` |
| Proje-özel validator'lar | `scripts/validators-local/` |

## PROJE-ÖZEL KURALLAR / AKTİF İŞ KÜLTÜRÜ

<!-- Proje-özel gate'ler, dondurulmuş-kök notları (frozen_readonly_paths), aktif sprint
     kültürü, müşteri-özel kısıtlar BURAYA. Örnek satırlar silinip doldurulur. -->
