# template_project — DEV_CORE tabanlı proje iskeleti

Bu repo, [`ix-works/DEV_CORE`](https://github.com/ix-works/DEV_CORE) metodoloji
çekirdeğine bağlı **yeni bir SAP/ABAP projesinin nasıl göründüğünü** gösterir.

> **⚠ Bu bir referans iskelettir.** Gerçek bir SAP sistemine bağlı değildir
> (`.conn_adt` yok, MCP çağrıları çalışmaz). İçinde müşteri verisi, sistem adı,
> transport numarası **yoktur** ve olmamalıdır — repo public'tir.

---

## Temel ilke: EDİNME, BAK

Yeni proje metodolojiyi **kopyalamaz**. `core/` adında bir junction ile DEV_CORE'a
**bakar** (ADR 0020). Diskte metodolojinin **tek** kopyası vardır: `C:\IX\DEV_CORE`.

```
C:\IX\DEV_CORE\              ← metodoloji (tek fiziksel kopya)
C:\IX\template_project\
  ├── core\            ══► C:\IX\DEV_CORE              (junction)
  ├── .claude\agents\  ══► C:\IX\DEV_CORE\claude\agents
  ├── .claude\skills\  ══► C:\IX\DEV_CORE\claude\skills
  └── .claude\commands\══► C:\IX\DEV_CORE\claude\commands
```

Sonuç: core'a düşen bir düzeltme **tüm projelere anında** yansır. Bu repoda
metodolojiden tek satır yoktur — `.gitignore` bunu zorlar, CI (`guard.yml`) de
sunucu tarafında doğrular.

**"Skill/agent klasörde nasıl oluşacak?" — OLUŞMAZ.** Junction sayesinde proje
içinden `core\scripts\...` ve `.claude\skills\...` olarak *görünürler*.

---

## DEV_CORE ↔ proje: hangi bilgi nerede yaşar

İki repo birbirine bağlıdır ama **sorumlulukları keskin biçimde ayrıdır**. Bir şeyi
yanlış tarafa yazmak, ya diğer projeleri kirletir ya da bu projede kaybolur.

| Soru | Cevap | Nereye |
|---|---|---|
| Bir **pattern / ADT dersi** öğrendim | Her projede geçerli | **DEV_CORE** → `playbook/` |
| Bir **kural** koydum, atlanamaz olmalı | Her projede geçerli | **DEV_CORE** → `standards/` + bir **gate** (validator/hook) |
| **Mimari karar** aldım (metodoloji) | Her projede geçerli | **DEV_CORE** → `governance/decisions/` (ADR) |
| Bir **script** yazdım, tekrar lazım olacak | Her projede geçerli | **DEV_CORE** → `scripts/` + playbook referansı |
| **Bu SAP sistemi**, bu müşteri, bu paket | Yalnız bu proje | **Proje** → `project.yaml`, `.conn_adt`, `CLAUDE.md` |
| Bir **iş kuralı** / sprint kararı | Yalnız bu proje | **Proje** → `governance/` |
| Bu **pakete özel** istisna | Yalnız bu paket | **Proje** → `SOURCE_CODES/<MOD>/<PKG>/.rules.md` |
| Core'daki bir kuralı bu projede **daraltmak** | Yalnız bu proje | **Proje** → `playbook-local/` · `standards-local/` · `scripts/validators-local/` |

Karar ağacı (`CLAUDE.core.md` §4, "SORU 0"):

```
Bu bilgi metodoloji mi, projeye mi özel?
  ├─ Metodoloji → DEV_CORE'a yaz. Yazarken:
  │     • genericize et  (proje/müşteri/sistem izi → placeholder)
  │     • link kuralı    (core-içi link core-göreli; core→proje link YASAK)
  │     • profil etiketi (applies_to: hangi SAP profilleri?)
  └─ Proje işi → bu repoya (tek paket mi, tüm proje mi?)
```

### Genericize kuralı — core'a ne giremez

DEV_CORE **public**tir ve her projeye bakar. Bu yüzden `pre_tool_guard` şunları
core'a yazılmaktan **bloklar**: müşteri/firma adı, SAP host adı, SAP kullanıcı adı,
transport numarası, kişisel handle. Core'a yazarken `<SAP_HOST>`, `<PROJECT_NAME>`,
`<TRANSPORT>` gibi placeholder'lar kullanılır; `ZSD001` demo paketi bilinçli istisnadır.

### Değişiklik hangi yöne akar

```
        ┌──────────────────────────────────────────┐
        │  DEV_CORE (public)                       │
        │  playbook · standards · gate'ler · ADR   │
        └───────┬──────────────────────────────────┘
                │ junction (salt-görünüm; proje ASLA core'a yazmaz)
        ┌───────▼──────────┐   ┌──────────────────┐   ┌─────────────┐
        │ template_project │   │ <müşteri projesi>│   │ <diğer>     │
        └──────────────────┘   └──────────────────┘   └─────────────┘
                │                       │
                └───────┬───────────────┘
                        │  T1–T11 tetikleri: projede öğrenilen
                        │  metodoloji dersi → core'a **PR** ile geri döner
                        ▼
                   DEV_CORE PR → review + CODEOWNERS → merge
                        │
                        ▼  makinede tek `git -C C:\IX\DEV_CORE pull`
                   TÜM projeler aynı anda güncellenir
```

Kritik nokta: **proje core'a doğrudan yazmaz.** Junction salt-görünümdür; core'daki bir
dosyayı düzenlemek DEV_CORE reposunda bir PR açmak demektir. Projenin `.gitignore`'u ve
CI'daki `core-leak` job'ı, core içeriğinin proje reposuna sızmasını ayrıca engeller.

### Core'a düzeltme göndermek

```powershell
cd C:\IX\DEV_CORE                       # junction'ın gerçek hedefi
git checkout -b fix/<konu>
# ... düzelt (genericize et!) ...
python scripts/validators/run_all_validators.py
git commit -m "fix(core): ..." ; git push -u origin fix/<konu>
gh pr create
```

PR merge edilince `git -C C:\IX\DEV_CORE pull` ile **bütün projeler** düzelmiş olur.
Bu repodaki "Bilinen sapmalar" listesi tam olarak böyle doğdu.

---

## Bu repoda ne var

| Yol | Ne |
|---|---|
| `CLAUDE.md` | İnce loader. Üstünde **KESİN YASAKLAR** fiziksel damgası (ADR 0005/0021 — junction kırılsa bile anayasa yüklü), altında `@core/CLAUDE.core.md` import'u |
| `project.yaml` | Proje kimliği: `sap_profile`, `release`, `master_language`, `source_root`. **Core script'leri buradan okur** — hard-code yok |
| `.claude/settings.json` | Hook kayıtları; hepsi `scripts/hook_shim.py` üzerinden core'a gider |
| `scripts/hook_shim.py` | Hook köprüsü (junction kopuksa net onarım mesajı verir) |
| `.mcp.json` | SAP ADT MCP server'ı core'dan yükler; bağlantı proje kökündeki `.conn_adt`'den |
| `.gitignore` | **Sızıntı kilidi**: `/core/`, `.claude/{agents,skills,commands}/`, `.conn_adt` |
| `.github/workflows/guard.yml` | CI: core-sızıntı ağı + davranış-yüzeyi çevre duvarı |
| `.github/CODEOWNERS` | Davranış-yüzeyi ve gate'ler için code-owner onayı |
| `SOURCE_CODES/<MOD>/<PKG>/` | SAP kaynak kodu. Her pakette `.rules.md` (L4 kuralları) |
| `governance/` | Proje ADR'leri, `deferred-triggers.md`, paket kaydı |
| `playbook-local/` · `standards-local/` · `scripts/validators-local/` | Proje-özel overlay'ler (core'u ezmez, tamamlar) |

