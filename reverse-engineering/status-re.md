# Status — Reverse engineering

Mappa dei concetti del percorso di reverse engineering. Come `status-cpp.md`, elenca e non spiega. È separato dal percorso C++: le sessioni di questo percorso stanno in `reverse-engineering/sessioni/` (un file per sessione, nome = timestamp UTC), le osservazioni tutoriali in `reverse-engineering/registro-tutor.md`.

Ruolo di Claude: mentore. Valentina fa i gesti in Ghidra, Claude spiega, verifica e lascia a lei le scelte ai bivi.

## Regole di sicurezza del percorso

- Solo analisi statica. Il campione e gli stadi decodificati non si eseguono mai.
- Il dominio del server di comando (C2) non si contatta e non si risolve.
- Nessun invio a servizi esterni (es. VirusTotal) senza richiesta esplicita di Valentina.

## Materiale attivo

### Campione: loader Go → Donut → payload x64

Catena a stadi, decodificata staticamente da Claude nella prima parte della sessione 2026-10-11, con Valentina che sceglieva la direzione:

1. **Loader in Go** — nasconde i dati in coda al file; li decodifica (alfabeto esadecimale personalizzato → zlib → AES-256-CBC) e li esegue in memoria tramite una sezione condivisa e il thread pool di Windows (`NtCreateSection`, `TpAllocWork`). Anti-analisi: attesa attiva di 8–12 s, controllo del nome della scheda video.
2. **Shellcode Donut** (versione modificata) — decifra e decomprime (LZNT1) l'eseguibile successivo.
3. **Payload x64 (MSVC)** — profila la macchina (utente, lingue, layout di tastiera, software, processi, schede di rete), cerca segni di VM e di antivirus, comunica con un server via WinHTTP e può scaricare ed eseguire uno stadio successivo (PowerShell / cmd). Non è uno stealer di per sé.

In Ghidra: il loader ha 2192 nomi recuperati dalla pclntab di Go; il payload è il programma su cui lavora Valentina a mano.

## Concetti acquisiti

- Navigazione Ghidra: pannelli (Program Trees, Symbol Tree, Data Type Manager, Listing, Decompiler), `G` per saltare a un indirizzo, `Alt+←` per tornare indietro
- Little-endian: i byte di un valore in memoria sono in ordine inverso rispetto a come si scrive il numero
- XREF: elenco dei punti che usano un indirizzo; `(c)` = chiamata, `(*)` = puntatore/dato; il riferimento in `.pdata` (tabella delle eccezioni) si ignora
- Rinominare (`L`) e cambiare tipo (`Ctrl+L`): due gesti diversi
- Decompilato pieno di `auVar…` = codice SIMD/SSE che il decompilatore rende male; leggere il Listing o saltare alle chiamate
- Offset in una struttura: `ptr + 0x50` = campo a distanza fissa dall'inizio di un blocco di memoria descritto da un tipo
- Archivi di tipi (`windows_vs12_64`): non contengono tutto; un tipo mancante si crea a mano
- Structure Editor: struttura parziale con solo i campi usati dal codice, il resto `undefined`; `pack` disattivato = offset decisi a mano; `Apply` per salvare
- Applicare un tipo a una variabile trasforma gli offset in nomi di campo (`lVar6->PhysicalAddress[0]`)

## Concetti in corso

- Indirizzo MAC e OUI (primi 3 byte = produttore; le VM hanno OUI propri)
- `CreateToolhelp32Snapshot`: fotografia dei processi, non snapshot di una VM
- Registro di Windows: chiavi dei servizi in `HKLM\SYSTEM\CurrentControlSet\Services`; gli strumenti delle VM lasciano lì le loro tracce
- Offuscamento delle stringhe con XOR a byte singolo (`0x55`)

## Parking lot

- Calling convention x64 di Windows (RCX, RDX, R8, R9): utile per leggere le chiamate nel Listing
- Struttura di un PE (sezioni, `.pdata`, tabella degli import)

## Menù possibile

- Finire la funzione anti-VM (`FUN_140001e30`) e risalire a chi la chiama
- Controllo antivirus / lingua (stringhe `kasper`, `drweb`, `yandex`, `punto` + `GetKeyboardLayoutList`)
- Comunicazione con il server (WinHTTP)
- Download ed esecuzione dello stadio successivo (PowerShell)
