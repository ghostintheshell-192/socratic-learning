# Registro tutoriale

Osservazioni su pattern, lacune, aree di esercizio. Aggiornato da Claude a fine sessione. Serve per decidere proattivamente quando proporre esercizi, ripasso, o domande mirate.

## Aree deboli — per esercizio mirato

### 1. Inizializzazione dei membri primitivi
- **Cosa**: dimentica di inizializzare `bool`, `int`, `HKEY` etc. nella dichiarazione della classe
- **Causa radice**: convinzione (corretta in sessione 2026-08-28) che `= default` inizializzi tutto. Ora sa che `= default` delega ai singoli membri e i primitivi restano indeterminati
- **Frequenza**: 4 occorrenze in una sessione (2026-08-28)
- **Esercizio suggerito**: quiz "cosa vale X dopo la costruzione?" su struct miste (primitivi + oggetti STL)
- **Ultima osservazione**: 2026-08-28

### 2. Overload resolution / firma delle funzioni
- **Cosa**: assume il comportamento di una funzione dal nome e dalla forma della chiamata senza verificare quale overload si invoca
- **Casi concreti**: `std::string::replace` vs `std::replace` (da `<algorithm>`), `find(char)` vs `find(const char*)`, `find_first_of` (significato del terzo parametro)
- **Causa radice**: impara facendo, non leggendo la documentazione. Quando una chiamata "sembra giusta" non verifica la firma
- **Esercizio suggerito**: dato un frammento, "quale overload viene chiamato qui e cosa fa?"
- **Ultima osservazione**: 2026-08-28

### 3. Tracking delle correzioni
- **Cosa**: quando riceve una lista di fix, i primi vengono applicati, gli ultimi evaporano
- **Causa radice**: frettolosità + distraibilità (tratto SCT). Non è un problema di comprensione
- **Contromisura concordata**: Claude dà meno correzioni per messaggio; usa la forma "in queste righe ci sono N errori, quali?" per forzare l'attenzione
- **Ultima osservazione**: 2026-08-28

### 4. Completezza del flusso di controllo
- **Cosa**: fall-through dopo un ramo di successo (es. `return` mancante dopo if), messaggi di errore stampati anche in caso di successo
- **Frequenza**: 2+ occorrenze (2026-08-28)
- **Esercizio suggerito**: code review mirate su funzioni con rami multipli — "cosa stampa questa funzione se tutto va bene?"
- **Ultima osservazione**: 2026-08-28

### 5. Fidarsi di IntelliSense vs il compilatore
- **Cosa**: assume che la mancanza di errori in IntelliSense significhi che il codice compila
- **Caso concreto**: (2026-09-09) aggiunto membro `Logger*` a `XmlReader`, IntelliSense non segnava errori, ma il compilatore sì (`Logger` non dichiarato — mancava l'include)
- **Causa radice**: IntelliSense ha un suo motore di parsing separato dal compilatore; può avere un'idea diversa di cosa è visibile
- **Contromisura**: compilare sempre prima di concludere che qualcosa funziona
- **Ultima osservazione**: 2026-09-09

## Teoria da consolidare

### Puntatori, reference, const
- **Stato**: li usa correttamente nella pratica ma la spiegazione teorica è frammentaria
- **Dettaglio**: "a malapena ti saprei spiegare cosa sono". Il doppio puntatore "l'ho capito e non l'ho capito — non riesco a immaginarmelo"
- **Approccio**: teoria + esercizio mirato, collegato al filo conduttore ownership/lifetime
- **Ultima osservazione**: 2026-08-28
- **Aggiornamento 2026-09-09**: la "giungla di puntatori" creata passando `DriverSettings*` a SettingsLoader ha provocato disagio e tentativo di cercare scorciatoie (extern, globale). Il concetto è usato correttamente (puntatore a struct esterna → entries puntano ai campi), ma la complessità percepita rimane alta

### Lambda functions
- **Stato**: usate correttamente nella pratica (cattura, `this`, parametri), teoria non consolidata
- **Dettaglio**: cattura e closure non affrontate in teoria. Le lambda sono usate concretamente in `std::visit` e in `GetSetting` (cattura di `raw_reg_value`, `value_key`, `this`)
- **Approccio**: ripasso teoria quando riemerge il tema naturalmente
- **Richiesta esplicita di Valentina**: 2026-09-09

### std::visit + std::variant
- **Stato**: pattern usato operativamente, da ripassare sia come meccanismo che come uso pratico
- **Dettaglio**: sa che `std::visit` chiama il callable col tipo concreto, sa usare `auto*` + `remove_pointer_t` + `decltype`. Da consolidare: perché funziona, le alternative, i limiti
- **Richiesta esplicita di Valentina**: 2026-09-09

### Static vs shared linking
- **Stato**: non compreso
- **Caso concreto**: (2026-09-09) errore LNK2038 — mismatch `_ITERATOR_DEBUG_LEVEL` e `RuntimeLibrary` tra `tinyxml2.lib` (Debug, CRT dinamica) e driver (CRT statica release). Risolto usando la versione shared/release della lib, ma Valentina ha dichiarato di non aver capito la differenza static/shared
- **Collegamento**: CRT statica vs dinamica, cosa succede al linking, perché il driver UMDF usa CRT statica in release anche in config Debug
- **Richiesta esplicita di Valentina**: 2026-09-09

### Visibilità tra unità di traduzione (extern, scope globale)
- **Stato**: regola nota ma non interiorizzata ("non mi ricordo queste cose")
- **Caso concreto**: (2026-09-09) voleva accedere a `g_settings` da `settings_loader.cpp` — ha proposto `extern`, poi `friend`, poi variabile globale, senza distinzione chiara tra i tre meccanismi. Dopo guida ha riconosciuto che passare per costruttore era la scelta coerente col design
- **Collegamento**: compilazione separata, unità di traduzione, dichiarazione vs definizione
- **Ultima osservazione**: 2026-09-09

## Pattern positivi — da sfruttare

- **Ragionamento architetturale forte**: le decisioni di design (separazione classi, precedenza registro, percorso unificato, variant di puntatori) arrivano spontaneamente prima dei suggerimenti
- **Resilienza nel debugging concettuale**: non molla quando qualcosa non torna (es. `find_first_of` — ha resistito, fatto prove, chiesto chiarimenti ripetuti finché non ha fatto click)
- **Sa eliminare**: riconosce quando la soluzione più semplice è togliere (dependency dllimport, wstring)
- **Chiede feedback strutturato**: a fine sessione ha chiesto valutazione esplicita pregi/difetti e ha suggerito lei stessa come migliorare il processo tutoriale
- **Autoconsapevolezza sui propri limiti**: "so usarli ma non ti saprei spiegare cosa sono" è una dichiarazione precisa e utile, non una lamentela
- **Riconosce quando sta correndo troppo**: (2026-09-09) "non mi calmo e inizio a fare le cose senza pensare" — consapevolezza in tempo reale del pattern, anche se il pattern si ripete

## Istruzioni operative (da Valentina, 2026-08-28)

- Se un errore si ripete, non limitarti a segnalarlo: scava con domande per capire la convinzione errata sottostante
- Il fatto che faccia giusto qualcosa non significa che la teoria sia solida — verificare periodicamente
- Dare meno correzioni per messaggio, su più messaggi
- Usare la forma "ci sono N errori, quali?" per forzare l'attenzione
