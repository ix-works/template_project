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

## PROJE KİMLİĞİ

- **Profil:** `project.yaml` → `sap_profile: <ecc|s4_private|s4_public|btp_abap>` ·
  `release: "<REL>"` · `master_language: <ML>` · `source_root: SOURCE_CODES`
- **SAP bağlantı:** `<PROJECT_ROOT>/.conn_adt` — Sistem: `<SYSTEM_ID>`, Client `<CLIENT>`,
  User: `<SAP_USER>`
- **Kaynak kod:** `SOURCE_CODES/<MODULE>/<PKG>/` (L4 kuralları: her pakette `.rules.md`)

## PROJE-ÖZEL DOSYA İNDEKSİ

| Konu | Dosya |
|---|---|
| Paket listesi (auto-generated) | `governance/package-registry.md` |
| Proje ADR'leri (`<PROJE>-NNN` serisi) | `governance/decisions/` |
| Ertelenmiş iş tetikleri | `governance/deferred-triggers.md` |
| Proje-özel pattern/standart overlay | `playbook-local/` · `standards-local/` |
| Proje-özel validator'lar | `scripts/validators-local/` |

## PROJE-ÖZEL KURALLAR / AKTİF İŞ KÜLTÜRÜ

<!-- Proje-özel gate'ler, dondurulmuş-kök notları (DİSİPLİN kuralı — runtime guard YOK;
     `frozen_readonly_paths` ölü anahtardır, yazma), aktif sprint kültürü, müşteri-özel
     kısıtlar BURAYA. Örnek satırlar silinip doldurulur. -->
