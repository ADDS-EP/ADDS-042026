# ADDS_LAB02 - Installazione AD DS e promozione del Domain Controller

## Dalla macchina Windows Server al primo dominio

---

# 1. Obiettivo del laboratorio

In questo laboratorio installerai il ruolo **Active Directory Domain Services** su `DC1` e promuoverai il server a **primo Domain Controller** della nuova foresta del corso.

Da questo momento in poi, tutto il resto del corso dipenderà da ciò che costruisci qui:

- join al dominio
- utenti e gruppi
- OU e deleghe
- Group Policy
- DNS integrato con AD
- servizi di rete e file server

Alla fine del laboratorio dovrai essere in grado di:

- verificare che `DC1` abbia i prerequisiti corretti
- installare il ruolo AD DS
- promuovere `DC1` a Domain Controller di una nuova foresta
- configurare correttamente il dominio `lab.local`
- capire il ruolo della password DSRM
- verificare il funzionamento iniziale di AD DS e DNS
- riconoscere cartelle e condivisioni fondamentali come `SYSVOL` e `NETLOGON`
- esplorare i primi oggetti creati automaticamente in Active Directory
- eseguire alcune verifiche di base con strumenti grafici e PowerShell

---

# 2. Scopo di questo laboratorio

Lo scopo non è solo “far comparire Active Directory nel menu Start”.

Lo scopo reale è costruire **in modo consapevole** il primo Domain Controller del laboratorio. Questo significa comprendere:

- quali prerequisiti sono davvero necessari
- perché il DNS è inseparabile da AD DS
- che cosa avviene durante la promozione
- quali componenti nascono o cambiano dopo la promozione
- quali controlli minimi fare prima di dichiarare il server “pronto”

Questo laboratorio prepara direttamente il successivo:

- `CLIENT1` verrà joinato al dominio
- verranno creati utenti, gruppi e OU
- verranno applicate GPO e deleghe

Se qui sbagli DNS, naming, IP o promozione, i problemi emergeranno più avanti sotto forma di errori poco eleganti e molto ripetitivi.

---

# 3. Architettura del laboratorio in questa sessione

In questa sessione devono essere accese solo queste VM:

- `DC1`
- `SRV1` (facoltativa per test rete)
- `CLIENT1` (spenta oppure accesa solo per verifica connettività)

VM da tenere spente:

- `CLU1`
- `CLU2`

## 3.1 Stato atteso di DC1 prima del laboratorio

`DC1` deve essere già pronta dal LAB00 con:

- Windows Server 2022 installato
- hostname corretto: `DC1`
- IP statico: `192.168.50.10/24`
- nessun gateway necessario nel lab
- DNS iniziale configurato in modo coerente
- rete collegata allo switch `vLab-ADDS`

## 3.2 Dominio che verrà creato

Useremo lo standard del corso:

- **FQDN dominio**: `lab.local`
- **NetBIOS**: `LAB`

`DC1` sarà il primo server del dominio e, inizialmente, ospiterà anche il DNS.

---

# Parte 1 - Spiegazione dei concetti e degli strumenti

# 4. Che cos’è il ruolo AD DS

Il ruolo **Active Directory Domain Services** trasforma un server Windows in un nodo capace di ospitare il database della directory, rispondere come Domain Controller e fornire i servizi necessari alla gestione centralizzata delle identità.

Detto in modo semplice ma corretto:

- prima dell’installazione `DC1` è solo un server Windows
- dopo l’installazione del ruolo ha i componenti necessari per diventare DC
- dopo la promozione entra davvero in funzione come Domain Controller

Questa distinzione è importante:

- **installare il ruolo** non significa ancora avere un dominio funzionante
- **promuovere il server** significa creare o integrare il dominio/foresta e attivare realmente la directory

---

# 5. Che cosa significa promuovere un server a Domain Controller

Promuovere un server a DC significa assegnargli il ruolo operativo di Domain Controller.

Nel nostro caso `DC1` diventerà:

- primo DC del dominio `lab.local`
- primo DC della foresta `lab.local`
- server DNS del dominio
- nodo che ospita il database iniziale AD

Durante la promozione Windows:

- installa e configura i componenti AD DS
- crea il database della directory
- crea `SYSVOL`
- configura le condivisioni necessarie
- prepara la struttura del nuovo dominio e della nuova foresta
- registra record DNS necessari alla localizzazione del Domain Controller

