# socratic-learning

Memoria di Claude per il percorso di studio di Valentina. Come insegnare sta nelle istruzioni del progetto su claude.ai; questo file descrive solo la struttura del repo ed è l'unica fonte su di essa. Se la struttura cambia, si aggiorna qui.

## Percorsi

| Percorso | Status | Sessioni | Materiale |
|---|---|---|---|
| C++ | `status-cpp.md` | `sessioni/` | fork VDD: `ghostintheshell-192/Virtual-Display-Driver-Ref` (repo separato) · upstream per confronto: `VirtualDrivers/Virtual-Display-Driver` (branch `master`) |
| Reverse engineering | `reverse-engineering/status-re.md` | `reverse-engineering/sessioni/` | campione sul PC di Valentina, non nel repo |

Ogni status ha le stesse sezioni: richieste esplicite, concetti acquisiti, concetti in corso, parking lot, materiale, menù possibile. I concetti acquisiti sono anche la riserva da cui pescare le domande di ripasso.

## Altri file

- `riferimenti/registro-tutor.md` — osservazioni su come impara Valentina, valide per tutti i percorsi
- `riferimenti/architettura-driver-windows.md` — mappa del driver VDD (ciclo di vita IddCx)

## Inizio sessione

1. Leggi `riferimenti/registro-tutor.md`.
2. Se Valentina non ha ancora scelto su cosa lavorare, leggi gli status di tutti i percorsi per offrirle il menù.
3. Per il percorso scelto: leggi il suo status e la sessione più recente nella sua cartella `sessioni/` (nome = timestamp UTC, quindi la più recente è l'ultima in ordine alfabetico).
4. Prima di analizzare codice in un repo esterno, rileggi sempre i file: Valentina può aver fatto commit nel frattempo.

## Chiusura

1. **Nuovo file di sessione** nella cartella `sessioni/` del percorso, nome `AAAA-MM-GGTHHMMSSZ.md` (UTC). Testo discorsivo e sintetico: cosa si è fatto, cosa ha funzionato e cosa no, intuizioni o difficoltà notevoli, decisioni prese, fili aperti (come possibilità, non come compiti). Serve alla prossima istanza per riprendere tono e contesto, non solo i contenuti.
2. **Status del percorso aggiornato**. Elenca, non spiega: una riga per voce. Niente date, niente impegni.
3. **`riferimenti/registro-tutor.md`** aggiornato se sono emerse osservazioni nuove.
4. Questo README, se la struttura è cambiata.

Un solo commit per chiusura, messaggio breve in italiano.

## Note tecniche sul connettore GitHub

- Per leggere da un branch diverso da quello di default, `ref` va passato completo: `refs/heads/<branch>`.
- Per aggiornare un file con `create_or_update_file` serve il suo `sha` attuale; `push_files` non lo richiede.
- I nomi di file con spazi (es. `Virtual Display Driver (HDR)/MttVDD/Driver.cpp`) si passano così come sono.
- Il merge delle PR sul fork si fa dall'interfaccia web: il connettore non ha i permessi.