Örnek paket: `SOURCE_CODES/SD/ZSD001_CLC/` — `.rules.md`, `SPEC.md`, `SESSION_NOTES.md`.

---

## Yeni proje nasıl açılır

**Bu repoyu klonlayıp içini boşaltma.** Kanonik yol `PROJECT_BOOTSTRAP.md` STEP 0–6:

```powershell
# STEP 0 — kararlar (kod yok): repo_mode? SAP profili+release+master_language?
#          SAP sistemi? source_root? paket prefix'leri?

# STEP 1 — repo + klasör (yalnız repo_mode=full)
gh repo create <ORG>/XYZ --private
git clone https://github.com/<ORG>/XYZ.git C:\IX\XYZ

# STEP 2 — iskeleti ÜRET (kopyalama!)
python C:\IX\DEV_CORE\scripts\init_project.py C:\IX\XYZ --name XYZ --repo-mode full

# STEP 3 — junction'lar + memory seed + bağımlılıklar
python C:\IX\DEV_CORE\scripts\team_setup.py --project C:\IX\XYZ

# STEP 4 — proje değerlerini doldur
#   project.yaml  → sap_profile / release / master_language / (cleancore_policy)
#   .conn_adt     → SAP host/client/user   (şablon: core/claude/conn_adt.template)
#   CLAUDE.md     → proje kimliği bölümü

# STEP 5 — KABUL GATE'İ (geçmeden iş yapılmaz) — aşağıya bak

# STEP 6 — ilk paket + ilk commit
python core/scripts/bootstrap_package.py ZSD001_CLC --module SD --title "..." \
       --templates-root core/templates/new-package     # ⚠ bkz. Bilinen sapmalar #5
git add -A ; git commit -m "chore(bootstrap): XYZ proje iskeleti" ; git push -u origin main
```

