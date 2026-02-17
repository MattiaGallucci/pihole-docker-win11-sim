# pihole-docker-win11-sim
Guida alla simulazione di Pi-hole su Windows 11 tramite Docker, con risoluzione problemi per router TIM e configurazione mobile.


# Pi-hole su Windows 11 con Docker Desktop

Questo repository documenta la configurazione completa di un'istanza Pi-hole in ambiente simulato su Windows 11. La guida include la risoluzione di problemi critici legati ai router TIM (suffissi DNS) e la configurazione per dispositivi mobili.

## 🛠️ Prerequisiti
- **Windows 11** con WSL2 installato.
- **Docker Desktop** avviato e funzionante.
- Accesso amministrativo al terminale (PowerShell/Terminale).

## 🚀 Fase 1: Configurazione Docker
Il cuore del progetto è il file `docker-compose.yml`. Questo file permette di creare il container con tutte le impostazioni necessarie senza dover scrivere lunghi comandi ogni volta.

### Il file Docker Compose spiegato:
- **Image**: `pihole/pihole:latest` (usa l'ultima versione ufficiale).
- **Network Mode**: `bridge` (permette a Windows di comunicare con il container tramite la porta 53).
- **Environment Variables**:
  - `WEBPASSWORD`: La tua password per il pannello admin.
  - `BLOCKINGMODE`: Impostato su `NULL` (fondamentale per evitare loop di indirizzi `127.0.0.1` su Windows).
  - `FTLCONF_LOCAL_IPV4`: Indica l'IP del server DNS.

## 🔐 Fase 2: Gestione Password e Primo Accesso
Spesso la password definita nel file `docker-compose.yml` (`WEBPASSWORD`) potrebbe non essere recepita correttamente al primo avvio. 

### Risoluzione errore Login:
Se non riesci ad accedere all'interfaccia web (`http://localhost/admin`), non provare a indovinare la password. Usa il metodo **interattivo** che forza il reset direttamente nel container:

1. Apri il terminale nella cartella del progetto.
2. Esegui il comando:
   ```powershell
   docker exec -it pihole pihole setpassword


## 🌐 Fase 3: Risoluzione del "Suffix DNS" (Problema Router TIM)
Uno dei problemi più complessi riscontrati è stato il **DNS Hijacking** o l'inserimento automatico del suffisso da parte del router TIM. 

### Il problema:
Eseguendo un `nslookup`, Windows aggiungeva automaticamente il suffisso `.homenet.telecomitalia.it` a ogni query (es. `google.com.homenet.telecomitalia.it`), causando il fallimento della risoluzione o risposte errate (`127.0.0.1`).

### La soluzione definitiva:
Per risolvere, abbiamo agito su più fronti:

1. **Modifica sul Router**: Accesso alla pagina di configurazione del modem TIM (`192.168.1.1`) e impostazione manuale dei server DNS (es. Google 8.8.8.8) per evitare l'iniezione del suffisso.
2. **Pulizia Cache Windows**:
   ```powershell
   ipconfig /flushdns
3. **Rimozione forzata via PowerShell**:
   ```powershell
   Set-DnsClientGlobalSetting -SuffixSearchList @()

### Test di verifica:
Per confermare il corretto funzionamento, il comando `nslookup` deve restituire l'IP reale e non l'indirizzo di loopback per i siti validi:
```powershell
# Risultato atteso per un sito lecito
nslookup google.com 127.0.0.1 -> Address: 142.251.x.x

# Risultato atteso per un sito bloccato
nslookup doubleclick.net 127.0.0.1 -> Address: 0.0.0.0

