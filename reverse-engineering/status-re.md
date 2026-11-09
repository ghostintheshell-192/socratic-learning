# Status — Reverse engineering

Elenca, non spiega: una riga per voce. Le sessioni stanno in `reverse-engineering/sessioni/`; le osservazioni su come impara Valentina in `riferimenti/registro-tutor.md`.

Livello di guida: massimo. Valentina parte da zero, quindi Claude spiega tutto, un gesto alla volta. I gesti in Ghidra li fa lei; le scelte ai bivi sono sue.

## Regole di sicurezza del percorso

- Solo analisi statica. Il campione e gli stadi decodificati non si eseguono mai.
- Il dominio del server di comando (C2) non si contatta e non si risolve.
- Nessun invio a servizi esterni (es. VirusTotal) senza richiesta esplicita di Valentina.

## Richieste esplicite

*(nessuna per ora)*

## Concetti acquisiti

- Pannelli di Ghidra (Program Trees, Symbol Tree, Data Type Manager, Listing, Decompiler); `G` per saltare a un indirizzo, `Alt+←` per tornare indietro
- Little-endian: in memoria i byte di un valore sono in ordine inverso
- XREF: i punti che usano un indirizzo; `(c)` = chiamata, `(*)` = puntatore o dato; il riferimento in `.pdata` si ignora
- Rinominare (`L`) e cambiare tipo (`Ctrl+L`) sono due gesti diversi
- Decompilato pieno di `auVar…` = codice SIMD: leggere il Listing o saltare alle chiamate di API
- `ptr + 0x50` = campo a distanza fissa dall'inizio di una struttura
- Gli archivi di tipi (`windows_vs12_64`) non contengono tutto; un tipo mancante si crea a mano
- Structure Editor: struttura parziale con solo i campi usati, il resto `undefined`; `pack` disattivato per decidere gli offset a mano; `Apply` per salvare
- Applicare un tipo a una variabile trasforma gli offset in nomi di campo

## Concetti in corso

- Indirizzo MAC e OUI (i primi 3 byte indicano il produttore; le VM hanno OUI propri)
- `CreateToolhelp32Snapshot`: fotografia dei processi, non snapshot di una VM
- Registro: le chiavi dei servizi in `HKLM\SYSTEM\CurrentControlSet\Services`, dove gli strumenti delle VM lasciano tracce
- Stringhe offuscate con XOR a byte singolo (`0x55`)

## Parking lot

- Calling convention x64 di Windows (RCX, RDX, R8, R9) — utile per leggere le chiamate nel Listing
- Struttura di un PE (sezioni, `.pdata`, tabella degli import)

## Materiale

### Campione: loader Go → Donut → payload x64

Catena decodificata staticamente da Claude, con Valentina che sceglieva la direzione:

1. **Loader in Go** — dati in coda al file, decodificati (alfabeto esadecimale personalizzato → zlib → AES-256-CBC) ed eseguiti in memoria (`NtCreateSection`, `TpAllocWork`). Anti-analisi: attesa di 8–12 s, controllo del nome della scheda video
2. **Shellcode Donut** (modificato) — decifra e decomprime (LZNT1) l'eseguibile successivo
3. **Payload x64 (MSVC)** — profila la macchina, cerca VM e antivirus, comunica via WinHTTP, può scaricare ed eseguire uno stadio successivo (PowerShell / cmd)

In Ghidra: il loader ha 2192 nomi recuperati dalla pclntab di Go; Valentina lavora a mano sul payload.

Punto in cui si è fermata: dentro `FUN_140001e30` (anti-VM principale), con `IP_ADAPTER_ADDRESSES_LH` creata e applicata al ciclo sulle schede di rete. `FUN_140001af0` rinominata `check_vm_registry` (cerca `VBoxGuest`/`VMTools`, ma per un errore di cifratura apre `SQSTEM\…` e non trova mai niente).

## Menù possibile
*(possibilità, non impegni)*

- Finire `FUN_140001e30`: rinomine (`lVar6` → `adapter`, `bVar2` → `vm_found`), decifrare a mano una delle 7 stringhe XOR dei processi, analizzare `FUN_1400017f0` e il controllo su RAM e CPU
- Dare un nome a `FUN_140001e30` e risalire a chi la chiama
- Verificare la stringa `virtual`ox` (probabilmente lo stesso errore di `SQSTEM`)
- Controllo antivirus e lingua (`kasper`, `drweb`, `yandex`, `punto` + `GetKeyboardLayoutList`)
- Comunicazione con il server (WinHTTP)
- Download ed esecuzione dello stadio successivo (PowerShell)
