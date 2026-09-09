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
- **`sizeof(std::string)` vs `std::string::size()`**: `sizeof` restituisce la dimensione dell'oggetto classe (32 su x64: puntatore, size, capacity, SSO buffer), non la lunghezza del contenuto. `size()` restituisce la lunghezza del contenuto
- **Double-call pattern per API registro**: prima chiamata con buffer NULL per ottenere la dimensione necessaria, `resize()` della stringa, seconda chiamata per leggere. Dopo la lettura `resize(dwBufferSize - 1)` per rimuovere il null terminator incluso nel conteggio
- **`std::string` e null terminator**: `std::string` non usa `\0` per determinare la lunghezza — tiene traccia separatamente. Un `\0` scritto dentro la stringa è un carattere come un altro; `size()` non lo riconosce come terminatore
- **`RegGetValue` parametri**: `lpSubKey` è una sotto-chiave relativa a `hKey` (comodità per evitare `RegOpenKeyEx` separata); `lpValueName` è il nome del valore. Nel VDD i valori sono piatti sotto la chiave principale → `lpSubKey = ""`, `lpValueName` = nome completo
- **Character set del progetto**: "Not Set" / "Multi-Byte" / "Unicode" determina se le macro TCHAR (`RegGetValue`, `CharUpperBuff` ecc.) risolvono a versione A (narrow) o W (wide). Coerenza obbligatoria tra apertura della chiave e chiamate successive
- **Igiene sugli errori**: se una funzione può fallire (tipo ritorno lo dice), controllare il risultato — anche se una chiamata precedente alla stessa funzione è riuscita. Condizioni esterne possono cambiare tra le due chiamate
- **`GetText()` di TinyXML2**: restituisce `nullptr` se l'elemento non ha testo. Assegnare `nullptr` a `std::string` è UB/crash. Check obbligatorio prima dell'assegnazione
- **`RootElement()` di TinyXML2**: può restituire `nullptr` anche se `LoadFile` ha avuto successo (file con solo dichiarazione XML, nessun elemento radice)
- **`strtok` vs `find`/`substr`**: `strtok` è C puro, modifica la stringa sorgente (riempie di `\0`), usa stato statico (non thread-safe). Tokenizzazione con `find`/`substr` è C++ idiomatico
- **Dangling pointer/handle**: avevi un riferimento valido a una risorsa, qualcun altro l'ha liberata, il tuo riferimento punta nel vuoto. Distinto da memory leak (perdi il riferimento, la risorsa resta) e da memoria non inizializzata (nessun valore scritto)
- **Named pipe**: meccanismo IPC di Windows — un "tubo" con un nome nel sistema (es. `\\.\pipe\VDDPipe`) tra due processi; uno scrive con `WriteFile`, l'altro legge dall'altro capo. Nel VDD: canale bidirezionale tra driver e companion app
- **`explicit` sui costruttori**: impedisce al compilatore di usare quel costruttore per conversioni implicite. `today = local_days_value` non compila; serve `today = year_month_day{local_days_value}`. Chi progetta la classe decide se una conversione è abbastanza "ovvia" da essere implicita
- **Dichiarazione vs assegnazione**: `Type var{value}` è inizializzazione alla dichiarazione; `var = Type{value}` è assegnazione a variabile esistente (costruisce un temporaneo e lo assegna). Sintassi diversa per momenti diversi nella vita della variabile
- **`std::chrono` C++20**: `std::format("{:%Y-%m-%d %X}", zt)` formatta direttamente tipi chrono. `zoned_time{tz, time_point}` converte da UTC a ora locale. `current_zone()` restituisce puntatore a oggetto statico della timezone database (non posseduto, non va liberato). `floor<days>` su `local_time` vs `system_clock::now()` (UTC) — differenza rilevante a cavallo della mezzanotte
- **Ordine inizializzazione membri**: i membri si inizializzano nell'ordine di dichiarazione nell'header, non nell'ordine dell'initializer list. La ragione: c'è un solo distruttore, che deve invertire un ordine unico. L'initializer list va allineato alla dichiarazione per evitare confusione
- **Macro TCHAR e versioni A/W**: `CreateDirectory`, `RegGetValue` ecc. sono macro che si espandono a `…W` (wide) o `…A` (narrow) in base al CharacterSet del progetto. Si può chiamare direttamente la versione esplicita (`CreateDirectoryA`) per bypassare la macro
- **Multibyte = codifica dove un carattere può occupare più di un byte**: `WideCharToMultiByte` con code page `CP_UTF8` produce UTF-8. "Multibyte" è il termine generico di Windows, non uno specifico encoding

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
- Lambda: teoria (cattura e closure) non consolidata — usata concretamente in `std::visit` e `GetSetting` (cattura di variabili, `this`). Richiesta esplicita di ripasso da Valentina
- `IDD_IS_FIELD_AVAILABLE` — versioning runtime del framework; ramo `if` (HDR / callback `…2`) è quello vivo sulla VM (Win11 + WDK recente), ramo `else` è codice morto per lei
- Doppia `RegCloseKey` in `EnabledQuery` (VDD) — `EnabledQuery` ora rimossa, il difetto non esiste più nel refactoring
- fat pointer (puntatori a funzione membro > 8 byte) — solo se emerge nel codice
- Guida alla creazione dei certificati di test — richiesta esplicita di Valentina, rimandata
- **Separazione architetturale driver/settings**: il driver dovrebbe ricevere la configurazione, non leggerla — in corso di realizzazione con il refactoring
- **Teoria su puntatori, reference, const, double pointer**: Valentina li usa ma la comprensione teorica è frammentaria. Da affrontare con teoria + esercizio mirato. Legato al filo conduttore ownership/lifetime
- **std::visit + std::variant — ripasso teoria**: meccanismo, perché funziona, alternative, limiti. Richiesta esplicita di Valentina
- **Static vs shared linking**: mismatch LNK2038 su `tinyxml2.lib` (debug/release, CRT statica/dinamica). Valentina ha dichiarato di non aver capito la differenza. Da affrontare come teoria
- **Visibilità tra unità di traduzione (extern, scope globale)**: confusione tra `extern`, `friend`, variabile globale. Da collegare a compilazione separata e dichiarazione vs definizione
- **Centralizzazione lettura monitor config**: la sezione risoluzioni/refresh rates nell'XML ha struttura diversa (liste ripetute, non scalari) dal pattern attuale di SettingsLoader. Valutare estensione futura

