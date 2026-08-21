# Architettura driver Windows — quadro d'insieme

Riferimento rapido costruito durante lo studio di VDD. Elenca, non spiega.

## Concetto chiave

Un driver non è un programma: è una libreria. Qualcun altro lo carica, qualcun altro decide quando chiamare le tue funzioni. Tu scrivi solo le risposte alle domande che il sistema ti fa.

## I tre strati

```
Windows (kernel, PnP manager, power manager)
        ↓ eventi
WDF — Windows Driver Framework (generico, qualsiasi driver)
        ↓ callback
IddCx — Indirect Display Driver Class Extension (specifico per display)
        ↓ callback
Il tuo codice (le callback che scrivi)
```

**WDF** è generico: sa cos'è un device, sa gestire accensione/spegnimento, sa fare cleanup. Non sa nulla di monitor o frame.

**IddCx** è specifico per display indiretti: aggiunge adapter, monitor, risoluzioni, frame dal compositor di Windows. "Indirect" perché non c'è hardware grafico vero.

Tu non parli mai direttamente con Windows. La catena è sempre Windows → WDF + IddCx → il tuo codice. Tutto gira dentro `WUDFHost.exe` (driver UMDF = user mode).

## Ciclo di vita di un driver IddCx

Sei stadi, sempre gli stessi, sempre in quest'ordine.

| # | Stadio | Chi chiama | Cosa succede | In VDD |
|---|--------|-----------|--------------|--------|
| 1 | Registrazione | Windows | Il sistema carica la DLL, chiama la entry point. Tu registri le callback. | `DriverEntry` (~r.2405) |
| 2 | Configurazione device | PnP manager | Il sistema trova un device corrispondente al tuo driver (via INF). Ti chiede di configurarlo. Tu crei il device object e ci attacchi il contesto. | `DeviceAdd` (~r.2812) |
| 3 | Accensione | Power manager | Il device si accende. Tu inizializzi l'adapter (punto di contatto WDF → IddCx). | `D0Entry` → `InitAdapter` |
| 4 | Configurazione display | IddCx | Il framework ti chiede di configurare i monitor e dichiarare le risoluzioni supportate. | `CreateMonitor`, `GetModes` |
| 5 | Funzionamento | IddCx | Arrivano i frame dal compositor, tu li ricevi e li processi. | `AssignSwapChain` |
| 6 | Teardown | PnP manager | Il device viene rimosso. Pulizia e rilascio risorse. | `EvtCleanupCallback` |

Gli stadi 1, 2, 6 sono WDF (generici). Gli stadi 3, 4, 5 sono IddCx (specifici display).

## Meccanismo delle callback

"Scrivere un driver" = due cose:

1. **Registrare le callback**: riempire i campi di una struct con puntatori alle tue funzioni (es. `PnpPowerCallbacks.EvtDeviceD0Entry = ...`). Dici al framework *quali* funzioni chiamare.
2. **Scrivere il corpo delle callback**: decidi *cosa fare* quando vengono chiamate.

## I sei anelli dello sviluppo

1. **Cosa scrivi** — codice sorgente (callback, entry point)
2. **Cosa dichiari** — file INF (dice a Windows "questo driver esiste, gestisce questo tipo di hardware")
3. **Cosa compili** — MSBuild + WDK → una DLL (non un .exe)
4. **Cosa firmi** — certificato + catalogo (Windows non carica driver non firmati)
5. **Cosa installi** — il sistema legge l'INF e registra il driver
6. **Cosa gira** — `WUDFHost.exe` carica la tua DLL, il framework chiama le tue callback
