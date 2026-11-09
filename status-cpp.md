# Status — C++

Elenca, non spiega: una riga per voce. La rielaborazione vive nei quaderni di Valentina. I concetti acquisiti sono anche la riserva per le domande di ripasso.

## Richieste esplicite di teoria
*(da offrire per prime nel menù)*

- Puntatori, reference, `const`, doppi puntatori — li usa bene ma la teoria è frammentaria; il doppio puntatore "non riesce a immaginarlo"
- Lambda — cattura e closure (le usa già in `std::visit` e `GetSetting`)
- `std::visit` + `std::variant` — perché funziona, alternative, limiti
- Linking statico e condiviso, CRT — dall'errore `LNK2038` su `tinyxml2.lib`
- Visibilità tra unità di traduzione — `extern`, scope globale, dichiarazione e definizione
- Memoria e rappresentazione binaria, complemento a 2 — come la memoria è organizzata e interpretata (nata da `npos + 1`)
- Creazione dei certificati di test

## Concetti acquisiti

### Memoria, puntatori, possesso
- lvalue/rvalue col test operativo (sta a sinistra di `=`, si può prendere con `&`)
- Tipo del puntatore = livelli di indirezione; dimensione sempre 8 byte su x64; il tipo governa la dereferenziazione
- `delete p` libera l'oggetto, non azzera `p`
- Possedere = avere il dovere di liberare, non avere l'indirizzo
- La cascata dei distruttori si ferma sul puntatore grezzo
- `delete` assente: dimenticanza o non-possesso; distruttore che non nomina un membro = non-possesso
- Uso senza possesso (il `Logger` usa la pipe, non la crea né la chiude)
- Dangling, leak, memoria non inizializzata: tre problemi diversi
- Passaggio per valore e per riferimento; `const&` per i parametri in sola lettura
- Range-for: `auto` copia, `auto&` no
- RVO
- Metodi come funzioni pure: variabile locale invece di stato membro

### Linguaggio
- Template: specializzazione, corpo obbligatorio nell'header, template sul tipo di ritorno (`apply_range<T>`)
- `std::visit` + `std::variant`, con `auto*` + `std::remove_pointer_t<decltype(ptr)>`
- `= default` non inizializza i primitivi; inizializzatori sui membri
- `Foo();` dichiara, `Foo() = default;` definisce
- I membri si inizializzano nell'ordine di dichiarazione
- Costruttori `explicit`
- Inizializzazione alla dichiarazione e assegnazione
- Overload resolution e le sue trappole (`replace`, `find`, `find_first_of`)
- `sizeof(std::string)` e `size()`; `std::string` non usa `\0` come terminatore
- `npos` = `(size_t)-1`: partecipa all'aritmetica
- `[[nodiscard]]`; l'operatore virgola come bug
- `static_cast` per il narrowing esplicito; `enum class`
- `UNREFERENCED_PARAMETER`
- Early return al posto di `else`; graffe per limitare la vita di un `lock_guard`
- `std::format` (specifier, cast per gli enum); `std::chrono` C++20 (`zoned_time`, `current_zone`)
- STL: `map::find`, `sort` con comparatore, `sort` + `unique`, `std::tie`; `strtok` contro `find`/`substr`
- Dispatch table con `std::function`

### Build e toolchain
- Compilazione (`.cpp` → `.obj`, errori `C`) e linking (errori `LNK`)
- Name mangling; le DLL esportano tipi C; `dllexport`/`dllimport` e `.lib` di import
- Character set del progetto e macro TCHAR (versioni A/W)
- "Multibyte" = codifica a più byte; `WideCharToMultiByte` + `CP_UTF8`
- Warning WDK trattati come errori e disabilitati uno per uno; clean + rebuild per simboli stale; rinominare un progetto Visual Studio

### Windows API
- Handle opachi: `DECLARE_HANDLE`/`STRICT`, struct fantasma per il controllo dei tipi
- Registro: `REG_DWORD`/`REG_SZ`, `RegGetValue` e `RegQueryValueEx`, `lpSubKey` e `lpValueName`, doppia chiamata (dimensione → allocazione → lettura)
- Controllare sempre il ritorno di ciò che può fallire
- Named pipe come IPC; `ReadFile` su qualsiasi handle di I/O
- Frequenza di refresh come frazione (`DISPLAYCONFIG_RATIONAL`)

### Driver (WDF / IddCx)
- Il driver è una libreria: niente `main`, scrivi callback e il sistema decide quando chiamarle; UMDF = DLL in `WUDFHost.exe`
- WDF generico, IddCx specifico per i display indiretti; ciclo di vita in 6 stadi
- Registrare una callback = riempire struct di puntatori a funzione
- `DriverEntry` → `DeviceAdd` → `D0Entry` → `InitAdapter` (asincrono, completamento via callback)
- Callback colore/HDR: i dati arrivano al driver; senza hardware sono no-op
- Matrice colore 3×4 (trasformazione 3×3 + offset); conversioni SMPTE ST.2086

