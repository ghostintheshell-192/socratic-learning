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
- **Driver come libreria**: non ha un `main()`, è caricato dal sistema; tu scrivi le risposte (callback), il sistema decide quando fare le domande
- **WDF vs IddCx**: WDF generico (device, power, cleanup — qualsiasi driver), IddCx specifico per display indiretti (adapter, monitor, frame). Lo stadio 3 (D0Entry → InitAdapter) è il punto di contatto
- **Ciclo di vita IddCx** — sei stadi in ordine fisso: registrazione → configurazione device → accensione → display → funzionamento → teardown
- **Callback**: registrare = riempire campi di struct con puntatori a funzione; scrivere = il corpo di quelle funzioni
- **D0Entry → InitAdapter**: D0Entry è WDF (power), dentro chiama InitAdapter che è IddCx. Il pattern è identico a DeviceAdd: riempi struct → passa al framework → ricevi handle. `IddCxAdapterInitAsync` ritorna subito; completamento via callback `AdapterInitFinished`. Lo stesso `IndirectDeviceContext` agganciato a due handle diversi (device e adapter) — secondo cartello, stessa casa
- **Passaggio per valore vs per riferimento**: senza `&` il C++ copia — la funzione modifica la copia locale, l'originale resta invariato. Con `&` il parametro è un alias dell'originale
- **Template specializations**: `template<> T convert_setting<T>(const wstring&)` — una specializzazione per tipo (bool, int, double, wstring), il chiamante sceglie il tipo, il template sceglie la conversione
- **Registry Windows**: `REG_DWORD` (intero 32 bit) e `REG_SZ` (stringa) sono i tipi rilevanti; non esiste un tipo nativo double; `RegQueryValueExW` con `lpType` restituisce il tipo del valore; il codice originale passava `NULL` e indovinava con due tentativi

## Concetti in corso
- Ownership transfer via puntatore consumato e azzerato dalla callee (visto su `pDeviceInit` in `WdfDeviceCreate`, VDD)
- Vita agganciata: oggetto C++ (`new` in `DeviceAdd`) la cui `delete` sta in `Cleanup()`, chiamata dal framework via `EvtCleanupCallback` prima di distruggere l'oggetto proprietario — `new` e `delete` in funzioni lontane, invocate da soggetti diversi
- Contesto WDF allocato *dentro* il blocco dell'oggetto device (`WDF_OBJECT_ATTRIBUTES_INIT_CONTEXT_TYPE` + `WdfDeviceCreate`); `WdfObjectGet_…` calcola un offset, non alloca
- Versioning a runtime vs compile-time (`IDD_IS_FIELD_AVAILABLE`, VDD) — corretta un'interpretazione iniziale errata (protezione memoria)
- **X-macros**: esplorazione in teoria — `#` (stringizzazione), `##` (token pasting), `.def` file come dati-only. Valentina ha identificato il caso d'uso (generare sia struct che mapping XML da un'unica definizione) ma vuole prima arrivare al codice finale "a mano" e poi estrarre il pattern

## Parking lot
*(tangenti parcheggiate: una riga di contesto ciascuna — perché è emersa, dove potrà rientrare)*

- `operator new` non ritorna `nullptr` su fallimento (lancia `std::bad_alloc`) — aggancia il difetto `C6011` su `new IndirectDeviceContext(Device)` nel driver e una misconception negli appunti di Valentina; chiudere al ritorno sul driver
- `~IndirectDeviceContext`: `lock_guard`, `mutex`, `swap` di una mappa, `unique_ptr` — visti di sfuggita leggendo il distruttore, tutti da aprire quando il percorso li incontra
- Lambda: vista meccanicamente (funzione senza nome scritta sul posto e assegnata a un campo, confrontata con `PnpPowerCallbacks.EvtDeviceD0Entry`); cattura e closure non toccate
- `IDD_IS_FIELD_AVAILABLE` — versioning runtime del framework; ramo `if` (HDR / callback `…2`) è quello vivo sulla VM (Win11 + WDK recente), ramo `else` è codice morto per lei
- Doppia `RegCloseKey` in `EnabledQuery` (VDD) — difetto locale trovato, potenzialmente proponibile upstream
- `RegQueryValueExW` senza terminatore null garantito in `initpath` (VDD) — bufferover-read; famiglia `C6387`
- fat pointer (puntatori a funzione membro > 8 byte) — solo se emerge nel codice
- Guida alla creazione dei certificati di test — richiesta esplicita di Valentina, rimandata
- `#include "Driver.cpp"` nel progetto test — funziona ma crea problemi con definizioni duplicate se il progetto cresce; da sistemare quando si stabilizza
- **Separazione architetturale driver/settings**: il driver dovrebbe ricevere la configurazione, non leggerla. Nel VDD originale le due responsabilità sono mescolate. Il refactoring in corso sta già isolando il codice settings in un progetto separato, che è il primo passo verso la separazione