`repo_mode=local` (yalnız git init) veya `none` (git'siz) seçilirse STEP 1 atlanır;
junction/memory/profil mekanizmalarının hiçbiri git'e bağımlı değildir.

---

## STEP 5 — Kabul gate'i

| # | Kanıt | Nasıl |
|---|---|---|
| 1 | Loader + hook'lar çalışıyor | Projede oturum aç → **ekran teyidi formatı** geliyor |
| 2 | MCP kendi sistemine bağlı | `ping` + read-only `adt_get` → **projenin** SAP sistemi |
| 3 | Validator'lar PASS | `python core/scripts/validators/run_all_validators.py` |
| 4 | Sızıntı kilidi çalışıyor | `git ls-files core/ .claude/agents` → **boş** |
| 5 | Kurulum sağlığı | `python core/scripts/ix_doctor.py` → FAIL yok |

Bu iskelette **madde 2 bilinçli olarak SKIP** edilmiştir (SAP bağlantısı yok).

---

## Bilinen sapmalar (bu prova sırasında bulundu, 2026-07-09)

Bu repo `PROJECT_BOOTSTRAP.md`'nin **ilk canlı provası** olarak açıldı. El kitabı ile
script'ler arasında beş boşluk çıktı. Hepsi core'a bildirildi; düzeltilene kadar
aşağıdaki telafileri uygula:

| # | Sapma | Telafi |
|---|---|---|
| 1 | `init_project` **CI workflow + CODEOWNERS üretmiyor** (el kitabı STEP 1 "kurulacak" diyor) | Bu repodaki `.github/` içeriğini örnek al |
| 2 | Bootstrap'ta **`behavior_manifest generate` adımı yok** → `ix_doctor` K4 daima FAIL | `python core/scripts/behavior_manifest.py generate` |
| 3 | **STEP 5, STEP 6'dan önce geçilemez** — `ix_doctor` commit/push/ruleset arıyor, henüz yok (K2/K3 FAIL) | Kabul gate'ini STEP 6 sonrası koş |
| 4 | `ix_doctor` **`.conn_adt` yoksa FAIL** verir — SAP'siz/LITE proje gate'ten geçemez | K5 FAIL'ini gerekçeli kabul et |
| 5 | **`bootstrap_package.py` kırık**: `--templates-root` varsayılanı `templates/new-package` **cwd-göreli** → proje kökünden çalışmaz | `--templates-root core/templates/new-package` ver |
| 6 | `bootstrap_package.py`, `--owner` verilmezse **`git config user.name`'i dosyalara gömer** → public/şablon repoda kimlik sızıntısı | `--owner "<OWNER>"` ver |

Sapma #5, bugün core'da kapatılan **CORE-01** ailesinin kardeşidir: *core script'i,
core-içi bir kaynağı proje dizininde arıyor.* `CORE-01` gate'i bunu yakalamaz çünkü
`__file__` değil **cwd** kullanılıyor.

Sapma #6 bu repoda canlı yakalandı: ilk üretimde paket dosyalarına gerçek kullanıcı
adı yazıldı, push öncesi iz taramasında görüldü ve paket placeholder'la yeniden
üretildi. Public bir repoda `git config user.name` sessizce kimlik sızdırabilir.

---

## Günlük çalışma

```powershell
python core/scripts/validators/run_all_validators.py   # tüm gate'ler
python core/scripts/ix_doctor.py                       # 7-katman kurulum sağlığı
python core/scripts/team_setup.py --repair-junctions   # junction koptuysa
```

`main`'e doğrudan push kapalıdır (ruleset). Her değişiklik: kısa branch → PR → CI →
code-owner onayı. Ayrıntı: `core/AGENTS.md` §1.

## Sorun giderme

| Belirti | Çözüm |
|---|---|
| Hook "CORE JUNCTION KOPUK" diyor | `python C:\IX\DEV_CORE\scripts\team_setup.py --repair-junctions` |
| Ekran teyidi gelmiyor | `CLAUDE.md`'deki `@core/CLAUDE.core.md` import'u + `core` junction'ı |
| MCP yanlış sisteme bağlı | Proje kökündeki `.conn_adt` (env `ADT_SAP_*` override eder) |
| Validator "CORE-modu" diyor | `project.yaml` proje kökünde mi + `sap_profile` dolu mu |

---

**Kanonik el kitabı:** [`core/PROJECT_BOOTSTRAP.md`](https://github.com/ix-works/DEV_CORE/blob/main/PROJECT_BOOTSTRAP.md)
· **Mimari karar:** ADR 0020 (çoklu-proje / junction'lı çekirdek)
