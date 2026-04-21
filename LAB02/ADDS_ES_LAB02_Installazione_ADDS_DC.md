# ADDS_ES_LAB02 - Esercitazione autonoma installazione AD DS e promozione del Domain Controller

## Traccia individuale di consolidamento

---

# 1. Obiettivo dell’esercitazione

Questa esercitazione autonoma serve a consolidare ciò che hai fatto nel laboratorio guidato.

Devi dimostrare di saper:

- verificare i prerequisiti di `DC1`
- descrivere correttamente la differenza tra ruolo AD DS e Domain Controller promosso
- installare o verificare il ruolo AD DS
- promuovere `DC1` a primo Domain Controller della foresta `lab.local`
- controllare i principali elementi nati dopo la promozione
- raccogliere evidenze tecniche minime

---

# 2. Scenario

Hai una macchina Windows Server 2022 denominata `DC1`, già predisposta nel LAB00 e progettata nel LAB01.

Il tuo compito è trasformarla nel primo Domain Controller del laboratorio didattico.

Vincoli obbligatori:

- dominio FQDN: `lab.local`
- NetBIOS: `LAB`
- `DC1` deve ospitare anche DNS
- il laboratorio resta su rete `192.168.50.0/24`

---

# 3. Risultato atteso

Al termine devi consegnare un report che contenga:

- verifica iniziale della macchina
- descrizione del ruolo AD DS
- descrizione della promozione
- dati del dominio creato
- verifica di `SYSVOL` e `NETLOGON`
- controlli PowerShell minimi
- verifica DNS minima
- eventuali problemi incontrati

---

# 4. Attività da svolgere

## Attività 1 - Verifica iniziale di DC1

Esegui su `DC1`:

```powershell
hostname
ipconfig /all
Get-NetIPAddress -AddressFamily IPv4
```

### Evidenza richiesta

Annota:

- hostname
- IPv4
- DNS configurato
- nome della rete usata

---

## Attività 2 - Spiega con parole tue che cos’è AD DS

Scrivi un testo breve di 6-10 righe in cui spieghi:

- che cos’è il ruolo AD DS
- perché da solo non basta ancora a creare il dominio
- che cosa aggiunge la promozione a Domain Controller

### Evidenza richiesta

Inserisci il testo nel report finale.

---

## Attività 3 - Installa o verifica il ruolo AD DS

Verifica la presenza del ruolo:

```powershell
Get-WindowsFeature AD-Domain-Services
```

Se necessario installalo:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Evidenza richiesta

Riporta:

- se il ruolo era già presente oppure no
- metodo usato
- esito dell’operazione

---

## Attività 4 - Promuovi DC1 a Domain Controller

Usa la procedura guidata e crea:

- nuova foresta
- dominio `lab.local`

Imposta:

- DNS attivo
- Global Catalog attivo
- no RODC
- password DSRM annotata

### Evidenza richiesta

Riporta nel report:

- FQDN dominio
- NetBIOS
- breve spiegazione della DSRM
- eventuali warning della procedura

---

## Attività 5 - Verifica post-promozione

Dopo il riavvio esegui:

```powershell
Get-SmbShare | Where-Object {$_.Name -in 'NETLOGON','SYSVOL'}
Get-Service NTDS,DNS
```

### Evidenza richiesta

Scrivi:

- se `NETLOGON` è presente
- se `SYSVOL` è presente
- stato sintetico dei servizi `NTDS` e `DNS`

---

## Attività 6 - Verifica il dominio da PowerShell

Esegui:

```powershell
Get-ADDomain
Get-ADForest
```

### Evidenza richiesta

Riporta almeno questi campi:

- `DNSRoot`
- `NetBIOSName`
- nome della foresta

Aggiungi 3-4 righe in cui spieghi che cosa ti dice l’output.

---

## Attività 7 - Verifica DNS minima

Esegui:

```powershell
nslookup dc1.lab.local
nslookup lab.local
```

### Evidenza richiesta

Descrivi se la risoluzione è coerente.

Aggiungi 4-5 righe per spiegare perché il DNS è indispensabile in un dominio AD.

---

## Attività 8 - Diagnostica iniziale

Esegui:

```powershell
dcdiag
```

Salva l’output:

```powershell
New-Item -ItemType Directory -Force C:\LabEvidence
```

```powershell
dcdiag > C:\LabEvidence\dcdiag_lab02.txt
```

### Evidenza richiesta

Nel report indica:

- se hai visto errori o warning evidenti
- una breve interpretazione del risultato generale

---

## Attività 9 - Checkpoint finale

Crea un checkpoint Hyper-V chiamato:

```text
LAB02_DC1_promosso_ADDS_OK
```

### Evidenza richiesta

Conferma nel report di aver creato il checkpoint.

---

# 5. Struttura suggerita del report finale

Crea il file:

```text
C:\LabEvidence\evidence_lab02.md
```

Usa questa struttura:

```md
# Evidence LAB02

## 1. Verifica iniziale di DC1
- hostname
- IP
- DNS
- rete

## 2. Che cos’è AD DS
- spiegazione personale

## 3. Installazione ruolo
- verifica/installazione
- esito

## 4. Promozione a Domain Controller
- dominio creato
- NetBIOS
- DNS
- DSRM
- warning eventuali

## 5. Verifiche post-promozione
- SYSVOL
- NETLOGON
- servizi

## 6. Verifiche PowerShell
- Get-ADDomain
- Get-ADForest

## 7. Verifiche DNS
- nslookup
- commento

## 8. DCDIAG
- sintesi esito

## 9. Checkpoint
- nome checkpoint

## 10. Problemi incontrati
- problemi
- soluzioni
```

---

# 6. Criteri di successo

L’esercitazione è completata con successo se:

- `DC1` è realmente promosso a Domain Controller
- il dominio `lab.local` esiste
- DNS e servizi principali risultano coerenti
- `SYSVOL` e `NETLOGON` sono presenti
- `Get-ADDomain` e `Get-ADForest` funzionano
- il report è compilato in modo tecnico e leggibile

---

# 7. Errori da evitare

- confondere il ruolo AD DS con il dominio già operativo
- dimenticare la DSRM o non annotarla
- ignorare il DNS
- saltare le verifiche post-promozione
- non salvare l’output di `dcdiag`

---

# 8. Conclusione

Questa esercitazione ti porta al primo vero punto di svolta del corso: non hai più una semplice VM Windows Server, ma il **primo Domain Controller** dell’ambiente didattico.

Dal prossimo laboratorio il dominio verrà usato da un client reale, e questo renderà immediatamente visibili sia i punti forti sia gli errori che hai lasciato indietro.
