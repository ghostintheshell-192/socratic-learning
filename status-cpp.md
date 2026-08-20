# Status — Studio C++

Mappa dei concetti del percorso. Questo file è la memoria di Claude tra le sessioni: elenca, non spiega. La rielaborazione dei contenuti vive nei quaderni di Valentina.

Gli handoff discorsivi delle singole sessioni stanno in `sessioni/` (un file per sessione, nome = timestamp UTC).

## Concetti acquisiti
- Puntatori — lvalue/rvalue col test operativo (a sinistra di `=` / prendibile con `&`), non l'euristica contenitore/contenuto: `*ptr` lvalue, `&ptr` rvalue
- Puntatori — tipo = numero di livelli di indirezione; `new` deve combaciare col numero di stelle
- Puntatori — dimensione (8 byte su x64, sempre) vs tipo (governa la deref); `sizeof` di un'espressione = dimensione del suo tipo
- Da re-interrogare periodicamente su richiesta di Valentina (non ancora consolidati al 100%: doppi puntatori, distinzione dato/indirizzo sotto pressione)

## Concetti in corso
- Ownership transfer via puntatore consumato e azzerato dalla callee (visto su `pDeviceInit` in `WdfDeviceCreate`, VDD)
- Versioning a runtime vs compile-time (`IDD_IS_FIELD_AVAILABLE`, VDD) — corretta un'interpretazione iniziale errata (protezione memoria)

## Parking lot
*(tangenti parcheggiate: una riga di contesto ciascuna — perché è emersa, dove potrà rientrare)*

- `operator new` non ritorna `nullptr` su fallimento (lancia `std::bad_alloc`) — aggancia il difetto `C6011` su `new IndirectDeviceContext(Device)` nel driver e una misconception negli appunti di Valentina; chiudere al ritorno sul driver
- `IDD_IS_FIELD_AVAILABLE` — versioning runtime del framework; ramo `if` (HDR / callback `…2`) è quello vivo sulla VM (Win11 + WDK recente), ramo `else` è codice morto per lei
- Dove viene chiamata `InitAdapter` (non in `DeviceAdd`) — prossimo anello del ciclo di vita
- Doppia `RegCloseKey` in `EnabledQuery` (VDD) — difetto locale trovato, potenzialmente proponibile upstream
- `RegQueryValueExW` senza terminatore null garantito in `initpath` (VDD) — bufferover-read; famiglia `C6387`
- fat pointer (puntatori a funzione membro > 8 byte) — solo se emerge nel codice
- Guida alla creazione dei certificati di test — richiesta esplicita di Valentina, rimandata

## Materiale attivo
- **Virtual Display Driver** (VirtualDrivers/Virtual-Display-Driver, branch `master`): lettura guidata del ciclo di vita, un anello alla volta. Posizione attuale: `VirtualDisplayDriverDeviceAdd` (riga ~2812), pronti a leggerne il corpo. Struttura reale: file unico `MttVDD/Driver.cpp` (~5000 righe), non una god-class ma un god-file con ~60 globali; `IndirectDeviceContext` è ~330 righe (3653-3988). Pipeline funzionante (VS 2026 + WDK + VM Win11 test-signing); cert `MttVDD Test Cert` (99F8…) allineato host/VM e ora fissato nel progetto.
- **Calcolatrice** (progetto personale ground-up): da portare in questo repo dal desktop.

## Menù possibile
*(possibilità, non impegni — nessun ordine, nessuna priorità)*

- VDD: leggere il corpo di `DeviceAdd` e seguire dove nasce `InitAdapter`
- VDD: chiudere i difetti parcheggiati (`new`/`C6011`, `RegCloseKey`, `RegQueryValueExW`/`C6387`)
- Puntatori: giro di ripasso a sorpresa (richiesto da Valentina)
- Portare la calcolatrice nel repo e fare il punto sul suo stato
- Guida alla creazione dei certificati di test