---

# 6. Perché il DNS è fondamentale

Questa è la parte che nei laboratori viene spesso sottovalutata, poi tutti passano un’ora a chiedersi perché il join non funziona.

In un dominio AD:

- i client trovano il Domain Controller tramite DNS
- i servizi di directory pubblicano record specifici nel DNS
- le autenticazioni e molte operazioni amministrative si appoggiano alla risoluzione dei nomi

Quindi:

- il DNS non è un accessorio
- il DNS non è “qualcosa che poi sistemiamo”
- il DNS è una parte strutturale del laboratorio

Nel nostro scenario, `DC1` ospiterà sia:

- **AD DS**
- **DNS**

---

# 7. Nuova foresta o dominio aggiuntivo?

In questo laboratorio creeremo una **nuova foresta**.

Questo significa che:

- non esiste ancora nessuna struttura AD precedente
- `DC1` sarà il primo Domain Controller dell’ambiente
- il dominio `lab.local` sarà anche la radice della foresta

Questa scelta è coerente con un corso introduttivo e con il laboratorio che stiamo costruendo.

Non stiamo:

- aggiungendo un secondo DC a un dominio esistente
- creando un child domain
- creando una trust tra foreste

Queste sono evoluzioni possibili, ma non servono in questo punto del percorso.

---

# 8. Che cos’è la password DSRM

Durante la promozione Windows richiede una password per **Directory Services Restore Mode**.

La DSRM è una modalità speciale di avvio utile per attività di manutenzione o ripristino di Active Directory.

Nel laboratorio ti basta ricordare questo:

- non è una password “ornamentale”
- non coincide automaticamente con altre credenziali del dominio
- va annotata in modo sicuro, perché potrebbe servire in scenari di recovery

Per il lab usa una password coerente con la policy del corso, meglio non improvvisare stringhe che poi dimentichi.

---

# 9. Che cosa nasce dopo la promozione

Dopo la promozione a Domain Controller compariranno diversi elementi importanti.

## 9.1 Oggetti logici iniziali

In AD vedrai da subito:

- il dominio `lab.local`
- container di default come `Users` e `Computers`
- gruppi built-in
- account e gruppi amministrativi standard

## 9.2 Componenti di file system e condivisioni

Compariranno o verranno configurati:

- cartella `SYSVOL`
- condivisione `SYSVOL`
- condivisione `NETLOGON`

Questi elementi diventeranno molto importanti nelle sessioni su:

- Group Policy
- script di logon
- replica tra Domain Controller (quando parleremo di evoluzioni più avanzate)

## 9.3 DNS integrato

Vedrai anche:

- zona DNS del dominio
- record SRV e altri record necessari
- integrazione fra directory e DNS

---

# 10. Strumenti che useremo

In questo laboratorio useremo sia strumenti grafici sia comandi.

## 10.1 Strumenti grafici

- **Server Manager**
- **Active Directory Users and Computers**
- **DNS Manager**

## 10.2 Strumenti a riga di comando / PowerShell

- `hostname`
- `ipconfig /all`
- `ping`
- `nslookup`
- `dcdiag`
- `Get-ADDomain`
- `Get-ADForest`
- `Get-Service`
- `Get-SmbShare`

L’obiettivo non è fare PowerShell avanzata, ma imparare a verificare il laboratorio senza dipendere solo dall’interfaccia grafica.

---

# 11. Prerequisiti obbligatori

Prima di iniziare, verifica che `DC1` abbia davvero questi prerequisiti.

## 11.1 Prerequisiti di naming e rete

- hostname = `DC1`
- IP statico = `192.168.50.10`
- subnet mask coerente con `/24`
- DNS iniziale coerente
- rete collegata a `vLab-ADDS`

## 11.2 Prerequisiti di stabilità

- data e ora plausibili
- nessun riavvio in sospeso
- spazio disco sufficiente
- credenziali amministrative locali disponibili

## 11.3 VM da tenere spente

- `CLU1`
- `CLU2`

Il laboratorio non richiede il cluster acceso. Non stiamo cercando di impressionare il Task Manager.

---

# Parte 2 - Step-by-step guidato

# 12. Verifica preliminare di DC1

Accedi a `DC1` come amministratore locale.

Apri PowerShell ed esegui:

```powershell
hostname
ipconfig /all
Get-NetIPAddress -AddressFamily IPv4
```

