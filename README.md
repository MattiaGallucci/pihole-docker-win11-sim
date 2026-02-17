# pihole-docker-win11-sim
Guida alla simulazione di Pi-hole su Windows 11 tramite Docker, con risoluzione problemi per router TIM e configurazione mobile.

# Pi-hole su Windows 11 con Docker Desktop

Questo repository documenta la configurazione completa di un'istanza Pi-hole in ambiente simulato su Windows 11. La guida include la risoluzione di problemi critici legati ai router TIM (suffissi DNS) e la configurazione per dispositivi mobili.

---

## 🚀 Guida Rapida: Avvio del Progetto

### Prerequisiti
Prima di iniziare, assicurati di avere:

1. **Windows 11** installato
2. **WSL2** abilitato su Windows
   - Apri PowerShell come Amministratore ed esegui:
   ```powershell
   wsl --install
   ```
   - Riavvia il PC se richiesto

3. **Docker Desktop** installato e avviato
   - Scarica da: https://www.docker.com/products/docker-desktop
   - Dopo l'installazione, assicurati che Docker Desktop sia in esecuzione (icona nella system tray)

4. **Accesso amministrativo** al terminale (PowerShell o Windows Terminal)

---

### Passo 1: Clonare il Repository

Apri il terminale e clona questo repository:

```powershell
git clone https://github.com/MattiaGallucci/pihole-docker-win11-sim.git
cd pihole-docker-win11-sim
```

Se non hai Git installato, puoi scaricare il repository come ZIP e estrarlo.

---

### Passo 2: Modificare la Password (Opzionale)

Apri il file `docker-compose.yml` con un editor di testo e modifica la password di default:

```yaml
WEBPASSWORD: 'TuaPasswordSicura'  # Sostituisci con la tua password
```

---

### Passo 3: Avviare Pi-hole

Dalla cartella del progetto, esegui:

```powershell
docker-compose up -d
```

Questo comando:
- Scarica l'immagine Pi-hole (se non presente)
- Crea il container
- Avvia Pi-hole in background (`-d` = detached mode)

---

### Passo 4: Verificare che Pi-hole sia Attivo

Controlla lo stato del container:

```powershell
docker ps
```

Dovresti vedere il container `pihole` in esecuzione.

---

### Passo 5: Accedere all'Interfaccia Web

Apri il browser e vai su:

```
http://localhost/admin
```

**Problema con il login?** Se la password non funziona, vai alla **Fase 2** più sotto per il reset.

---

### Passo 6: Configurare Windows per Usare Pi-hole

1. Apri **Impostazioni** → **Rete e Internet** → **Proprietà** (della tua connessione attiva)
2. Clicca su **Modifica** accanto a "Assegnazione server DNS"
3. Seleziona **Manuale**
4. Attiva **IPv4**
5. Inserisci come DNS preferito: `127.0.0.1`
6. Salva

---

### Passo 7: Test di Funzionamento

Apri PowerShell ed esegui:

```powershell
nslookup google.com 127.0.0.1
```

**Risultato atteso:** Dovresti vedere un indirizzo IP reale (es. `142.251.x.x`)

Per testare un sito bloccato:

```powershell
nslookup doubleclick.net 127.0.0.1
```

**Risultato atteso:** Dovresti vedere `0.0.0.0`

---

### Passo 8: Aggiungere Liste di Blocco

1. Vai su `http://localhost/admin`
2. Nel menu laterale: **Adlists**
3. Aggiungi questa lista consigliata:
   ```
   https://v.firebog.net/hosts/lists.php?type=tick
   ```
4. Vai su **Tools** → **Update Gravity**
5. Clicca **Update**

Ora hai Pi-hole completamente funzionante! 🎉

---

## 🛠️ Comandi Utili

| Comando | Descrizione |
|---------|-------------|
| `docker-compose up -d` | Avvia Pi-hole |
| `docker-compose down` | Ferma e rimuove il container |
| `docker-compose restart` | Riavvia Pi-hole |
| `docker-compose logs -f` | Visualizza i log in tempo reale |
| `docker exec -it pihole pihole setpassword` | Reset password interattivo |

---

## 🔐 Fase 2: Gestione Password e Primo Accesso
Spesso la password definita nel file `docker-compose.yml` (`WEBPASSWORD`) potrebbe non essere recepita correttamente al primo avvio. 

### Risoluzione errore Login:
Se non riesci ad accedere all'interfaccia web (`http://localhost/admin`), non provare a indovinare la password. Usa il metodo **interattivo** che forza il reset direttamente nel container:

