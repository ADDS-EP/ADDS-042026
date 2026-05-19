# ADDS_LAB08 - DNS avanzato per Active Directory

## Dal DNS "che sembra andare" al DNS che determina davvero autenticazione, logon, GPO e reperibilità dei servizi di dominio

---

# 1. Obiettivo del laboratorio

In questo laboratorio affronterai il tema **DNS avanzato in ambiente Active Directory** nel dominio `lab.local`.

Non farai una semplice rassegna di record o una visita guidata alla console DNS. Costruirai invece una comprensione operativa del perché, in un dominio Windows, il DNS non sia un servizio accessorio ma il meccanismo che consente a client, server e controller di dominio di:

- trovare il Domain Controller corretto
- localizzare servizi LDAP e Kerberos
- eseguire il logon di dominio
- applicare GPO e script
- registrare dinamicamente i nomi host
- mantenere coerenza tra identità di rete e servizi di directory

Al termine del laboratorio dovrai essere in grado di:

- spiegare perché **Active Directory dipende dal DNS**
- distinguere una **zona DNS standard** da una **zona integrata in AD**
- leggere e interpretare i record **SRV** fondamentali per il dominio
- verificare e forzare la **registrazione dinamica** di un client
- usare in modo ragionato strumenti come:
  - `nslookup`
  - `Resolve-DnsName`
  - `ipconfig /registerdns`
  - `dcdiag /test:dns`
  - `nltest /dsgetdc`
- simulare almeno un problema DNS realistico e diagnosticarlo
- collegare sintomo e causa in scenari di logon o localizzazione del DC

---

# 2. Scopo del laboratorio

Lo scopo reale di questa sessione è far superare ai partecipanti una delle illusioni più diffuse nei laboratori Windows:

- “se il ping risponde, allora il DNS va bene”

Non basta.

In Active Directory, il DNS non serve solo a risolvere `dc1.lab.local` in un indirizzo IP. Serve soprattutto a dire ai client:

- dove si trova un controller di dominio
- dove si trova il servizio Kerberos
- dove si trovano i servizi LDAP del dominio
- quale server è autorevole per il naming della directory

Se questi meccanismi non sono chiari, molti problemi futuri vengono letti male:

- join dominio che fallisce
- logon lento o impossibile
- GPO che non si applicano
- secure channel instabile
- errori poco leggibili ma tutti con la stessa radice: **DNS configurato male**

Questo laboratorio quindi non è “solo DNS”. È una sessione di lettura strutturale di Active Directory attraverso il suo servizio di naming.

---

# 3. Architettura del laboratorio

## 3.1 VM da tenere accese

- `DC1`
- `CLIENT1`
- `SRV1`

## 3.2 VM da tenere spente

- `CLU1`
- `CLU2`

## 3.3 Prerequisiti

Si assume che siano già completati:

- **LAB00** setup ambiente Hyper-V
- **LAB01** architettura e progettazione ADDS
- **LAB02** installazione AD DS e promozione Domain Controller
- **LAB03** join di `CLIENT1` al dominio `lab.local`
- **LAB04** OU, utenti e gruppi
- **LAB05** deleghe amministrative
- **LAB06** GPO base
- **LAB07** GPO avanzate

## 3.4 Account suggeriti

Per le operazioni amministrative usa:

- `LAB\Administrator`

Per i test di accesso o di visibilità client puoi usare anche un utente standard, se utile.

## 3.5 Configurazione di rete attesa

Esempio coerente con il setup del corso:

- `DC1` = `192.168.50.10`
- `SRV1` = `192.168.50.20`
- `CLIENT1` = `192.168.50.30`
- DNS preferito dei member = `192.168.50.10`

Adatta gli indirizzi se il tuo ambiente usa valori diversi, ma mantieni la logica:

- **tutti i member del dominio devono usare come DNS il Domain Controller che ospita il DNS del dominio**

---

# Parte 1 - Concetti operativi

# 4. Perché Active Directory dipende dal DNS

Molti sistemi usano il DNS per comodità. Active Directory lo usa per esistere operativamente.

Quando un client di dominio deve autenticarsi, non gli basta conoscere un nome “simbolico”. Deve poter localizzare servizi specifici.

Per esempio:

- LDAP per interrogare la directory
- Kerberos per l'autenticazione
- Domain Controller disponibili per il dominio
- catalogo globale, quando presente e necessario

