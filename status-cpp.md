# Status — Studio C++

Mappa dei concetti del percorso. Questo file è la memoria di Claude tra le sessioni: elenca, non spiega. La rielaborazione dei contenuti vive nei quaderni di Valentina.

Gli handoff discorsivi delle singole sessioni stanno in `sessioni/` (un file per sessione, nome = timestamp UTC).

## Concetti acquisiti
- Puntatori — lvalue/rvalue col test operativo (a sinistra di `=` / prendibile con `&`), non l'euristica contenitore/contenuto: `*ptr` lvalue, `&ptr` rvalue
- Puntatori — tipo = numero di livelli di indirezione; `new` deve combaciare col numero di stelle
- Puntatori — dimensione (8 byte su x64, sempre) vs tipo (governa la deref); `sizeof` di un'espressione = dimensione del suo tipo
- Handle opaco: `DECLARE_HANDLE` / `STRICT`, una struct fantasma per ogni *tipo* di handle (mai istanziata) al solo scopo di type-checking a compile-time; senza `STRICT` tutti `void*` e intercambiabili
- Il framework come codice e memoria, non solo come contratto: driver UMDF = DLL caricata in `WUDFHost.exe` insieme a framework e IddCx, un solo spazio di indirizzamento; l'handle punta a una struct di cui non hai l'header (incapsulamento in C + compatibilità binaria)
- `delete p` libera l'oggetto puntato, non azzera `p` (da cui `p = nullptr` esplicito)
- Possesso ≠ possesso dell'indirizzo: dello stesso indirizzo esistono più copie, proprietario è chi ha il *dovere* di liberarlo
- Cascata dei distruttori: si ferma sul primo puntatore grezzo (distruggere 8 byte non tocca l'altro capo)
- Assenza di `delete` = ambigua: o dimenticanza, o non-possesso; il distruttore scritto per esteso che *non* nomina un membro è una dichiarazione di non-possesso
- Da re-interrogare periodicamente su richiesta di Valentina (non ancora consolidati al 100%: doppi puntatori, distinzione dato/indirizzo sotto pressione)

## Concetti in corso
- Ownership transfer via puntatore consumato e azzerato dalla callee (visto su `pDeviceInit` in `WdfDeviceCreate`, VDD)
- Vita agganciata: oggetto C++ (`new` in `DeviceAdd`) la cui `delete` sta in `Cleanup()`, chiamata dal framework via `EvtCleanupCallback` prima di distruggere l'oggetto proprietario — `new` e `delete` in funzioni lontane, invocate da soggetti diversi
- Contesto WDF allocato *dentro* il blocco dell'oggetto device (`WDF_OBJECT_ATTRIBUTES_INIT_CONTEXT_TYPE` + `WdfDeviceCreate`); `WdfObjectGet_…` calcola un offset, non alloca
- Versioning a runtime vs compile-time (`IDD_IS_FIELD_AVAILABLE`, VDD) — corretta un'interpretazione iniziale errata (protezione memoria)

## Parking lot
*(tangenti parcheggiate: una riga di contesto ciascuna — perché è emersa, dove potrà rientrare)*

- `operator new` non ritorna `nullptr` su fallimento (lancia `std::bad_alloc`) — aggancia il difetto `C6011` su `new IndirectDeviceContext(Device)` nel driver e una misconception negli appunti di Valentina; chiudere al ritorno sul driver
- `~IndirectDeviceContext`: `lock_guard`, `mutex`, `swap` di una mappa, `unique_ptr` — visti di sfuggita leggendo il distruttore, tutti da aprire quando il percorso li incontra
- Lambda: vista meccanicamente (funzione senza nome scritta sul posto e assegnata a un campo, confrontata con `PnpPowerCallbacks.EvtDeviceD0Entry`); cattura e closure non toccate
- `IDD_IS_FIELD_AVAILABLE` — versioning runtime del framework; ramo `if` (HDR / callback `…2`) è quello vivo sulla VM (Win11 + WDK recente), ramo `else` è codice morto per lei
- Dove viene chiamata `InitAdapter` (non in `DeviceAdd`) — prossimo anello del ciclo di vita
- Doppia `RegCloseKey` in `EnabledQuery` (VDD) — difetto locale trovato, potenzialmente proponibile upstream
- `RegQueryValueExW` senza terminatore null garantito in `initpath` (VDD) — bufferover-read; famiglia `C6387`
- fat pointer (puntatori a funzione membro > 8 byte) — solo se emerge nel codice
- Guida alla creazione dei certificati di test — richiesta esplicita di Valentina, rimandata

## Materiale attivo
- **Virtual Display Driver** (VirtualDrivers/Virtual-Display-Driver, branch `master`): lettura guidata del ciclo di vita, un anello alla volta. `VirtualDisplayDriverDeviceAdd` (2812-2947) letta e chiusa: configurazione → `WdfDeviceCreate` → contesto agganciato. Struttura reale: file unico `MttVDD/Driver.cpp` (~5000 righe), non una god-class ma un god-file con ~60 globali; `IndirectDeviceContext` è ~330 righe (3653-3988). Driver **user mode (UMDF)**, non kernel. Pipeline funzionante (VS 2026 + WDK + VM Win11 test-signing); cert `MttVDD Test Cert` (99F8…) allineato host/VM e ora fissato nel progetto.
- **Calcolatrice** (progetto personale ground-up): da portare in questo repo dal desktop.

## Menù possibile
*(possibilità, non impegni — nessun ordine, nessuna priorità)*

- VDD: **giro top-down sull'architettura dei driver Windows** — come è strutturato un driver, chi lo carica, chi chiama cosa, usando questo codice come esempio concreto (richiesta esplicita di Valentina: le manca il quadro d'insieme, l'analisi bottom-up ne soffre)
- VDD: `DriverEntry` (riga 2405) — chi chiama `DeviceAdd` e come gliela si registra
- VDD: `InitAdapter` — cosa fa il device una volta acceso
- VDD: chiudere i difetti parcheggiati (`new`/`C6011`, `RegCloseKey`, `RegQueryValueExW`/`C6387`)
- Puntatori: giro di ripasso a sorpresa (richiesto da Valentina)
- Portare la calcolatrice nel repo e fare il punto sul suo stato
- Guida alla creazione dei certificati di test
