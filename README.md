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