### Librerie
- TinyXML2: `GetText` e `RootElement` possono restituire `nullptr`; DOM e file sono cose separate (`SaveFile`)

### Design
- Dipendenze esplicite nel costruttore invece di globali
- Il driver riceve la configurazione, non la legge
- Il codice morto si elimina
- Dati grezzi nel profilo, conversione al momento dell'uso
- Tipi del framework separati dai tipi generici (`globals.h` / `globals_iddcx.h`)
- Valori derivati e valori letti
- Migrazione produttore → consumatori con ponte temporaneo
- "Cosa modella questa classe?" (`AdapterOption` = una selezione, non un adapter)

## Concetti in corso
- Passaggio di possesso con puntatore consumato e azzerato dalla funzione chiamata (`pDeviceInit` in `WdfDeviceCreate`)
- Vita agganciata: `new` in `DeviceAdd`, `delete` in `Cleanup()` chiamata dal framework
- Contesto WDF allocato dentro il blocco del device; `WdfObjectGet_…` calcola un offset, non alloca
- Versioning a runtime (`IDD_IS_FIELD_AVAILABLE`)
- X-macros: viste in teoria, scartate a favore di `std::visit`

## Parking lot
*(tangenti: si riprendono quando il percorso le incontra o quando Valentina le richiama)*

- `operator new` lancia `std::bad_alloc`, non restituisce `nullptr` — difetto `C6011` su `new IndirectDeviceContext` e un'idea sbagliata negli appunti
- `~IndirectDeviceContext`: `lock_guard`, `mutex`, `swap` di una mappa, `unique_ptr` — visti di sfuggita
- Fat pointer (puntatori a funzione membro) — solo se emergono nel codice
- `SettingsLoader`: separare Get e Set solo se emergono cicli di vita diversi
- Due caricamenti separati (`LoadSettings` in `DriverEntry`, `loadSettings` in `DeviceAdd`)
- Rilascio ufficiale del fork e certificato di firma valido per i PC dei clienti — quando il refactoring sarà stabile

## Materiale

### VDD — fork di studio
- Repo: `ghostintheshell-192/Virtual-Display-Driver-Ref`. Per ora da parte (al lavoro si usa l'upstream): occasione di studio. Obiettivo futuro: versione stabile e rilascio ufficiale del fork
- Convenzioni: `STYLE_GUIDE.md` + `.clang-format` in root (snake_case per variabili e funzioni nostre, PascalCase per classi e callback, UPPER_CASE per costanti; base Microsoft, tab, 150 colonne)
- Banco di prova: progetto console `MttVddRefactor`
- Mergiati: `refactor/globals`, `refactor/settings-reading` (PR #3), `refactor/logging`
- In corso: `refactor/driver-cleanup`, PR da aprire. Moduli settings/logging/utilities integrati nel driver; `vddlog()` sostituita dal `Logger`; codice morto rimosso (Phase 5, catena EDID, strutture intermedie colore/HDR); `MonitorProfile` è la fonte dei modi per le callback; driver in C++20. Il dettaglio è nella storia git del fork
- Fili aperti del refactoring:
  - `AdapterOption`: separare la selezione (da dove viene il nome) dalla ricerca (nome → LUID) e portarla nel driver
  - `s_KnownMonitorModes2`: dove far vivere la conversione verso `DISPLAYCONFIG_VIDEO_SIGNAL_INFO` senza mettere tipi IddCx in `MonitorProfile`
  - Loader per `g_custom_profile` (XML con elementi ripetuti)
  - Spostare i campi fisici (primari, luminanza, gamma, color space) da `DriverSettings` a `MonitorProfile`
  - Separare gamma e matrice colore nei metodi `Get_*`, quando avranno un consumatore
  - Mappa comandi di `HandleClient` → dispatch table (idea del collega)

### VDD — lettura guidata del ciclo di vita
- Mappa in `riferimenti/architettura-driver-windows.md`
- Letti: `DriverEntry`, `DeviceAdd`, `D0Entry` + `InitAdapter`. Non ancora: stadi 4-5 (monitor, frame)

### Calcolatrice
- Progetto personale da zero, ancora sul PC di Valentina e non nel repo

## Menù possibile
*(possibilità, non impegni)*

- Una delle richieste esplicite di teoria
- VDD: uno dei fili aperti del refactoring
- VDD: stadi 4-5 del ciclo di vita
- VDD: chiudere `C6011` / `operator new`
- Esercizi mirati dal registro tutor (inizializzazione dei membri, overload, copia e riferimento, flusso di controllo)
- Domanda di ripasso a sorpresa sui puntatori
- Calcolatrice: portarla nel repo e fare il punto
- Code review del codice scritto finora