## 12.1 Controlli da fare

Verifica che:

- il nome host sia `DC1`
- l’IP sia `192.168.50.10`
- la scheda corretta sia sulla rete `vLab-ADDS`
- non ci siano configurazioni strane ereditate da prove precedenti

### Evidenza richiesta

Annota nel report:

- hostname
- indirizzo IPv4
- DNS configurato
- nome della scheda di rete

---

# 13. Test di rete minima

Se `SRV1` o `CLIENT1` sono accese, puoi verificare connettività locale.

Da `DC1` esegui:

```powershell
ping 192.168.50.20
ping 192.168.50.100
```

Se le altre macchine sono spente, non è un problema.

Lo scopo di questo passaggio è solo verificare che la rete del laboratorio sia coerente.

---

# 14. Installazione del ruolo AD DS

Apri **Server Manager**.

## 14.1 Procedura grafica

1. Vai su **Manage**
2. Seleziona **Add Roles and Features**
3. Procedi con **Role-based or feature-based installation**
4. Seleziona il server locale `DC1`
5. Seleziona il ruolo **Active Directory Domain Services**
6. Accetta le feature aggiuntive richieste
7. Prosegui fino alla conferma
8. Avvia l’installazione

## 14.2 Cosa osservare

Durante l’installazione:

- Windows prepara i componenti necessari
- non è ancora stata creata alcuna foresta
- il server non è ancora un Domain Controller operativo

### Evidenza richiesta

Annota nel report:

- schermata o descrizione del ruolo selezionato
- eventuali feature aggiuntive installate
- esito dell’installazione

---

# 15. Installazione del ruolo via PowerShell (facoltativa ma utile)

Se vuoi fare anche la verifica da PowerShell:

```powershell
Get-WindowsFeature AD-Domain-Services
```

Per installare il ruolo da PowerShell:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

## 15.1 Perché è utile saperlo

Perché ti permette di:

- verificare rapidamente se il ruolo è presente
- ripetere l’operazione in modo più controllato
- prendere confidenza con una modalità amministrativa usata spesso in contesti reali

Non è obbligatorio usare sempre PowerShell, ma ignorarla del tutto sarebbe un peccato didattico.

---

# 16. Avvio della promozione a Domain Controller

Una volta terminata l’installazione del ruolo, in Server Manager comparirà il messaggio per la configurazione post-deployment.

## 16.1 Procedura

1. In Server Manager clicca sulla notifica post-installazione
2. Scegli **Promote this server to a domain controller**
3. Seleziona **Add a new forest**
4. Inserisci il dominio:

```text
lab.local
```

5. Prosegui alla schermata delle opzioni del Domain Controller

---

# 17. Configurazione delle opzioni del Domain Controller

In questa fase dovrai confermare alcuni parametri importanti.

## 17.1 Parametri da impostare

- Domain Name System (DNS) server: **attivo**
- Global Catalog (GC): **attivo**
- Read Only Domain Controller (RODC): **non attivo**
- Password DSRM: impostare una password nota e annotata

## 17.2 Note didattiche

Nel nostro laboratorio:

- `DC1` è il primo Domain Controller, quindi sarà anche Global Catalog
- non stiamo creando un RODC
- il DNS deve essere installato insieme al ruolo

### Evidenza richiesta

Annota nel report:

- perché il DNS è stato selezionato
- che cos’è la password DSRM in parole tue

---

# 18. Controllo del nome NetBIOS

Durante la procedura guidata Windows proporrà un nome NetBIOS coerente.

Nel nostro caso deve risultare:

```text
LAB
```

Se il wizard propone questo valore, lascialo.

---

# 19. Percorsi di database, log e SYSVOL

Il wizard mostrerà i percorsi predefiniti per:

- database AD
- file di log
- cartella `SYSVOL`

Nel laboratorio puoi lasciare i percorsi di default.

## 19.1 Perché lasciamo i default

Perché in questa fase vogliamo:

- capire la procedura
- ottenere un laboratorio funzionante
- non introdurre complessità infrastrutturali premature

In ambienti enterprise reali si valutano anche:

- separazione dei volumi
- prestazioni storage
- ridondanza
- backup

Ma qui non serve complicarsi la vita senza guadagno didattico.

---

# 20. Verifica prerequisiti e installazione finale

Procedi con la verifica prerequisiti del wizard.

