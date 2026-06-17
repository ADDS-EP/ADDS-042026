# DOCENTE_Q&A_LAB10 - DHCP in ambiente Active Directory

Versione per il docente - risposte e spiegazioni alle domande di consolidamento

Questo file raccoglie risposte e spunti di spiegazione per le domande di consolidamento del LAB10. È pensato come supporto al docente e non come materiale da distribuire necessariamente ai partecipanti prima della discussione finale.

---

## 1. Perché in un dominio Active Directory il server DHCP deve essere autorizzato?

In un dominio Active Directory, l'autorizzazione serve a registrare esplicitamente quali server DHCP sono approvati per distribuire configurazioni IP nella rete di dominio.

Il punto didattico da sottolineare è che DHCP non distribuisce solo indirizzi IP, ma anche parametri critici come DNS, gateway e suffisso di dominio. Un DHCP non autorizzato o configurato male può impedire ai client di trovare il dominio anche se la rete sembra funzionare.

Esempio da richiamare in aula:

```text
CLIENT1 riceve IP 192.168.50.115
ma riceve DNS 8.8.8.8
```

Il client può avere rete IP, ma non risolvere correttamente `lab.local`.

---

## 2. Quale differenza c'è tra scope, lease, exclusion e reservation?

- **Scope**: intervallo di indirizzi che il DHCP server può gestire per una subnet.
- **Lease**: assegnazione temporanea di un indirizzo IP a un client.
- **Exclusion**: intervallo o indirizzo dentro il range dello scope che il DHCP non deve assegnare.
- **Reservation**: associazione tra uno specifico client e uno specifico indirizzo IP.

Spiegazione utile:

```text
Lo scope definisce il contenitore.
Il lease è l'assegnazione corrente.
L'exclusion sottrae indirizzi dal pool dinamico.
La reservation mantiene un client stabile, ma sempre sotto gestione DHCP.
```

---

## 3. Perché l'opzione DHCP 006 DNS Servers è critica per un client di dominio?

Perché i client Active Directory devono usare un DNS capace di risolvere i record interni del dominio, compresi i record legati ai Domain Controller.

Se l'opzione 006 distribuisce un DNS pubblico o non AD-integrato, il client può non riuscire a:

- trovare un Domain Controller;
- eseguire correttamente il logon di dominio;
- applicare Group Policy;
- risolvere nomi interni come `SRV1.lab.local`.

Risposta sintetica da accettare:

```text
L'opzione 006 deve puntare al DNS AD-integrato, nel laboratorio DC1, perché Active Directory dipende dalla risoluzione DNS interna.
```

---

## 4. In quali casi configuriamo l'opzione 003 Router?

L'opzione 003 Router indica il gateway predefinito da distribuire ai client.

Va configurata quando i client devono comunicare con reti diverse dalla propria subnet, per esempio Internet o un'altra rete aziendale.

Nel laboratorio va configurata solo se esiste davvero un gateway nella rete virtuale usata. Se non esiste, non va inventata.

Spiegazione didattica:

```text
DNS serve a risolvere nomi.
Gateway serve a uscire dalla subnet.
Sono funzioni diverse.
```

---

## 5. Perché un client può avere un IP valido ma non riuscire a trovare il Domain Controller?

Perché l'indirizzo IP è solo una parte della configurazione. Il client può ricevere un IP coerente con la subnet, ma avere:

- DNS errato;
- suffisso DNS mancante;
- gateway non necessario ma configurato male;
- cache DNS incoerente;
- problemi di raggiungibilità verso `DC1`.

Nel laboratorio il caso più importante è il DNS errato tramite opzione 006.

Comandi utili per distinguere il problema:

```cmd
ipconfig /all
nslookup lab.local
nltest /dsgetdc:lab.local
```

---

## 6. Che cosa indica un indirizzo 169.254.x.x su CLIENT1?

Un indirizzo `169.254.x.x` indica APIPA, cioè una configurazione automatica assegnata dal client quando non riesce a ottenere un lease DHCP valido.

Possibili cause:

- DHCP server non raggiungibile;
- scope disattivato;
- servizio DHCP fermo;
- rete virtuale errata;
- firewall o configurazione di rete che impedisce il traffico;
- indirizzi disponibili esauriti.

Risposta sintetica:

```text
CLIENT1 non ha ricevuto una configurazione valida dal DHCP server.
```

---

## 7. Come distinguiamo un problema DHCP da un problema DNS?

Partiamo da `ipconfig /all`.

Se il client non ha IP valido oppure ha APIPA, il problema è probabilmente DHCP o rete.

Se il client ha IP, DHCP Server e DNS ricevuti, ma non risolve `lab.local`, il problema è probabilmente DNS o opzioni DHCP errate.

Sequenza consigliata:

```cmd
ipconfig /all
ping SRV1
nslookup lab.local
nltest /dsgetdc:lab.local
```

Schema di ragionamento:

```text
Nessun IP valido → DHCP/rete
IP valido ma DNS errato → opzione 006
DNS corretto ma DC non trovato → DNS AD, record, servizio DC, rete verso DC1
```

---

## 8. Quale informazione osserviamo in Address Leases?

In **Address Leases** osserviamo gli indirizzi assegnati dal server DHCP ai client.

Informazioni tipiche:

- IP assegnato;
- nome del client;
- MAC address o client identifier;
- scadenza lease;
- eventuale stato della reservation.

Punto da chiarire:

```text
Address Leases mostra cosa il DHCP ha assegnato davvero, non solo cosa era previsto dallo scope.
```

---

## 9. Perché una reservation non è la stessa cosa di un IP statico configurato manualmente?

Con un IP statico, la configurazione è salvata manualmente sul client. Il server DHCP non gestisce quell'assegnazione.

Con una reservation, il client resta configurato in DHCP, ma riceve sempre lo stesso IP perché il server riconosce il suo MAC address o client identifier.

Vantaggio della reservation:

- gestione centralizzata;
- possibilità di distribuire anche DNS, gateway e suffisso dominio;
- minore rischio di incoerenza rispetto a configurazioni manuali sparse sui client.

---

## 10. Quali controlli eseguiamo prima di chiudere il laboratorio per evitare regressioni?

Controlli minimi lato client:

```cmd
ipconfig /all
nslookup lab.local
nltest /dsgetdc:lab.local
ping SRV1
```

Controlli minimi lato server:

```powershell
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 192.168.50.0
```

Controlli organizzativi:

- verificare se `CLIENT1` deve restare in DHCP o tornare statico;
- documentare scope, opzioni, lease e reservation;
- conservare backup/export DHCP se generato;
- verificare che `DC1` non sia stato modificato impropriamente.

---

## Spunti per debrief finale

Durante il debrief conviene far emergere tre concetti:

1. DHCP e DNS sono servizi distinti ma collegati nel funzionamento reale di Active Directory.
2. Un indirizzo IP valido non dimostra da solo che il client sia configurato bene.
3. La diagnosi deve partire dal sintomo osservato, non dal servizio che si presume sia guasto.

Domanda utile da proporre oralmente:

```text
Se CLIENT1 riceve un IP dallo scope ma non applica le GPO, quali controlli facciamo prima di modificare le GPO?
```

Risposta attesa:

```text
Controlliamo ipconfig /all, DNS ricevuto, nslookup lab.local, nltest /dsgetdc:lab.local e solo dopo passiamo alla diagnosi GPO.
```