## Argomenti toccati — indice compatto

### Linguaggio C++
- Puntatori, reference, passaggio per valore/riferimento
- Template: specializzazione, istanziazione, definizione in header
- `std::variant`, `std::visit`, `std::remove_pointer_t`, `decltype`
- `using` (type alias)
- Overload resolution e le sue trappole
- Costruttori `= default` e inizializzazione dei membri
- Costruttori `explicit` e conversioni implicite
- `sizeof` su classi vs `size()` su contenuti
- `std::string` e gestione interna del null terminator
- `std::chrono` C++20: `zoned_time`, `current_zone`, `std::format` con tipi chrono
- Dangling pointer/handle
- Ordine inizializzazione membri e initializer list
- Lambda: cattura variabili e `this` (uso pratico, teoria da consolidare)

### Build system e toolchain
- Compilazione vs linking (`.cpp` → `.obj` → `.exe`/`.dll`)
- Name mangling, `dllexport`/`dllimport`, `.lib` di import
- Toolset diversi tra progetti nella stessa solution
- Clean + rebuild per risolvere simboli stale / PDB disallineati
- Character set del progetto (Not Set / Multi-Byte / Unicode) e impatto sulle API TCHAR
- Rinominazione progetto Visual Studio: allineare cartella, `.vcxproj`, `.vcxproj.filters`, `.sln`, `<RootNamespace>`
- Warning WDK trattati come errori (`/WX`): disabilitare warning specifici per header di sistema (`4471`)
- Mismatch CRT tra librerie: `_ITERATOR_DEBUG_LEVEL`, `RuntimeLibrary` — static vs shared, debug vs release

