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

## Teoria da consolidare

### Puntatori, reference, const
- **Stato**: li usa correttamente nella pratica ma la spiegazione teorica è frammentaria
- **Dettaglio**: "a malapena ti saprei spiegare cosa sono". Il doppio puntatore "l'ho capito e non l'ho capito — non riesco a immaginarmelo"
- **Approccio**: teoria + esercizio mirato, collegato al filo conduttore ownership/lifetime
- **Ultima osservazione**: 2026-08-28

## Pattern positivi — da sfruttare

- **Ragionamento architetturale forte**: le decisioni di design (separazione classi, precedenza registro, percorso unificato, variant di puntatori) arrivano spontaneamente prima dei suggerimenti
- **Resilienza nel debugging concettuale**: non molla quando qualcosa non torna (es. `find_first_of` — ha resistito, fatto prove, chiesto chiarimenti ripetuti finché non ha fatto click)
- **Sa eliminare**: riconosce quando la soluzione più semplice è togliere (dependency dllimport, wstring)
- **Chiede feedback strutturato**: a fine sessione ha chiesto valutazione esplicita pregi/difetti e ha suggerito lei stessa come migliorare il processo tutoriale
- **Autoconsapevolezza sui propri limiti**: "so usarli ma non ti saprei spiegare cosa sono" è una dichiarazione precisa e utile, non una lamentela

## Istruzioni operative (da Valentina, 2026-08-28)

- Se un errore si ripete, non limitarti a segnalarlo: scava con domande per capire la convinzione errata sottostante
- Il fatto che faccia giusto qualcosa non significa che la teoria sia solida — verificare periodicamente
- Dare meno correzioni per messaggio, su più messaggi
- Usare la forma "ci sono N errori, quali?" per forzare l'attenzione