## 20.1 Cosa aspettarti

Potrebbero comparire avvisi non bloccanti, per esempio su aspetti di naming o DNS in ambienti isolati di laboratorio.

L’importante è distinguere:

- **warning**: segnalazioni da capire
- **error**: problemi che bloccano davvero la promozione

Quando i prerequisiti sono superati, avvia la promozione.

Il server verrà riavviato automaticamente.

### Evidenza richiesta

Riporta nel report:

- eventuali warning visualizzati
- esito della verifica prerequisiti
- conferma del riavvio del server

---

# 21. Primo accesso dopo la promozione

Dopo il riavvio, accedi nuovamente a `DC1`.

Osserva che ora il contesto del logon è cambiato.

## 21.1 Cosa verificare

- il dominio `LAB` o `lab.local` compare nel contesto di accesso
- il server è ora un Domain Controller
- Server Manager mostra i nuovi strumenti e servizi collegati ad AD

### Nota didattica

Questo è il primo momento in cui puoi percepire davvero la differenza fra:

- server standalone
- server appartenente e fondante un dominio

---

# 22. Verifica delle condivisioni fondamentali

Apri PowerShell ed esegui:

```powershell
Get-SmbShare | Where-Object {$_.Name -in 'NETLOGON','SYSVOL'}
```

Oppure da prompt:

```cmd
net share
```

## 22.1 Risultato atteso

Devi vedere le condivisioni:

- `NETLOGON`
- `SYSVOL`

Se non compaiono, fermati e verifica la promozione. Queste condivisioni non sono dettagli decorativi.

---

# 23. Verifica delle cartelle di sistema AD

Controlla almeno la presenza di:

- `C:\Windows\SYSVOL`
- struttura coerente con `SYSVOL`

Non serve esplorare tutto in profondità, ma è utile collegare la teoria a ciò che esiste davvero nel file system.

---

# 24. Apertura di Active Directory Users and Computers

Apri:

- **Active Directory Users and Computers**

## 24.1 Cosa osservare

Verifica la presenza di:

- dominio `lab.local`
- container `Users`
- container `Computers`
- unità e gruppi built-in

## 24.2 Domande guida

Mentre osservi la console, chiediti:

- quali oggetti sono stati creati automaticamente?
- che differenza c’è fra container e OU?
- dove finiranno per default i computer joinati, se non cambiamo nulla?

Queste domande ti torneranno utili nel LAB04.

---

# 25. Apertura di DNS Manager

Apri:

- **DNS Manager**

## 25.1 Cosa osservare

Controlla:

- presenza della zona `lab.local`
- presenza dei record necessari
- struttura generale delle zone forward

## 25.2 Obiettivo didattico

L’obiettivo non è ancora diventare esperto DNS. L’obiettivo è vedere che il DNS del dominio è parte integrante del laboratorio AD e non un servizio separato “messo lì per caso”.

---

# 26. Verifica dei servizi principali

In PowerShell esegui:

```powershell
Get-Service NTDS,DNS
```

## 26.1 Risultato atteso

I servizi devono risultare presenti e in stato coerente con il funzionamento del DC.

Puoi anche verificare:

```powershell
Get-Service | Where-Object {$_.DisplayName -like '*Active Directory*' -or $_.Name -like 'DNS*'}
```

---

# 27. Prime verifiche PowerShell sul dominio

Apri PowerShell ed esegui:

```powershell
Get-ADDomain
Get-ADForest
```

## 27.1 Che cosa osservare

Controlla almeno:

- nome del dominio
- NetBIOSName
- DistinguishedName
- nome della foresta
- Domain Controllers della foresta o del dominio

### Evidenza richiesta

Riporta nel report:

- `DNSRoot`
- `NetBIOSName`
- nome della foresta

---

# 28. Diagnostica iniziale con DCDIAG

Esegui:

```powershell
dcdiag
```

## 28.1 Perché farlo già ora

Perché ti abitua a verificare il Domain Controller con uno strumento diagnostico reale.

Non devi interpretare ogni dettaglio come un consulente forense del dominio, ma devi imparare a:

- lanciare il comando
- vedere se ci sono errori grossolani
- conservare l’output come evidenza tecnica

### Evidenza richiesta

Salva l’output in un file, per esempio:

```powershell
dcdiag > C:\LabEvidence\dcdiag_lab02.txt
```

