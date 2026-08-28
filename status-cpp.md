# Status — Studio C++

Mappa dei concetti del percorso. Questo file è la memoria di Claude tra le sessioni: elenca, non spiega. La rielaborazione dei contenuti vive nei quaderni di Valentina.

Gli handoff discorsivi delle singole sessioni stanno in `sessioni/` (un file per sessione, nome = timestamp UTC).

Il registro delle osservazioni tutoriali (lacune, pattern, aree di esercizio) sta in `riferimenti/registro-tutor.md`.

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
- **Compilazione vs linking**: il compilatore lavora su un `.cpp` alla volta producendo un `.obj`; errori `C`-prefisso = compilatore, `LNK`-prefisso = linker. Il linker incolla gli `.obj` in un eseguibile
- **Name mangling C++**: il compilatore codifica nome funzione + tipi parametri + tipo ritorno in una stringa unica. Tipi C (`void`, `const char*`) producono mangling stabile tra toolset; tipi C++ (`std::string`) possono divergere tra toolset diversi — ecco perché le DLL esportano quasi solo tipi C
- **`__declspec(dllexport/dllimport)`**: `dllexport` nella DLL sorgente, `dllimport` nel consumatore. Il linker del consumatore cerca i simboli in un `.lib` di import, non direttamente nella `.dll`
- **Template: definizione nell'header obbligatoria**: un template è una ricetta, non codice; il compilatore genera il codice solo quando incontra un uso concreto con un tipo specifico, e in quel momento deve vedere il corpo. Se il corpo è in un `.cpp` separato, l'unità di traduzione che usa il template non può generare nulla → `LNK2019`
- **`std::visit` + `std::variant`**: `std::visit` prende un callable e una variant, chiama il callable col tipo concreto contenuto. Con `auto*` nel lambda + `std::remove_pointer_t<decltype(ptr)>` si deduce il tipo senza bisogno di enum o switch esterno. La variant è sia contenitore che discriminante
- **`= default` non inizializza i tipi primitivi**: il costruttore generato delega ai singoli membri; i tipi con costruttore proprio (`std::string`, `std::vector`) si inizializzano, i primitivi (`bool`, `int`, `HKEY`) restano a valore indeterminato
- **Overload resolution**: lo stesso nome di funzione può avere comportamenti radicalmente diversi a seconda dei tipi passati — `std::string::replace` vs `std::replace` (da `<algorithm>`), `find(char)` vs `find(const char*)`, `find_first_of(const char*, pos, count)` dove `count` è la lunghezza del set di caratteri da cercare, non il range di ricerca
- **`RegGetValue` vs `RegQueryValueEx`**: `RegGetValue` garantisce null-termination per `REG_SZ` e permette filtraggio per tipo via flag `RRF_RT_*`

## Concetti in corso
- Ownership transfer via puntatore consumato e azzerato dalla callee (visto su `pDeviceInit` in `WdfDeviceCreate`, VDD)
- Vita agganciata: oggetto C++ (`new` in `DeviceAdd`) la cui `delete` sta in `Cleanup()`, chiamata dal framework via `EvtCleanupCallback` prima di distruggere l'oggetto proprietario — `new` e `delete` in funzioni lontane, invocate da soggetti diversi
- Contesto WDF allocato *dentro* il blocco dell'oggetto device (`WDF_OBJECT_ATTRIBUTES_INIT_CONTEXT_TYPE` + `WdfDeviceCreate`); `WdfObjectGet_…` calcola un offset, non alloca
- Versioning a runtime vs compile-time (`IDD_IS_FIELD_AVAILABLE`, VDD) — corretta un'interpretazione iniziale errata (protezione memoria)
- **X-macros**: esplorazione in teoria — `#` (stringizzazione), `##` (token pasting), `.def` file come dati-only. Valentina ha identificato il caso d'uso (generare sia struct che mapping XML da un'unica definizione) ma ha poi optato per `std::visit` + variant

## Parking lot
*(tangenti parcheggiate: una riga di contesto ciascuna — perché è emersa, dove potrà rientrare)*

- `operator new` non ritorna `nullptr` su fallimento (lancia `std::bad_alloc`) — aggancia il difetto `C6011` su `new IndirectDeviceContext(Device)` nel driver e una misconception negli appunti di Valentina; chiudere al ritorno sul driver
- `~IndirectDeviceContext`: `lock_guard`, `mutex`, `swap` di una mappa, `unique_ptr` — visti di sfuggita leggendo il distruttore, tutti da aprire quando il percorso li incontra
- Lambda: vista meccanicamente (funzione senza nome scritta sul posto); cattura e closure non toccate — ora usata concretamente nel `std::visit`, buon punto di rientro
- `IDD_IS_FIELD_AVAILABLE` — versioning runtime del framework; ramo `if` (HDR / callback `…2`) è quello vivo sulla VM (Win11 + WDK recente), ramo `else` è codice morto per lei
- Doppia `RegCloseKey` in `EnabledQuery` (VDD) — difetto locale trovato, potenzialmente proponibile upstream
- fat pointer (puntatori a funzione membro > 8 byte) — solo se emerge nel codice
- Guida alla creazione dei certificati di test — richiesta esplicita di Valentina, rimandata
- **Separazione architetturale driver/settings**: il driver dovrebbe ricevere la configurazione, non leggerla. Nel VDD originale le due responsabilità sono mescolate
- **Teoria su puntatori, reference, const, double pointer**: Valentina li usa ma la comprensione teorica è frammentaria. Da affrontare con teoria + esercizio mirato. Legato al filo conduttore ownership/lifetime
- **`InitializePath` nel `RegistryReader`**: usa `RegQueryValueExW` (wide) su un `std::string` (narrow) — type mismatch. `sizeof(path)` dà la dimensione dell'oggetto string, non del contenuto. Da correggere