### Windows API
- Registry: `RegOpenKeyEx`, `RegQueryValueExW`, `RegGetValue`, `RegCloseKey`
- Tipi registro: `REG_DWORD`, `REG_SZ`
- Handle: tipi opachi, `HKEY`
- Double-call pattern per lettura valori registro (size query → allocate → read)
- `RegGetValue`: parametri `lpSubKey` vs `lpValueName`, flag `RRF_RT_*`
- Named pipe: IPC tra processi, `WriteFile` per scrivere su `HANDLE`
- Versioni A/W delle API: `CreateDirectoryA` vs `CreateDirectoryW`, bypass della macro TCHAR
- `WideCharToMultiByte` con `CP_UTF8`: conversione wide→narrow, termine "multibyte" in Windows
- DXGI: `DXGI_ADAPTER_DESC::Description` è `WCHAR[128]`, non ha versione narrow

### Design e architettura
- Separazione responsabilità: reader/loader/utility/logger
- Source of truth per i settings (XML primario, registro override)
- Mapping path→campo con variant di puntatori
- Struttura dati allineata al DOM XML
- Gestione risorse: open/close nella stessa funzione vs split tra funzioni diverse (trade-off)
- Uso vs possesso: il logger *usa* la pipe (`HANDLE*`) ma non la possiede — non crea, non chiude
- File di log tenuto aperto con rotazione a cambio data (vs open/close a ogni messaggio)
- Dipendenze esplicite via costruttore vs globali/Singleton: il costruttore rende visibile chi dipende da cosa
- Separazione tipi IddCx dai settings generici: `globals.h` (tipi base) + `globals_iddcx.h` (tipi framework)
- Valori derivati vs valori letti: `SDR_COLOR`/`HDR_COLOR` sono calcolati da settings, non settings loro stessi

### Driver Windows (IddCx/WDF)
- Ciclo di vita a 6 stadi
- Callback: registrazione e implementazione
- `DeviceAdd`, `D0Entry`, `InitAdapter`
- UMDF come DLL in `WUDFHost.exe`
- Pipe driver↔companion app: `StartNamedPipeServer` → `NamedPipeServer` → `HandleClient`; ciclo di vita dell'handle gestito da `HandleClient`
- Ordine inizializzazione driver: `DriverEntry` (settings) → `DeviceAdd` (`new IndirectDeviceContext`) → `D0Entry` (`InitAdapter`)
- `IddCxGetVersion` + `IDARG_OUT_GETVERSION`: versioning runtime del framework