Questo significa che, in un dominio Windows, un DNS sbagliato non produce solo “nomi non risolti”. Produce problemi a catena su:

- join dominio
- logon
- Group Policy
- strumenti amministrativi
- localizzazione del DC
- secure channel della macchina

## 4.1 Formula da ricordare

Nel contesto di ADDS:

- **AD dice chi sei**
- **DNS dice dove trovare i servizi che lo dimostrano**

---

# 5. Zona DNS standard e zona integrata in Active Directory

![Dipendenza operativa di Active Directory dal DNS del dominio](images/lab08_dipendenza_dns_ad.png)


## 5.1 Zona DNS standard

Una zona standard salva i dati DNS come database locale del servizio DNS.

È utile in molti scenari, ma in ambiente Active Directory non è la forma più interessante da studiare qui.

## 5.2 Zona integrata in Active Directory

Una zona **AD-integrated** salva i dati DNS dentro Active Directory.

Questo comporta vantaggi importanti:

- replica coerente tramite meccanismi della directory
- migliore integrazione con il dominio
- possibilità di usare **secure dynamic updates**
- minore frammentazione della gestione

## 5.3 Perché qui ci interessa

Perché nel nostro dominio `lab.local` il DNS non è un servizio generico separato: è parte dell'infrastruttura del dominio.

Il partecipante deve capire che il DNS del dominio non va trattato come:

- DNS pubblico generico
- DNS “a caso” della scheda di rete
- servizio neutro intercambiabile con 8.8.8.8

Questo equivoco rompe mezza aula ogni volta che qualcuno cambia il DNS del client “per provare se Internet va meglio”.

---

# 6. Record A, PTR e SRV: differenze operative

![Confronto operativo tra record A e record SRV in Active Directory](images/lab08_record_a_vs_srv.png)


## 6.1 Record A

Associano un nome host a un indirizzo IPv4.

Esempi:

- `dc1.lab.local -> 192.168.50.10`
- `client1.lab.local -> 192.168.50.30`

Servono a risolvere i nomi delle macchine.

## 6.2 Record PTR

Associano un indirizzo IP a un nome.

Sono usati nella risoluzione inversa. Non sono sempre il primo elemento che un principiante guarda, ma sono utili per:

- coerenza diagnostica
- troubleshooting
- alcuni strumenti amministrativi

## 6.3 Record SRV

Sono i più importanti nel contesto Active Directory.

Non dicono solo “questo host ha questo IP”, ma:

- **questo servizio si trova qui**

Nel dominio ci interessano soprattutto i record che aiutano a trovare:

- LDAP
- Kerberos
- Domain Controller

## 6.4 Punto didattico chiave

Se il record A è “dov'è la macchina”, il record SRV è “dov'è il servizio”.

Per Active Directory, i record SRV sono il ponte tra:

- identità logica del dominio
- localizzazione reale dei servizi che lo rendono utilizzabile

---

# 7. La struttura _msdcs e i record SRV principali

Aprendo la zona del dominio o interrogandola via strumenti CLI, troverai cartelle e record che a prima vista sembrano poco intuitivi.

Tra i più importanti:

- `_msdcs.lab.local`
- `_tcp`
- `_udp`
- `_sites`

## 7.1 Record da saper riconoscere

Nel laboratorio dovrai almeno leggere e interpretare:

- `_ldap._tcp.dc._msdcs.lab.local`
- `_kerberos._tcp.lab.local`
- `_ldap._tcp.lab.local`

## 7.2 Cosa indicano

Esempi interpretativi:

- `_ldap._tcp.dc._msdcs.lab.local`
  - indica dove trovare i Domain Controller che offrono LDAP nel dominio
- `_kerberos._tcp.lab.local`
  - indica dove trovare il servizio Kerberos del dominio
- `_ldap._tcp.lab.local`
  - indica servizi LDAP del dominio in un contesto più generale

## 7.3 Non imparare a memoria senza capire

L'obiettivo non è memorizzare tutte le sottocartelle. L'obiettivo è saper rispondere a domande del tipo:

- quale record un client userà per trovare un DC?
- quale record mi interessa se sospetto un problema di logon?
- dove guardo se voglio verificare che il DC stia pubblicando correttamente i suoi servizi?