## Argomenti toccati — indice compatto

### Linguaggio C++
- Puntatori, reference, passaggio per valore/riferimento
- Template: specializzazione, istanziazione, definizione in header
- `std::variant`, `std::visit`, `std::remove_pointer_t`, `decltype`
- `using` (type alias)
- Overload resolution e le sue trappole
- Costruttori `= default` e inizializzazione dei membri

### Build system e toolchain
- Compilazione vs linking (`.cpp` → `.obj` → `.exe`/`.dll`)
- Name mangling, `dllexport`/`dllimport`, `.lib` di import
- Toolset diversi tra progetti nella stessa solution
- Clean + rebuild per risolvere simboli stale / PDB disallineati

### Windows API
- Registry: `RegOpenKeyExW`, `RegQueryValueExW`, `RegGetValue`, `RegCloseKey`
- Tipi registro: `REG_DWORD`, `REG_SZ`
- Handle: tipi opachi, `HKEY`

### Design e architettura
- Separazione responsabilità: reader/loader/utility
- Source of truth per i settings (XML primario, registro override)
- Mapping path→campo con variant di puntatori
- Struttura dati allineata al DOM XML

### Driver Windows (IddCx/WDF)
- Ciclo di vita a 6 stadi
- Callback: registrazione e implementazione
- `DeviceAdd`, `D0Entry`, `InitAdapter`
- UMDF come DLL in `WUDFHost.exe`

### Librerie esterne
- TinyXML2: navigazione DOM, `FirstChildElement`, `GetText`
- Visitor pattern (discusso, scartato per il caso d'uso)

## Materiale attivo

### Virtual Display Driver — analisi e refactoring
- **Fork di studio**: `ghostintheshell-192/Virtual-Display-Driver-Ref` — fork di `itsmikethetech/Virtual-Display-Driver`. Detach dal parent richiesto a GitHub support (le PR defaultano sull'upstream). Pubblico.
- **Convenzioni stabilite**: `STYLE_GUIDE.md` + `.clang-format` in root. Naming: `snake_case` variabili/funzioni nostre, `PascalCase` classi/struct e callback framework, `UPPER_CASE` costanti. Formattazione: Microsoft base, tab, 120-col.
- **Branch `refactor/globals` (mergiato)**: ~50 globali migrate in `DriverSettings` con sotto-struct in `globals.h`. Istanza globale `g_settings`.
- **Branch `refactor/settings-reading` (in corso)**: progetto console `MttVddSettings` per la lettura impostazioni. Architettura completata:
  - `SettingsLoader`: orchestratore. Possiede `DriverSettings`, il vettore `entries` (coppie chiave-puntatore), e i due reader. `Init()` apre le sorgenti e popola le entries. `LoadSettings()` fa un unico loop: XML prima, registro dopo (l'ultimo che scrive vince)
  - `RegistryReader`: apre/chiude `HKEY`, `GetSetting` riceve chiave stringa + `SettingValuePtr` (variant di puntatori), `GetRawRegistryValue` usa `RegGetValue`
  - `XmlReader`: carica il file con TinyXML2, `GetSetting` tokenizza la chiave su `.` e naviga il DOM segmento per segmento
  - `utilities.h`: `convert_setting<T>` (specializzazioni bool/int/double/string), `tokenize`, type alias `SettingValuePtr`
  - `globals.h`: `DriverSettings` con sotto-struct allineate alla struttura XML/globali originali (9 sotto-struct: Log, Cursor, Edid, EdidIntegration, Colour, HdrAdvanced con ColorPrimaries e ColorSpace, AutoResolution con EdidModeFiltering e PreferredMode, ColorAdvanced con BitDepthManagement e ColorFormatExtended, MonitorEmulation)
  - **Da fare**: correggere `InitializePath` (type mismatch wide/narrow), pulire codice commentato residuo

### Ciclo di vita IddCx — lettura guidata
- Mappa architetturale committata in `riferimenti/architettura-driver-windows.md`
- `DriverEntry`: localizzata (~riga 2405), non ancora letta in dettaglio
- `DeviceAdd`: letta e chiusa
- `D0Entry` + `InitAdapter`: esplorati
- Stadi 4-5 (monitor, frame): non ancora toccati

### Calcolatrice
- Progetto personale ground-up: da portare in questo repo dal desktop.

## Menù possibile
*(possibilità, non impegni — nessun ordine, nessuna priorità)*

- VDD refactoring: pulire il codice commentato residuo nel settings-reading branch
- VDD refactoring: correggere `InitializePath` (type mismatch wide/narrow)
- VDD refactoring: pensare alla scrittura dei settings (registro) — `SetSetting`
- VDD refactoring: X-macros — applicare se emerge un pattern ripetitivo nella forma finale
- VDD: `DriverEntry` in dettaglio
- VDD: stadi 4-5 del ciclo di vita (monitor, frame)
- VDD: chiudere i difetti parcheggiati (`new`/`C6011`, `RegCloseKey`)
- Teoria + esercizio: puntatori, reference, const, double pointer — consolidamento fondamenta
- Teoria + esercizio: inizializzazione membri in C++ — esercizi mirati
- Teoria + esercizio: overload resolution — riconoscere quale overload si sta invocando
- Puntatori: giro di ripasso a sorpresa (richiesto da Valentina)
- Portare la calcolatrice nel repo e fare il punto sul suo stato
- Guida alla creazione dei certificati di test
