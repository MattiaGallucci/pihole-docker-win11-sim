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
```

## 📱 Fase 4: Configurazione Dispositivi Mobili
Per estendere la protezione del Pi-hole allo smartphone, il dispositivo deve "vedere" il PC all'interno della rete locale Wi-Fi.

### 1. Trovare l'IP del PC Host
Il telefono non può usare `127.0.0.1`. Deve puntare all'indirizzo IP privato del tuo computer.
Dal terminale, esegui:
```powershell
ipconfig
```
Segnati l'Indirizzo IPv4 (solitamente 192.168.1.xxx).
### 2. Aprire il Firewall di Windows (Cruciale)
Senza questo passaggio, il telefono non riuscirà a connettersi. Esegui questo comando in un Terminale (Amministratore) per permettere le richieste DNS in entrata sulla porta 53:
```powershell
New-NetFirewallRule -DisplayName "Pi-hole DNS" -Direction Inbound -LocalPort 53 -Protocol UDP -Action Allow
```
### 3. Configurazione Smartphone
1. Vai nelle impostazioni Wi-Fi del telefono.
2. Modifica la rete a cui sei connesso:
  - Android: Imposta IP su Statico e inserisci l'IP del PC nel campo DNS 1.
  - iOS: Configura DNS su Manuale e aggiungi l'IP del PC.
3. Importante: Disabilita la funzione "DNS Privato" (Android) o i profili "DNS Sicuro" (iOS), altrimenti il telefono ignorerà il Pi-hole.
### 4. Verifica
Apri la dashboard di Pi-hole sul PC (`http://localhost/admin`) e controlla il Query Log. Dovresti vedere apparire l'IP del tuo telefono nella colonna Client.
