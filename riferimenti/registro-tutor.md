# Registro tutoriale

Osservazioni su come impara Valentina, valide per tutti i percorsi. Aggiornato da Claude a fine sessione. Serve a decidere quando proporre esercizi, ripasso o domande mirate. Le regole generali su come insegnare (comprese quelle date da Valentina il 2026-08-28) stanno nelle istruzioni del progetto.

## Trasversali

### Ritmo e forma dei messaggi
- **Cosa**: (2026-10-11, reverse engineering) Claude rispondeva più in fretta di quanto lei riuscisse a leggere ed eseguire; ha interrotto una risposta per incollare il codice che aveva davanti
- **Contromisura**: un passo per messaggio, poi fermarsi e aspettare il riscontro (meglio se incolla il codice). Non anticipare il passo successivo
- **Nota**: si è scusata per essere "lenta a leggere". Il ritmo è suo; non è un difetto
- **Ultima osservazione**: 2026-10-11

### Tracking delle correzioni
- **Cosa**: quando riceve una lista di correzioni, applica le prime e perde le ultime
- **Causa**: fretta + distraibilità. Non è un problema di comprensione
- **Contromisura**: poche correzioni per messaggio; forma "in queste righe ci sono N errori: quali?"
- **Ultima osservazione**: 2026-08-28

### Dedurre il comportamento dal nome
- **Cosa**: deduce cosa fa una funzione dal nome o dalla forma della chiamata, senza verificare firma e documentazione
- **Casi**: C++ — `std::string::replace` e `std::replace`, `find(char)` e `find(const char*)`, terzo parametro di `find_first_of` (2026-08-28). Reverse engineering — `CreateToolhelp32Snapshot` letto come "snapshot della VM" (2026-10-11)
- **Causa**: impara facendo, non leggendo la documentazione
- **Esercizio**: dato un frammento, "quale overload viene chiamato e cosa fa?"; per ogni API sconosciuta, leggere la prima frase e i parametri della pagina Microsoft prima di fare ipotesi
- **Ultima osservazione**: 2026-10-11

## C++ — aree deboli

### Inizializzazione dei membri primitivi
- **Cosa**: dimentica di inizializzare `bool`, `int`, `HKEY` nella dichiarazione della classe
- **Causa**: credeva che `= default` inizializzasse tutto (corretto il 2026-08-28)
- **Esercizio**: quiz "quanto vale X dopo la costruzione?" su struct miste (primitivi + oggetti STL)
- **Ultima osservazione**: 2026-08-28 (4 occorrenze in una sessione)

### Completezza del flusso di controllo
- **Cosa**: fall-through dopo un ramo di successo (`return` mancante), messaggi di errore stampati anche quando va tutto bene
- **Esercizio**: code review su funzioni con più rami — "cosa stampa se va tutto bene?"
- **Ultima osservazione**: 2026-08-28

### Fidarsi di IntelliSense invece del compilatore
- **Caso**: (2026-09-09) membro `Logger*` aggiunto a `XmlReader` senza include: IntelliSense taceva, il compilatore no
- **Contromisura**: compilare sempre prima di concludere che funziona
- **Ultima osservazione**: 2026-09-09

### Copia e riferimento
- **Cosa**: dimentica `&` nel range-for; parametri per valore dove bastava `const&`
- **Casi**: (2026-09-16) `vector_trim` con `for (auto tok : tokens)`; `SetConfigurationFile(std::string path)`
- **Causa**: non ha ancora interiorizzato che il default del C++ è la copia. Il codice compila, il bug è silenzioso
- **Contromisura**: a ogni range-for o parametro, chiedersi "leggo o modifico?"
- **Ultima osservazione**: 2026-09-16

### Valori sentinella della standard library
- **Cosa**: tratta `npos`, `end()`, `nullptr` come valori magici senza chiedersi cosa sono concretamente
- **Caso**: (2026-09-16) `tokenize` funzionava per caso grazie all'overflow di `npos + 1`
- **Contromisura**: davanti a una costante della standard library, chiedersi "che valore è? come si comporta nell'aritmetica?"
- **Ultima osservazione**: 2026-09-16

### Complicare prima di semplificare
- **Cosa**: quando un algoritmo non funziona, prova approcci diversi (ricorsione, altri pattern) prima di correggere la versione che ha
- **Caso**: (2026-09-16) riscrittura di `tokenize`
- **Causa**: il pensiero associativo la porta a esplorare; utile per imparare, meno per finire
- **Contromisura**: "prima correggi, poi rifattorizza". Cambiare approccio solo se quello attuale è inadatto nella struttura
- **Ultima osservazione**: 2026-09-16

### Puntatori e visibilità: teoria frammentaria
- **Puntatori**: li usa correttamente, ma "a malapena ti saprei spiegare cosa sono"; il doppio puntatore "l'ho capito e non l'ho capito". (2026-09-09) La "giungla di puntatori" nel passare `DriverSettings*` a `SettingsLoader` le ha dato disagio e l'ha spinta a cercare scorciatoie (`extern`, globale)
- **Visibilità**: (2026-09-09) per accedere a `g_settings` ha proposto `extern`, poi `friend`, poi una globale, senza distinguere i tre meccanismi. Con guida ha scelto il costruttore
- **Ultima osservazione**: 2026-09-09

## Reverse engineering — aree deboli

### Sopraffazione davanti al decompilato rumoroso
- **Cosa**: "è lunga, tutta variabili, e pochissimo codice" davanti a una funzione SIMD
- **Contromisura**: regola insegnata — con molti `auVar`, saltare alle chiamate di API e leggere il resto nel Listing
- **Ultima osservazione**: 2026-10-11

## Pattern positivi — da sfruttare

- **Ragionamento architetturale forte**: le decisioni di design (separazione delle classi, precedenza del registro, variant di puntatori) arrivano spontaneamente, prima dei suggerimenti
- **Fa la domanda strutturale**: (2026-10-11) "perché proprio 0x50, 0x51, 0x52?" — la domanda che porta alle strutture
- **Mappa da sola i consumatori**: (2026-09-16) tutti i punti che leggono `monitorModes`, tutti i consumatori di `AdapterOption`
- **Riconosce i punti decisivi del flusso**: (2026-10-11) ha notato da sola che il confronto sugli offset era una condizione di `break`
- **Sa eliminare**: riconosce quando la soluzione più semplice è togliere (`UpdateXmlSetting`, `split` duplicata, struct `Options`)
- **Resilienza nel debugging concettuale**: non molla finché qualcosa non torna (es. `find_first_of`)
- **Formula ipotesi**: (2026-10-11) "è un info-stealer" — non esatto, ma l'idea di fondo era giusta
- **Porta a termine operazioni lunghe**: (2026-10-11) prima struttura nello Structure Editor senza errori, verificando ogni riga
- **Accetta i vicoli ciechi**: (2026-09-16) nella riscrittura di `tokenize` è tornata indietro senza drammi; l'autocritica è stata momentanea
- **Consapevole dei propri limiti**: "so usarli ma non ti saprei spiegare cosa sono" è una dichiarazione precisa, non una lamentela; riconosce in tempo reale quando sta correndo troppo
- **Chiede feedback strutturato** e propone lei come migliorare il processo tutoriale
