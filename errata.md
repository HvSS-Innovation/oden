# Errata (docs vs kod/test)

Datum: 2026-02-11

Det här dokumentet listar verifierade avvikelser mellan dokumentationen (krav), enhetstesterna och nuvarande implementation.

**Princip:** Dokumentationen i `docs/` behandlas som facit. Där kod/test avviker från docs noteras det som errata.

## Källor som jämförts

- Dokumentation: `docs/WEB_GUI.md`, `docs/FEATURES.md`, `docs/SETUP_FLOW.md`
- Kod: `oden/web_server.py`, `oden/web_handlers/*`, `oden/templates/web/js/dashboard/*`
- Tester: `tests/test_web_gui.py`, `tests/test_security.py`, `tests/test_processing.py`, `tests/test_formatting.py`

## Statuslegend

- **DOCS → KOD/TEST**: Implementation/test bör ändras för att matcha docs.
- **DOCS (intern)**: Dokumentationen är självmotsägande eller otydlig; kräver förtydligande innan kod/test kan bedömas.

## Avvikelsetabell

| ID | Område | Docs säger (facit) | Kod/test idag (verifierat) | Risk/impact | Rekommenderad åtgärd (ingen ändring gjord nu) |
| ---: | --- | --- | --- | --- | --- |
| 1 | Setup-mode routing | I setup-mode är *enbart setup-routes* tillgängliga och **alla andra anrop omdirigeras till `/setup`** (`docs/WEB_GUI.md`). | `create_app(setup_mode=True)` lägger endast en redirect för `/` → `/setup`. Övriga okända paths blir 404 (dashboard-routes är inte registrerade). | Låg–medel: docs och faktisk UX skiljer (404 vs redirect). | Anpassa setup-mode routing så att docs stämmer (t.ex. middleware/”catch-all” redirect) eller förtydliga docs (om redirect endast gäller `/`). |
| 2 | Setup endpoint-namn | Setup-tabellen anger bl.a. `POST /api/setup/set-home` och `POST /api/setup/save` (`docs/WEB_GUI.md`). | Faktiska routes är `POST /api/setup/oden-home` och `POST /api/setup/save-config` (se `oden/web_server.py`). | Medel: klienter som följer docs kommer anropa fel endpoints. | Antingen aliasa gamla/”doc”-paths i servern eller ändra docs så de matchar implementationen (kravbeslut behövs). |
| 3 | Setup endpoints tillgänglighet | ”Dessa är enbart tillgängliga i setup-mode.” (`docs/WEB_GUI.md`). | Setup-routes registreras *alltid* (även i dashboard-mode) i `oden/web_server.py`. I dashboard-mode gäller auth-middleware, men setup-endpoints (förutom `/api/setup/reset`) är inte listade som protected → kan nås utan token. | Hög: oavsiktlig åtkomst till setup-flödesoperationer i driftläge, potentiellt störande/oväntat beteende. | Lås ned setup-endpoints i dashboard-mode (returnera 404/405) eller skydda dem med auth + explicit “inte i detta läge”. |
| 4 | ~~Auth: config reset~~ | ~~`DELETE /api/config/reset` kräver auth ✅ (`docs/WEB_GUI.md`).~~ | **ÅTGÄRDAD**: `/api/config/reset` tillagd i `PROTECTED_ENDPOINTS`. Auth-tester skapade (401 utan token, !401/!404 med token). | — | — |
| 5 | Dashboard: INI-export | INI-export ska fungera men kräver auth ✅ (`docs/WEB_GUI.md`). | `oden/templates/web/js/dashboard/config.js` anropar `fetch('/api/config/export')` utan token. Koden kräver auth (protected). Tester visar att `/api/config/export` kräver auth (401 utan token). | Medel: GUI-funktionen “Ladda ner INI” riskerar ge 401 i praktiken. | Uppdatera dashboard-JS att skicka token (header eller `?token=`) för export. |
| 6 | Auth: template export | `/api/templates/{name}/export` och `/api/templates/export` är markerade Auth=Nej i docs (`docs/WEB_GUI.md`). | Auth-middleware skyddar alla paths som börjar med `/api/templates/` (`PROTECTED_PREFIXES` i `oden/web_server.py`) → export-endpoints kräver token. Enhetstester anropar export med auth-header. | Medel: docs säger “öppet”, implementation är “skyddat”; externa klienter kan få oväntade 401. | Om docs är krav: undanta export paths från prefix-skyddet (och justera tester därefter). Om skydd är avsiktligt: förtydliga docs (kravbeslut behövs). |
| 7 | Konfigurationsnyckel: whitelist | Docs använder `whitelisted_groups` (i `docs/WEB_GUI.md` och `docs/FEATURES.md`). | Kod, DB och GUI använder `whitelist_groups` (`oden/config_db.py`, `oden/web_handlers/*`, `oden/processing.py`, dashboard JS). | **Hög (migreringsrisk)**: att ändra nyckeln påverkar lagrad config, export/import och runtime-beteende. | Välj officiell nyckel. Om docs ska gälla: planera migration/compat (stöd båda en period, migrera DB, uppdatera export/import, lägg tester). Om kod ska gälla: uppdatera docs. |
| 8 | Setup-dokumentation: QR | Setup-flow beskriver QR-generering via `segno` (`docs/SETUP_FLOW.md`). | Koden använder `qrcode` och `qrcode.image.svg` (`oden/web_handlers/setup_handlers.py`). | Låg: teknisk detalj, men docs stämmer inte med implementation. | Förtydliga docs eller byt bibliotek (vanligen docs-fix, men kravbeslut behövs om docs är facit på detaljnivå). |
| 9 | INI-import läge | Docs listar `POST /api/config-file` som dashboard-endpoint (Auth=Nej). | `/api/config-file` registreras även i setup-mode (och dashboard-mode). Kommentar i `oden/web_server.py` säger “only available during setup”, men koden gör den tillgänglig i båda lägen. | Låg–medel: oklar produktpolicy (var får import ske?). | Besluta om policy (setup-only vs alltid). Justera kod/docs/test när krav är tydligt. |
| 10 | ideas.md: docs/test-analys | ideas.md innehåller flera påståenden om docs vs tester (t.ex. `++` fallback “kastas bort”, och endpoint-namn). | Nuvarande kod för `++` fallback faller vidare till ny fil (se `oden/processing.py`) och testerna är gröna (218 passed). Flera endpoint-namn i ideas.md matchar inte nuvarande docs. | Låg: påverkar inte runtime, men riskerar att styra fel prioritering. | Uppdatera ideas.md eller flytta “historiska observationer” till en separat sektion så den inte tolkas som aktuell sanning. |

## Noteringar

- Alla tester passerar (`247 passed`). Errata-listan är alltså **krav-/dokumentationsdriven**: den visar var implementationen inte följer docs (eller där docs behöver förtydligas innan man kan säga vad som är "rätt").
- Flera av punkterna ovan (särskilt #4 och #7) har tydlig säkerhets-/migreringsrisk och bör hanteras med kravbeslut och plan, inte via ad-hoc ändringar.
