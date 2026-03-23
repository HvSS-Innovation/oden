# Ideas

## Refresh groups button

Add a manual "Uppdatera" button in the groups tab that re-fetches group data
from signal-cli via `log_groups()` and updates `app_state.groups`. This way
newly joined groups appear in the web GUI without restarting Oden.

The startup task `log_groups` already does this on boot — the new endpoint
would reuse that function. The handler needs access to `writer` from
`app_state`, and should return a clear error if signal-cli isn't connected.

---

## Dokumentation vs tester — avvikelser

Jämförelse mellan `docs/FEATURES.md`, `docs/WEB_GUI.md` och enhetstesterna. Tabellen listar fall där dokumentationen och testerna inte stämmer överens, eller där dokumentation saknar testning.

### API-endpoint-namn skiljer sig

Dokumentationen (WEB_GUI.md) använder andra endpoint-sökvägar än vad testerna faktiskt testar:

| Funktion | Dokumenterat i WEB_GUI.md | Testat i test_web_gui.py | Faktiskt i koden | Bedömning | Åtgärd |
|----------|---------------------------|--------------------------|-------------------|-----------|--------|
| Spara config | `POST /api/config` | `POST /api/config-save` | `POST /api/config-save` | ❌ Docs fel | Uppdatera docs |
| Gå med i grupp | `POST /api/groups/join` | `POST /api/join-group` | `POST /api/join-group` | ❌ Docs fel | Uppdatera docs |
| Toggla ignorera | `POST /api/groups/ignore` | `POST /api/toggle-ignore-group` | `POST /api/toggle-ignore-group` | ❌ Docs fel | Uppdatera docs |
| Toggla whitelist | `POST /api/groups/whitelist` | `POST /api/toggle-whitelist-group` | `POST /api/toggle-whitelist-group` | ❌ Docs fel | Uppdatera docs |
| Hämta mall | `GET /api/templates/{name}` | `GET /api/templates/report.md.j2` (med auth) | `GET /api/templates/{name}` | ✅ Sökväg OK, auth-krav saknas | Uppdatera auth i docs |
| Skapa autosvar | `POST /api/responses` | `POST /api/responses/new` | `POST /api/responses/new` | ❌ Docs fel | Uppdatera docs |

### Auth-krav skiljer sig

Auth-middleware i `web_server.py` definierar `PROTECTED_ENDPOINTS` (exakta sökvägar) och `PROTECTED_PREFIXES` (`/api/responses/`, `/api/templates/`). Prefix-matchning innebär att alla sökvägar som *börjar med* prefixet kräver auth.

| Endpoint | Docs: Auth | Test: Auth | Faktiskt i koden | Bedömning | Åtgärd |
|----------|------------|------------|------------------|-----------|--------|
| `GET /api/templates/{name}` | Nej | ✅ Ja (401) | ✅ Ja — prefix `/api/templates/` | ❌ Docs fel | Uppdatera docs: Auth = ✅ |
| `GET /api/templates` (lista) | Nej | Ej testat | ❌ Nej — matchar inte prefix | ✅ Docs OK | Ingen åtgärd |
| `GET /api/responses` (lista) | Nej | Ej testat | ❌ Nej — matchar inte prefix | ✅ Docs OK | Ingen åtgärd |
| `GET /api/responses/{id}` | Nej | Ej testat | ✅ Ja — prefix `/api/responses/` | ❌ Docs fel | Uppdatera docs: Auth = ✅ |
| `POST /api/config-file` (INI-import) | Nej | Ej testat | ❌ Nej | ✅ Docs OK | Ingen åtgärd |
| `DELETE /api/config/reset` | Nej | Ej testat | ✅ Ja — `PROTECTED_ENDPOINTS` | ✅ **Åtgärdad** | Docs uppdaterade: Auth = ✅. Test skapade. |

### Filnamnsformat — namnkonvention

| Funktion | Dokumenterat i FEATURES.md | Testat i test_formatting.py | Faktiskt i koden (`formatting.py`) | Bedömning | Åtgärd |
|----------|----------------------------|----------------------------|-------------------------------------|-----------|--------|
| Timestamp-only format | `timestamp_only` | `tnr` | `if filename_format == "tnr":` | ❌ Docs fel | Uppdatera docs: `tnr` |
| Timestamp-name format | `timestamp_name` | `tnr-name` | `if filename_format == "tnr-name":` | ❌ Docs fel | Uppdatera docs: `tnr-name` |

### Funktioner som saknar testning

| Funktion (dokumenterad) | Teststatus | Notering | Åtgärd |
|-------------------------|------------|----------|--------|
| Setup-wizard (SETUP_FLOW.md) | Grundläggande | Inga tester för `start-link`, `save`, `verify-code`, `install-obsidian-template` | Framtida testning |
| INI-export | Enbart auth-test | Verifierar 200, men inte filinnehåll | Framtida testning |
| INI-import | Ej testat | Inget test alls för `POST /api/config-file` | Framtida testning |
| Template preview | Ej testat | Inget test för `POST /api/templates/{name}/preview` | Framtida testning |
| Template reset | Ej testat | Inget test för `POST /api/templates/{name}/reset` | Framtida testning |
| Template export | Ej testat | Inget test för export-endpoints | Framtida testning |
| Shutdown-knapp | Enbart auth-test | Verifierar 401, inte att nedstängning sker | Framtida testning |
| System tray | Ej testat | Ingen test för `tray.py` alls | Framtida testning |
| Platsextraktion: Apple Maps `ll`-param | ✅ Testat | `test_apple_maps_ll_param` + `test_apple_maps_ll_with_label` | Ingen åtgärd |

### Append-fallback vid `++` — beteendeskillnad

| Källa | Beskrivet beteende | Faktiskt i koden (`processing.py`) | Åtgärd |
|-------|--------------------|-------------------------------------|--------|
| **FEATURES.md** | "Om ingen nylig fil hittas, behandlas meddelandet som nytt (utan `++`-prefixet)" | — | ✅ Docs är korrekt krav |
| **Test** `test_process_message_append_plus_plus_failure` | Verifierar att `mock_open.assert_not_called()` — inget skrivs, loggar "APPEND FAILED" | — | ❌ Fixa test — ska verifiera ny fil |
| **Kod** | — | `++`: Loggar "APPEND FAILED", strippar `++`, men returnerar (`if is_plus_plus_append: return`). Meddelandet **kastas bort**. | ❌ Fixa kod — ta bort early return så `++` faller vidare till ny fil |
| **Reply-append** | — | Loggar "APPEND FAILED", behåller citat, faller vidare till ny fil. Fungerar korrekt. | Ingen åtgärd |
| **Bedömning** | | ⚠️ **Kod och test är fel** — `++` ska falla vidare till ny fil precis som reply-append gör. | Fixa `processing.py` + `test_processing.py` |
