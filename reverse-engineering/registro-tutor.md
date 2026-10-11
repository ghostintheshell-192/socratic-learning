# Registro tutoriale — Reverse engineering

Osservazioni su pattern, lacune e aree di esercizio, specifiche di questo percorso. Aggiornato da Claude a fine sessione. Le osservazioni generali sul modo di imparare di Valentina restano in `riferimenti/registro-tutor.md`, che vale anche qui.

## Ritmo e forma dei messaggi

- **Cosa**: (2026-10-11) Claude rispondeva più in fretta di quanto lei riuscisse a leggere ed eseguire; ha interrotto una risposta per incollare il codice che aveva davanti
- **Contromisura**: un passo per messaggio, poi fermarsi e aspettare il suo riscontro (meglio se incolla il codice). Non anticipare il passo successivo. Coerente con "meno correzioni per messaggio" del registro C++
- **Nota**: si è scusata per essere "lenta a leggere". Il ritmo è suo; non va trattato come un difetto

## Aree deboli — per esercizio mirato

### 1. Dedurre la funzione di un'API dal nome
- **Cosa**: interpreta un nome di API per assonanza col contesto
- **Caso concreto**: (2026-10-11) `CreateToolhelp32Snapshot` letto come "snapshot della VM"; incerta su cosa c'entri il registro con una VM
- **Collegamento**: è lo stesso pattern dell'area 2 del registro C++ (assumere il comportamento dal nome senza verificare la firma)
- **Contromisura suggerita**: per ogni API sconosciuta, aprire la pagina Microsoft e leggere almeno la prima frase e i parametri prima di formulare un'ipotesi
- **Ultima osservazione**: 2026-10-11

### 2. Senso di sopraffazione davanti al decompilato rumoroso
- **Cosa**: "è lunga, tutta variabili, e pochissimo codice" davanti a una funzione SIMD
- **Contromisura**: regola insegnata: quando il decompilato è pieno di `auVar`, saltare alle chiamate di API e leggere il resto nel Listing
- **Ultima osservazione**: 2026-10-11

## Pattern positivi — da sfruttare

- **Fa la domanda strutturale**: (2026-10-11) "perché proprio 0x50, 0x51, 0x52?" Ha capito da sola che erano posizioni precise in memoria e ha chiesto il perché, che è la domanda che porta alle strutture
- **Riconosce i punti decisivi del flusso**: ha notato da sola che il confronto sugli offset era una condizione di `break`, "quindi mi pare importante"
- **Lettura d'insieme corretta dai nomi**: dalle API in chiaro ha capito che la funzione cercava tracce di una VM
- **Formula ipotesi**: sul payload ha proposto "è un info-stealer". Non lo è di per sé, ma l'idea di fondo (dati inviati a un server web) è giusta: da verificare sullo stadio successivo
- **Porta a termine un'operazione lunga**: ha creato la sua prima struttura nello Structure Editor senza errori, seguendo i passi e verificando a ogni riga
