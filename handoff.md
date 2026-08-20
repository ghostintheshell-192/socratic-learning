# Handoff — ultima sessione

*(Questo file contiene solo l'ultima sessione e viene sovrascritto ogni volta. Lo storico è in git log.)*

## Sessione di setup (agosto 2026)

Sessione fondativa: nessun contenuto C++, tutta infrastruttura e definizione del metodo.

**Cosa è stato deciso** (il dettaglio è nelle istruzioni di progetto, che sono la fonte autorevole):
- Approccio socratico calibrato: domande sui concetti, spiegazioni dirette su sintassi/meccanica.
- Parking lot per le tangenti; nessun concetto nominato prima del suo momento (regola ferrea).
- Niente schedule né solleciti: le sessioni avvengono quando avvengono, chiusura dichiarata da Valentina.
- Filo conduttore del percorso: la gestione della memoria (ownership, lifetime) come fondamenta.
- Divisione dei ruoli: questo repo è la memoria di Claude; la rielaborazione vive nei quaderni di Valentina. I file di stato elencano, non spiegano.

**Infrastruttura**: connettore GitHub MCP configurato e testato (lettura e scrittura ok). Nota di troubleshooting: per le GitHub App, *autorizzazione* (identità) e *installazione* (accesso ai contenuti dei repo) sono due atti separati — se le scritture falliscono con 403 "Resource not accessible by integration", verificare che l'app "Claude Github MCP Connector" risulti installata sull'account con accesso a questo repo (github.com/apps/claude-github-mcp-connector). La creazione di repository non rientra nei permessi dell'app: va fatta a mano da Valentina.

**Stato del materiale**: repo appena creato; la calcolatrice è ancora sul desktop di Valentina (verrà portata qui con un push, eventualmente via Claude Code che ha già accesso). Nessun progetto open source ancora scelto — ne esiste uno già incontrato sul lavoro che Valentina sta analizzando, da identificare in una prossima sessione.

**Per la prossima istanza**: non c'è ancora alcun contenuto didattico avviato. Si parte da dove Valentina vorrà — il menù è in status-cpp.md.