---

# 8. Registrazione dinamica e secure dynamic updates

## 8.1 Che cos'è la registrazione dinamica

La **dynamic registration** consente a client e server di registrare o aggiornare automaticamente i propri record DNS.

Questo evita di gestire tutto a mano e mantiene la zona più coerente.

## 8.2 Perché è utile

Permette, per esempio:

- aggiornamento automatico del record A del client
- coerenza dopo cambi IP o rejoin
- minore manutenzione manuale

## 8.3 Perché può creare problemi

Quando non funziona come atteso, puoi trovarti con:

- record mancanti
- record stantii
- nomi non coerenti
- ambiguità nella diagnosi

## 8.4 Secure dynamic updates

In una zona integrata in AD, è buona pratica usare aggiornamenti dinamici sicuri.

Per il laboratorio, l'obiettivo non è costruire una policy enterprise completa di controllo aging/scavenging, ma capire:

- che il record non sempre nasce o si aggiorna “per magia”
- che `ipconfig /registerdns` ha senso solo se sai cosa stai provando a forzare

---

# 9. Errori DNS tipici in ambiente AD

Questi sono gli errori che contano davvero in aula e in azienda.

## 9.1 DNS del client impostato su DNS pubblico

Sintomi:

- impossibilità di trovare il dominio
- join fallito
- logon di dominio problematico
- `nltest` o `gpupdate` che producono errori

## 9.2 Record del client mancante o incoerente

Sintomi:

- risoluzione del nome che fallisce
- strumenti amministrativi che si comportano in modo strano
- diagnosi poco chiare

## 9.3 Il nome host risolve ma i servizi di dominio no

Sintomo classico:

- `ping dc1.lab.local` funziona
- ma la localizzazione del DC o del servizio Kerberos fallisce

Questo è il caso perfetto per capire perché A e SRV non sono la stessa cosa.

## 9.4 DNS “funziona” ma non è quello corretto per il dominio

A volte Internet funziona e l'utente pensa che sia tutto a posto. Ma il dominio no.

È il tipo di errore più subdolo e più didatticamente utile da far vedere.

---

# Parte 2 - Step-by-step guidato

# 10. Prerequisiti operativi immediati

Prima di iniziare verifica:

- `DC1` acceso e raggiungibile
- `CLIENT1` e `SRV1` accesi
- `CLIENT1` e `SRV1` configurati con DNS verso `DC1`
- logon amministrativo disponibile su `DC1`
- dominio `lab.local` funzionante

Su `CLIENT1` esegui:

```cmd
hostname
ipconfig /all
whoami
echo %logonserver%
```

Su `SRV1` esegui:

```cmd
hostname
ipconfig /all
whoami
echo %logonserver%
```

Annota nel report:

- nome macchina
- indirizzo IPv4
- DNS server configurato
- server di logon visualizzato

---

# 11. Esplorazione della console DNS su DC1

Su `DC1` apri:

```text
Server Manager -> Tools -> DNS
```

Espandi:

- `DC1`
- `Forward Lookup Zones`
- `lab.local`

Verifica la presenza di:

- host record principali
- cartelle `_msdcs`, `_tcp`, `_udp`, `_sites`
- eventuali record di `DC1`, `SRV1`, `CLIENT1`

## 11.1 Cosa osservare davvero

Non limitarti a “vedere che c'è qualcosa”. Devi documentare:

- se la zona è integrata in AD
- se gli aggiornamenti dinamici sono permessi
- quali record SRV principali risultano presenti
- quali host record sono già registrati

## 11.2 Verifica PowerShell facoltativa ma consigliata

Su `DC1` puoi usare:

```powershell
Get-DnsServerZone -Name "lab.local"
```

E, se vuoi più dettaglio:

```powershell
Get-DnsServerResourceRecord -ZoneName "lab.local" | Select-Object HostName,RecordType,Timestamp
```

---

# 12. Interrogazione dei record SRV con nslookup

Su `CLIENT1` apri il prompt dei comandi e lancia:

```cmd
nslookup
set type=SRV
_ldap._tcp.dc._msdcs.lab.local
_kerberos._tcp.lab.local
_ldap._tcp.lab.local
exit
```

## 12.1 Che cosa devi annotare

Per ogni interrogazione documenta:

- quale record stai interrogando
- quale server risponde come DNS
- quale host viene restituito come target del servizio
- che relazione c'è tra il record e il ruolo del Domain Controller

## 12.2 Domande guida

Scrivi nel report una risposta breve ma precisa a queste domande:

- quale record useresti per verificare la localizzazione di un Domain Controller?
- quale record è più direttamente collegato al servizio Kerberos?
- perché un record SRV è più informativo di un semplice record A nel contesto del dominio?

---

# 13. Interrogazione con Resolve-DnsName da PowerShell

Su `DC1` o `SRV1` esegui:

```powershell
Resolve-DnsName dc1.lab.local
Resolve-DnsName client1.lab.local
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.lab.local
Resolve-DnsName -Type SRV _kerberos._tcp.lab.local
```

## 13.1 Perché farlo anche in PowerShell

Perché i partecipanti devono vedere che il DNS non si verifica solo da GUI o da `ping`, ma anche con strumenti più leggibili e scriptabili.

Nel report confronta brevemente:

- `nslookup`
- `Resolve-DnsName`

Scrivi quale dei due trovi più chiaro e perché.

---

# 14. Verifica della registrazione dinamica del client

Su `CLIENT1` esegui:

```cmd
hostname
ipconfig /all
ipconfig /registerdns
ipconfig /flushdns
ipconfig /displaydns
```

Attendi alcuni secondi.

Poi su `DC1` verifica dalla console DNS o via PowerShell che il record di `CLIENT1` sia presente.

Esempio PowerShell:

```powershell
Get-DnsServerResourceRecord -ZoneName "lab.local" -Name "CLIENT1"
```

Se il nome nel tuo ambiente è diverso, adattalo.

## 14.1 Cosa osservare

Annota:

- se il record A del client era già presente
- se la registrazione forzata ha cambiato qualcosa
- se il client usa effettivamente il DNS corretto
- se emergono discrepanze tra nome macchina e record DNS

---

# 15. Verifica della localizzazione del Domain Controller

Su `CLIENT1` esegui:

```cmd
nltest /dsgetdc:lab.local
```

Osserva:

- nome del Domain Controller trovato
- eventuali flag rilevanti
- esito della chiamata

Poi esegui anche:

```cmd
echo %logonserver%
```

## 15.1 Scopo didattico

Qui devi collegare tre cose:

- DNS corretto
- localizzazione del Domain Controller
- logon di dominio

Nel report scrivi con parole tue:

- perché `nltest /dsgetdc:lab.local` è un test più utile di un semplice ping

---

# 16. Diagnostica DNS del Domain Controller

![Flusso di diagnostica DNS in ambiente Active Directory](images/lab08_flusso_diagnostica_dns.png)


Su `DC1` esegui:

```cmd
dcdiag /test:dns /v
```

Il comando può produrre output lungo. Non devi incollare tutto il mondo nel report.

## 16.1 Cosa devi cercare

Identifica almeno:

- eventuali errori o warning
- conferme sul corretto funzionamento del DNS del DC
- riferimenti ai record registrati o ai test eseguiti

Nel report riporta:

- sintesi del risultato
- una o due righe significative
- tua interpretazione tecnica

---

# 17. Test di risoluzione diretta e confronto tra macchine

Su `CLIENT1` esegui:

```cmd
nslookup dc1.lab.local
nslookup srv1.lab.local
ping dc1.lab.local
ping srv1.lab.local
```

Su `SRV1` esegui:

```cmd
nslookup dc1.lab.local
nslookup client1.lab.local
```

Confronta:

- risultato su client
- risultato su member server
- coerenza della risoluzione nel dominio

---

# 18. Simulazione errore 1 - DNS del client configurato in modo errato

Questa è la simulazione più importante del laboratorio.

## 18.1 Salva la situazione iniziale

Su `CLIENT1` annota la configurazione corretta corrente:

```cmd
ipconfig /all
```

## 18.2 Introduci l'errore

Modifica temporaneamente il DNS del client impostando un server DNS non autorevole per `lab.local`, ad esempio:

- un DNS pubblico
- oppure un indirizzo non valido nella tua rete di lab

## 18.3 Riesegui i test

Esegui:

```cmd
nslookup lab.local
nslookup dc1.lab.local
nltest /dsgetdc:lab.local
gpupdate /force
echo %logonserver%
```