### Librerie esterne
- TinyXML2: navigazione DOM, `FirstChildElement`, `GetText`, `RootElement` — null checks necessari
- Visitor pattern (discusso, scartato per il caso d'uso)

## Materiale attivo

### Virtual Display Driver — analisi e refactoring
- **Fork di studio**: `ghostintheshell-192/Virtual-Display-Driver-Ref` — fork di `itsmikethetech/Virtual-Display-Driver`. Detach dal parent richiesto a GitHub support (le PR defaultano sull'upstream). Pubblico.
- **Convenzioni stabilite**: `STYLE_GUIDE.md` + `.clang-format` in root. Naming: `snake_case` variabili/funzioni nostre, `PascalCase` classi/struct e callback framework, `UPPER_CASE` costanti. Formattazione: Microsoft base, tab, 120-col.
- **Progetto console `MttVddRefactor`**: banco di prova isolato per le classi refactorizzate. Naming file allineato: `.vcxproj` e `.vcxproj.filters` rinominati da `MttVddSettings` a `MttVddRefactor`, `<RootNamespace>` aggiornato.
- **Branch `refactor/globals` (mergiato)**: ~50 globali migrate in `DriverSettings` con sotto-struct in `globals.h`. Istanza globale `g_settings`.
- **Branch `refactor/settings-reading` (mergiato — PR #3)**: architettura settings completata:
  - `SettingsLoader`: orchestratore. Riceve `DriverSettings*` dall'esterno, possiede il vettore `entries` (coppie chiave-puntatore), e i due reader
  - `RegistryReader`: `RegGetValue` con flag `RRF_RT_*`, double-call pattern
  - `XmlReader`: TinyXML2, navigazione DOM segmento per segmento
  - `utilities.h`: `convert_setting<T>`, `tokenize`, `WStringToString`, `StringToWstring`, type alias `SettingValuePtr`
  - `globals.h`: `DriverSettings` con sotto-struct (tipi base)
  - `globals_iddcx.h`: struct con campi di tipo IddCx (`IDDCX_BITS_PER_COMPONENT`, `IDDCX_XOR_CURSOR_SUPPORT`), incluso solo dal driver
- **Branch `refactor/logging` (mergiato)**: classe `Logger` estratta da `vddlog()`:
  - `enum class LogType` al posto del char per il tipo di log
  - File tenuto aperto con rotazione a cambio data (check `HasDateChanged()` a ogni messaggio)
  - `HANDLE*` alla pipe globale — uso senza possesso, dereferenziato a ogni chiamata
  - `zoned_time` + `current_zone()` per timestamp locali (C++20)
  - `ToggleStandardLogs`/`ToggleDebugLogs`/`TogglePipedLogs` con apertura/chiusura file coerente
  - `SendToPipe` privato, chiamato internamente da `Message`
  - Debug logging in `GetSetting`: confronto vecchio/nuovo valore dentro la lambda `std::visit`
- **Branch `refactor/driver-cleanup` (in corso)**: integrazione moduli nel progetto driver
  - Moduli (settings, logging, utilities) importati in MttVDD, compilano con il driver
  - `EnabledQuery`, `GetIntegerSetting`, `GetStringSetting`, `GetDoubleSetting`, `LogQueries` rimosse — sostituite da `g_settings_manager.Init()` + `LoadSettings()`
  - `SettingsQueryMap` ancora presente ma non più usata — da rimuovere
  - Phase 5 (`ValidateEdidIntegration`, `ValidateAndSanitizeConfiguration` ecc.) ripulita: legge da `g_settings` invece di rileggere da disco. Bug trovati: `validationPassed` mai messo a false, sanitizzazione su variabili locali mai riscritte in `g_settings`
  - `vddlog()` ancora definita con ~318 chiamate — sostituzione con `g_log.Message()` è il prossimo pezzo grosso
  - `WStringToString` duplicata rimossa: una sola copia in `utilities.h` (namespace `Refactoring`), dichiarazione globale tolta da `Driver.h`
  - Driver passato a C++20 (per chrono); warning WDK 4471/4499/4505 disabilitati; `CreateDirectoryA`/`RegGetValueA` al posto delle macro
  - `tinyxml2.lib`: usa versione shared/release per compatibilità con CRT del driver UMDF

### Ciclo di vita IddCx — lettura guidata
- Mappa architetturale committata in `riferimenti/architettura-driver-windows.md`
- `DriverEntry`: letta — carica settings, poi registra callback `DeviceAdd`
- `DeviceAdd`: letta e chiusa
- `D0Entry` + `InitAdapter`: esplorati
- Stadi 4-5 (monitor, frame): non ancora toccati

### Calcolatrice
- Progetto personale ground-up: da portare in questo repo dal desktop.

## Menù possibile
*(possibilità, non impegni — nessun ordine, nessuna priorità)*

- VDD refactoring: sostituire 318 chiamate `vddlog()` con `g_log.Message()` — meccanico ma grosso
- VDD refactoring: rimuovere `SettingsQueryMap` (non più usata)
- VDD refactoring: rimuovere definizione di `vddlog()` e `SendToPipe()` vecchie
- VDD refactoring: pensare alla scrittura dei settings (registro) — `SetSetting`
- VDD refactoring: estrarre caricamento monitor config (risoluzioni/refresh rates) — forma diversa dagli scalari
- VDD refactoring: X-macros — applicare se emerge un pattern ripetitivo nella forma finale
- VDD: stadi 4-5 del ciclo di vita (monitor, frame)
- VDD: chiudere i difetti parcheggiati (`new`/`C6011`)
- Teoria + esercizio: puntatori, reference, const, double pointer — consolidamento fondamenta
- Teoria + esercizio: lambda — cattura, closure, teoria
- Teoria + esercizio: std::visit + std::variant — meccanismo e uso
- Teoria + esercizio: static vs shared linking, CRT
- Teoria + esercizio: visibilità tra unità di traduzione (extern, dichiarazione vs definizione)
- Teoria + esercizio: inizializzazione membri in C++ — esercizi mirati
- Teoria + esercizio: overload resolution — riconoscere quale overload si sta invocando
- Puntatori: giro di ripasso a sorpresa (richiesto da Valentina)
- Portare la calcolatrice nel repo e fare il punto sul suo stato
- Guida alla creazione dei certificati di test