Se la cartella non esiste, creala prima.

---

# 29. Verifica di risoluzione nome locale

Esegui:

```powershell
nslookup dc1.lab.local
nslookup lab.local
```

## 29.1 Risultato atteso

Il server deve essere in grado di risolvere i nomi del dominio in modo coerente.

Questo non esaurisce il tema DNS, ma è un primo controllo minimo sensato.

---

# 30. Checkpoint del laboratorio

Quando il Domain Controller funziona e le verifiche principali sono state eseguite, crea un checkpoint Hyper-V con nome chiaro.

Nome suggerito:

```text
LAB02_DC1_promosso_ADDS_OK
```

## 30.1 Perché è utile

Perché ti permette di:

- tornare rapidamente a uno stato valido
- evitare di rifare tutta la promozione da zero
- avere una base affidabile per i laboratori successivi

---

# 31. Troubleshooting minimo

## 31.1 Il wizard non consente la promozione

Possibili cause:

- hostname non corretto
- rete incoerente
- riavvio in sospeso
- installazione ruolo non completata

Verifica:

```powershell
hostname
ipconfig /all
Get-WindowsFeature AD-Domain-Services
```

---

## 31.2 DNS configurato in modo errato

Possibili sintomi:

- risoluzione nomi incoerente
- errori in `nslookup`
- problemi in verifiche post-promozione

Controlla la configurazione IP/DNS del server:

```powershell
ipconfig /all
```

---

## 31.3 Dopo il riavvio non vedi SYSVOL o NETLOGON

Possibili cause:

- promozione non andata a buon fine
- accesso non completato correttamente
- servizi non ancora stabilizzati

Controlla:

```powershell
Get-SmbShare
Get-Service NTDS,DNS
```

Se il problema persiste, non andare avanti con il corso facendo finta di niente.

---

## 31.4 `Get-ADDomain` non funziona

Possibili cause:

- modulo AD non disponibile
- ruolo non installato correttamente
- PowerShell aperta prima del completamento della configurazione

Prova a chiudere e riaprire PowerShell come amministratore e verifica il ruolo:

```powershell
Get-WindowsFeature AD-Domain-Services
```

---

## 31.5 `dcdiag` mostra errori

Non tutti i messaggi hanno la stessa gravità.

In questo punto del corso devi soprattutto distinguere tra:

- warning non bloccanti
- errori reali di DNS, servizi o promozione

Annota i risultati nel report e, se necessario, ripeti con calma le verifiche di base.

---

# 32. Evidenze richieste

Crea il file:

```text
C:\LabEvidence\evidence_lab02.md
```

Inserisci una struttura simile a questa:

```md
# Evidence LAB02

## 1. Verifica iniziale di DC1
- hostname
- IP
- DNS
- note sulla rete

## 2. Installazione ruolo AD DS
- metodo usato
- esito installazione
- feature aggiuntive installate

## 3. Promozione a Domain Controller
- nuova foresta creata
- dominio FQDN
- NetBIOS
- DSRM impostata
- eventuali warning

## 4. Verifiche post-promozione
- presenza SYSVOL
- presenza NETLOGON
- console ADUC aperta correttamente
- zona DNS visibile

## 5. Verifiche PowerShell
- output sintetico di Get-ADDomain
- output sintetico di Get-ADForest
- esito dcdiag

## 6. Verifiche DNS
- risultati di nslookup

## 7. Checkpoint creato
- nome checkpoint

## 8. Problemi incontrati
- problemi
- cause ipotizzate
- soluzione adottata
```

---

# 33. Consegna

Al termine del laboratorio devi consegnare:

- screenshot o note delle principali fasi di installazione/promozione
- file `evidence_lab02.md`
- file `dcdiag_lab02.txt`

Se stai lavorando in repository locale per il corso, salva anche lì la documentazione del laboratorio.

---

# 34. Conclusione del laboratorio

In questo laboratorio hai trasformato `DC1` da semplice server Windows in **primo Domain Controller** della foresta `lab.local`.

Hai quindi:

- installato il ruolo AD DS
- creato la nuova foresta
- attivato il DNS del dominio
- verificato i componenti fondamentali del DC
- ottenuto la base tecnica necessaria per i passi successivi

Dal laboratorio seguente non parleremo più di un server isolato. Parleremo di un ambiente di dominio vero, con tutto il carico di ordine e responsabilità che questo comporta.


