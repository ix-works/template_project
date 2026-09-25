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
| **B — Standart tablo verileri** | Direkt `INSERT/UPDATE/DELETE/MODIFY` YASAK (Z'li programda yazdığın kod içinde bile). Sıralı arama: released API (RAP BO/EML · released BAPI · released OData) → BAPI → RFC FM → transaction (BDC) → kullanıcıdan manuel (ayrıntı: `core/standards/10-standart-veriye-yazma-api-secimi.md`). Asla direkt SQL. |
| **C — Sistem state** | Transport request yaratma, release etme YASAK. Package yaratma YASAK. Enqueue lock silme YASAK. |
| **D — Z'li obje yaratma** | Login dili = projenin **master_language**'idir (`project.yaml`). Tüm 4 field label (short/medium/long/heading) o dilde ve TAM yazılır. Title/description boş bırakılmaz. Activate öncesi REST GET ile doğrulanır. |

**Yapılması gerekiyorsa:** DUR → AÇIKLA → ÖNERİ SUN → KULLANICIDAN İSTE → BEKLE → DEVAM. "Küçük dokunuş" istisnası YOK.

> **🧭 ÇEKİRDEK DAVRANIŞ — lider + TÜM alt-ajanlar:** **TAHMİN YASAK = kanıtlı hareket et.** Yöntem/pattern/syntax/alan-adını mevcut artefakt + playbook/standard'dan doğrula, canlı teyit et; "activated/uploaded/çalıştı" mesajına güvenme; emin değilsen DUR → sor; DTEL/append adı önerme (kullanıcı verir).

📖 Detay: `core/governance/decisions/0005-sap-standart-obje-koruma-ve-sistem-state-yasaklari.md` · Bu blok her projenin kök `CLAUDE.md`'sine FİZİKSEL damgalıdır (junction'dan bağımsız daima yüklü); `check_kesin_yasaklar.py` guard'ı kanonikle eşliğini zorlar. Değişiklik: kanoniği düzenle → `sync_yasaklar.py` tüm projeleri yeniden damgalar.
<!-- KESIN-YASAKLAR:END -->

<!-- KESİN YASAKLAR bloğu init_project tarafından buraya FİZİKSEL damgalanır (junction-
     bağımsız daima yüklü). Metodoloji çekirdeği bu dosyadan import EDİLMEZ — aşağıya bak. -->

> **Metodoloji çekirdeği (protokol, SORU 0, gate'ler) bu dosyadan `@import` ile YÜKLENMEZ**
> (Q286, 2026-09-12): `core/` junction'ının ardındaki dosya harness için DIŞ import'tur ve
> onaysız sessizce atlanır. Çekirdek, `team_setup.py`'nin ürettiği **fiziksel kopya**
> `.claude/rules/00-claude-core.md` olarak her oturum yüklenir (`paths:` yok). Bu dosyaya
> `@core/...` satırı EKLEME. Yükleme durumunu `session_start`'ın `[YUKLEME — session_start]`
> satırı söyler — "yüklendi" diye kendin beyan etme, o satırı aktar.
> **Yasaklar yukarıda fiziksel damgalıdır — import'a bağlı değil** (`check_kesin_yasaklar`
> guard'ı damganın kanonikle eşliğini zorlar).
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

# Compact instructions

Varsayılan özet bölümlerini KORU; aşağıdakileri onların içine ekle.

Öncelik sırasıyla KORU — kaybolursa geri getirilemez:
1. Yarım kalan SAP işlemi: hangi obje, push edildi mi, aktive edildi mi,
   transport / kilit / ATC durumu.
2. Bu oturumda ÖLÇÜLEN sonuçlar: sayı + birimi + kaynağı (`dosya:satır` ya da
   çalıştırılan komut). Niteleyiciyi DÜŞÜRME — "alt kırılımda boş" ≠ "hepsinde boş".
3. Alt ajanların döndürdüğü raporlar ve kullanıcının AskUserQuestion cevapları —
   özetleme, aynen taşı.
4. Değiştirilen dosyaların listesi + o değişikliği doğrulayan komut
   (validator / ATC / test) ve sonucu.
5. Verilen kararlar + GEREKÇESİ; açık kalan sorular; denenip çalışmayan yollar ve nedeni.
6. Aktif paket adı; koşan alt ajan varsa hangisi ve ne görev verildiği.

Emin olmadığın bir şeyi kesinmiş gibi yazma: "DOĞRULANMADI" diye etiketle.

Özete ALMA — compact sonrası zaten geri geliyor: CLAUDE.md kuralları ve yasaklar,
çekirdek kopyası `.claude/rules/00-claude-core.md` (ölçüldü: compact sonrası
`load_reason=compact` ile yeniden yüklenir — print modu, tek ölçüm), hook / system-reminder
çıktıları, skill ve araç listeleri.

⚠ `paths:`'li kurallar compact'ta yeniden yüklenMEZ; eşleşen bir dosya yeniden okununca
geri gelir. Yarım iş böyle bir kurala dayanıyorsa özete kuralın ADINI ve tetikleyen dosyayı al.