1. Apri il terminale nella cartella del progetto.
2. Esegui il comando:
   ```powershell
   docker exec -it pihole pihole setpassword
   ```
3. Inserisci la nuova password quando richiesto
4. Riprova ad accedere all'interfaccia web

---

## 🌐 Fase 3: Risoluzione del "Suffix DNS" (Problema Router TIM)
Uno dei problemi più complessi riscontrati è stato il **DNS Hijacking** o l'inserimento automatico del suffisso da parte del router TIM. 

### Il problema:
Eseguendo un `nslookup`, Windows aggiungeva automaticamente il suffisso `.homenet.telecomitalia.it` a ogni query (es. `google.com.homenet.telecomitalia.it`), causando il fallimento della risoluzione o risposte errate (`127.0.0.1`).

### La soluzione definitiva:
Per risolvere, abbiamo agito su più fronti:

1. **Modifica sul Router**: Accesso alla pagina di configurazione del modem TIM (`192.168.1.1`) e rimozione di `.homenet.telecomitalia.it` per evitare l'iniezione del suffisso.
2. **Pulizia Cache Windows**:
   ```powershell
   ipconfig /flushdns
   ipconfig /release
   ipconfig /renew
   ```
3. **Rimozione forzata via PowerShell**:
   ```powershell
   Set-DnsClientGlobalSetting -SuffixSearchList @()
   ```

### Test di verifica:
Per confermare il corretto funzionamento, il comando `nslookup` deve restituire l'IP reale e non l'indirizzo di loopback per i siti validi:
```powershell
# Risultato atteso per un sito lecito
nslookup google.com 127.0.0.1 -> Address: 142.251.x.x

# Risultato atteso per un sito bloccato
nslookup doubleclick.net 127.0.0.1 -> Address: 0.0.0.0
```

---

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

---

## 🛡️ Fase 5: Liste di Blocco (Adlists) e Manutenzione
Un Pi-hole senza liste è come un buttafuori senza una lista degli invitati. Per bloccare davvero la pubblicità e il tracciamento, dobbiamo "istruirlo".

### 1. Dove trovare le liste
Il punto di riferimento è **The Firebog**. Consigliamo di iniziare con le liste "Tick" (quelle testate che non rompono la navigazione):
- [The Firebog - Non-checking Lists](https://v.firebog.net/hosts/lists.php?type=tick)

### 2. Come aggiungere una lista
1. Accedi al pannello admin: `http://localhost/admin`.
2. Vai su **Adlists** nel menu a sinistra.
3. Incolla l'URL della lista nel campo **Address**.
4. Clicca su **Add**.

### 3. Aggiornare la "Gravity" (Fondamentale) ⚠️
Dopo aver aggiunto una lista, Pi-hole deve scaricare fisicamente i domini per renderli attivi.
1. Vai su **Tools** -> **Update Gravity**.
2. Clicca sul pulsante **Update**.
3. Una volta terminato, vedrai il numero di "Domains on Adlist" salire vertiginosamente nella Dashboard.

### 4. Backup e Ripristino
Se hai intenzione di reinstallare tutto o spostarti su un Raspberry Pi fisico in futuro, usa la funzione **Teleporter**:
- **Settings** -> **Teleporter** -> **Export**.
Questo genererà un file compresso con tutta la tua configurazione (liste, whitelist, impostazioni DNS).

---

## 🐛 Risoluzione Problemi Comuni

### Pi-hole non si avvia
```powershell
# Controlla i log
docker-compose logs

# Verifica che Docker Desktop sia in esecuzione
```

### La porta 53 è già in uso
```powershell
# Ferma il servizio DNS di Windows (se presente)
Stop-Service -Name "Dnscache"
```

### Non riesco a raggiungere internet dopo aver configurato DNS
1. Rimuovi temporaneamente il DNS personalizzato dalle impostazioni di rete
2. Controlla che Pi-hole sia in esecuzione: `docker ps`
3. Verifica i log: `docker-compose logs -f`

---

## 📝 Conclusioni
Questa configurazione simulata su Windows 11 dimostra che è possibile avere un controllo totale sul proprio traffico DNS anche senza hardware dedicato, superando i limiti imposti dai router commerciali (come quelli TIM).

---

## 📚 Risorse Utili
- [Documentazione ufficiale Pi-hole](https://docs.pi-hole.net/)
- [The Firebog - Liste di blocco](https://firebog.net/)
- [Docker Desktop per Windows](https://docs.docker.com/desktop/windows/install/)
- [Guida WSL2](https://learn.microsoft.com/it-it/windows/wsl/install)
