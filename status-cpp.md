# Status — Studio C++

Mappa dei concetti del percorso. Questo file è la memoria di Claude tra le sessioni: elenca, non spiega. La rielaborazione dei contenuti vive nei quaderni di Valentina.

## Concetti acquisiti
*(ancora nessuno — il percorso inizia qui)*

## Concetti in corso
- Ownership transfer via puntatore consumato e azzerato dalla callee (visto su `pDeviceInit` in `WdfDeviceCreate`, VDD)
- Versioning a runtime vs compile-time (`IDD_IS_FIELD_AVAILABLE`, VDD) — corretta un'interpretazione iniziale errata (protezione memoria)

## Parking lot
*(tangenti parcheggiate: una riga di contesto ciascuna — perché è emersa, dove potrà rientrare)*

—

## Materiale attivo
- **Virtual Display Driver** (VirtualDrivers/Virtual-Display-Driver, branch `master`): lettura guidata del ciclo di vita, un anello alla volta. Posizione attuale: `VirtualDisplayDriverDeviceAdd`. Pipeline funzionante (VS 2026 + WDK + VM Win11 test-signing, cert `MttVDD Test Cert` allineato host/VM). Fili aperti: dove viene chiamata `InitAdapter` (non in `DeviceAdd`); difetto `new IndirectDeviceContext` (fallimento di `operator new` in UMDF). Rimandata a sessione futura: guida alla creazione dei certificati.
- **Calcolatrice** (progetto personale ground-up): da portare in questo repo dal desktop.

## Menù possibile
*(possibilità, non impegni — nessun ordine, nessuna priorità)*

- VDD: riprendere i fili aperti (`InitAdapter`, difetto del `new`)
- VDD: proseguire la lettura del ciclo di vita da dove si è arrivati
- Portare la calcolatrice nel repo e fare il punto sul suo stato
- Prima sessione sui fondamenti: partire dal filo conduttore (memoria, ownership, lifetime) e vedere dove siamo