## 18.4 Cosa devi osservare

Documenta con precisione:

- quali comandi falliscono
- quali restituiscono esiti ambigui
- quali sintomi avresti notato se fossi stato un utente e non un amministratore

## 18.5 Ripristino obbligatorio

Ripristina il DNS corretto verso `DC1`.

Poi riesegui almeno:

```cmd
ipconfig /flushdns
ipconfig /registerdns
nltest /dsgetdc:lab.local
```

Nel report confronta chiaramente:

- comportamento con DNS corretto
- comportamento con DNS errato

---

# 19. Simulazione errore 2 - Record incoerente o verifica manuale controllata

Questa seconda simulazione è meno distruttiva ma molto istruttiva.

Scegli **una** delle due varianti.

## Variante A - Crea temporaneamente un record host fittizio

Su `DC1`, nella zona `lab.local`, crea manualmente un record A ad esempio:

- `app-demo.lab.local -> 192.168.50.250`

Poi verifica:

```cmd
nslookup app-demo.lab.local
ping app-demo.lab.local
```

Scopo:

- mostrare che il DNS può risolvere un nome anche quando dietro non esiste un servizio reale utile

## Variante B - Verifica l'effetto di un nome non registrato

Scegli un nome di macchina che non esiste nella zona, ad esempio:

```cmd
nslookup client9.lab.local
```

Scopo:

- distinguere tra errore di risoluzione diretto e problema di localizzazione di un servizio AD

## 19.1 Commento richiesto

Nel report spiega la differenza tra:

- nome host non risolto
- servizio di dominio non localizzato
- record presente ma non utile al servizio atteso

---

# 20. Analisi finale ragionata

Scrivi una sezione finale in cui rispondi in modo ragionato a queste domande:

1. Perché in Active Directory il DNS del client non deve puntare a un DNS pubblico?
2. Qual è la differenza pratica tra record A e record SRV?
3. In quale test del laboratorio hai visto più chiaramente la dipendenza di AD dal DNS?
4. Perché `ipconfig /registerdns` non è una formula magica ma un'azione mirata?
5. Se un partecipante dicesse “il ping funziona quindi il dominio è a posto”, come gli risponderesti?

---

# Parte 3 - Verifiche ed evidenze

# 21. Verifiche minime obbligatorie

A fine laboratorio devi poter dimostrare almeno queste evidenze:

- interrogazione riuscita di almeno due record SRV
- verifica del record A del client o del server membro
- output di `nltest /dsgetdc:lab.local`
- sintesi di `dcdiag /test:dns`
- simulazione controllata di DNS errato e successivo ripristino

## 21.1 Comandi chiave da riportare almeno in parte

```cmd
nslookup
nltest /dsgetdc:lab.local
dcdiag /test:dns /v
ipconfig /registerdns
```

```powershell
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.lab.local
Get-DnsServerResourceRecord -ZoneName "lab.local"
```

---

# 22. File di evidenza richiesto

Crea:

```text
docs/evidence_lab08.md
```

Usa questa struttura minima:

```md
# Evidence LAB08

## 1. VM usate

## 2. Configurazione DNS iniziale

## 3. Record SRV analizzati

## 4. Verifica registrazione dinamica

## 5. Test nltest e localizzazione del DC

## 6. Sintesi dcdiag /test:dns

## 7. Simulazione DNS errato

## 8. Ripristino e verifica finale

## 9. Problemi incontrati

## 10. Cosa ho capito sul rapporto tra DNS e AD
```

---

# Parte 4 - Consegna

# 23. Consegna richiesta

Al termine devi consegnare:

- ambiente ripristinato correttamente
- file `docs/evidence_lab08.md`
- eventuali screenshot solo se realmente utili a chiarire un risultato

---

# 24. Conclusione del laboratorio

Con questo laboratorio hai affrontato uno dei nodi più importanti dell'intero corso.

Da qui in avanti nessun partecipante dovrebbe più dire:

- “Active Directory non va”

senza prima chiedersi:

- “il DNS è davvero configurato come servizio del dominio?”
- “i record SRV ci sono?”
- “il client sta usando il DNS corretto?”
- “riesco a localizzare il DC e i servizi di autenticazione?”

Se questa catena logica è chiara, il laboratorio ha raggiunto il suo scopo.