## Materiale attivo

### Virtual Display Driver — analisi e refactoring
- **Fork di studio**: `ghostintheshell-192/Virtual-Display-Driver-Ref` — fork di `itsmikethetech/Virtual-Display-Driver`. Detach dal parent richiesto a GitHub support (le PR defaultano sull'upstream). Pubblico.
- **Convenzioni stabilite**: `STYLE_GUIDE.md` + `.clang-format` in root. Naming: `snake_case` variabili/funzioni nostre, `PascalCase` classi/struct e callback framework, `UPPER_CASE` costanti. Formattazione: Microsoft base, tab, 120-col.
- **Branch `refactor/globals` (mergiato)**: ~50 globali migrate in `DriverSettings` con 7 sotto-struct (`LogSettings`, `EdidSettings`, `CursorSettings`, `ColorSettings`, `HdrSettings`, `AutoResolutionSettings`, `MonitorEmulationSettings`) in `globals.h`. Istanza globale `g_settings`.
- **Branch `refactor/settings-reading` (in corso)**: progetto console `MttVddSettings` per sperimentare il refactoring della lettura impostazioni. Stato attuale:
  - Mappa semplificata (rimossa colonna registry, derivata da uppercase del nome XML)
  - `read_registry_value()` restituisce sempre `wstring` (normalizza DWORD e REG_SZ)
  - `convert_setting<T>` con specializzazioni per bool/int/double/wstring
  - `get_setting<T>` come collante: lettura + conversione
  - **Bug trovato**: il parser XML cerca per nome elemento senza contesto parent; 7 entry `"enabled"` risolvono tutte alla prima occorrenza nel file
  - **Decisione aperta**: come strutturare la lettura XML. TinyXML2 scelto come parser (albero navigabile vs parser sequenziale di XmlLite). Due sorgenti (registry e XML) da tenere separate, non mischiare in un'unica funzione
  - **Prossimo passo**: decidere l'architettura della lettura settings (due reader separati, mappa pre-caricata, interfaccia comune) prima di scrivere altro codice

### Ciclo di vita IddCx — lettura guidata
- Mappa architetturale committata in `riferimenti/architettura-driver-windows.md`
- `DriverEntry`: localizzata (~riga 2405), non ancora letta in dettaglio
- `DeviceAdd`: letta e chiusa
- `D0Entry` + `InitAdapter`: esplorati — pattern identico a DeviceAdd (riempi struct → framework → handle), asincronia via callback `AdapterInitFinished` → `FinishInit` → `CreateMonitor`
- Stadi 4-5 (monitor, frame): non ancora toccati

### Calcolatrice
- Progetto personale ground-up: da portare in questo repo dal desktop.

## Menù possibile
*(possibilità, non impegni — nessun ordine, nessuna priorità)*

- VDD refactoring: **decidere l'architettura della lettura settings** — TinyXML2, separazione registry/XML, struttura della mappa pre-caricata
- VDD refactoring: **X-macros** — applicare al codice settings una volta raggiunta la forma finale
- VDD: `DriverEntry` in dettaglio
- VDD: stadi 4-5 del ciclo di vita (monitor, frame)
- VDD: chiudere i difetti parcheggiati (`new`/`C6011`, `RegCloseKey`, `RegQueryValueExW`/`C6387`)
- Puntatori: giro di ripasso a sorpresa (richiesto da Valentina)
- Portare la calcolatrice nel repo e fare il punto sul suo stato
- Guida alla creazione dei certificati di test
