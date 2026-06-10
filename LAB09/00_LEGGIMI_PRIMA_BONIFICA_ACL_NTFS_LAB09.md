# LEGGIMI_PRIMA - Bonifica ACL NTFS, ereditarietà e gruppi generici

Questo file evidenzia il punto operativo che deve essere eseguito nel LAB09 prima di considerare affidabile il test con Access Based Enumeration.

## Problema osservato

Durante il test della share `\\SRV1\DatiLab09`, un utente del reparto Sales, per esempio `LAB\lab09.sales1`, può continuare a vedere o aprire la cartella `HR` anche se ABE è attiva.

Questo comportamento non indica un errore di ABE. Indica quasi sempre che sulla cartella `HR` esiste ancora un permesso NTFS ereditato o generico, ad esempio:

```text
SRV1\Users
BUILTIN\Users
Authenticated Users
Domain Users
Everyone
```

Se uno di questi gruppi ha `Read`, `Read & execute`, `List folder contents`, `Write` o `Modify`, Windows considera l'utente autorizzato almeno alla lettura o all'elenco della cartella. Di conseguenza ABE mostra la cartella.

## Correzione da eseguire sulle cartelle riservate

La correzione deve essere svolta almeno su:

```text
D:\DatiLab09\Sales
D:\DatiLab09\HR
```

Se nel laboratorio è stato usato il percorso alternativo, lavoriamo su:

```text
C:\Shares\DatiLab09\Sales
C:\Shares\DatiLab09\HR
```

Per ogni cartella riservata:

1. apriamo **Properties**;
2. apriamo la scheda **Security**;
3. selezioniamo **Advanced**;
4. selezioniamo **Disable inheritance**;
5. scegliamo **Convert inherited permissions into explicit permissions on this object**;
6. rimuoviamo i gruppi generici non previsti, se presenti;
7. manteniamo solo gli account amministrativi necessari;
8. aggiungiamo il gruppo Domain Local corretto;
9. assegniamo `Modify` al gruppo Domain Local;
10. verifichiamo che `Applies to` sia `This folder, subfolders and files`.

## Gruppi da rimuovere dalle cartelle `Sales` e `HR`

Rimuoviamo dalle cartelle riservate eventuali voci come:

```text
SRV1\Users
BUILTIN\Users
Authenticated Users
Domain Users
Everyone
```

La rimozione è necessaria quando questi gruppi hanno permessi di lettura, elenco, scrittura o modifica.

## Gruppi da mantenere

Sulle cartelle riservate manteniamo:

```text
SYSTEM
Administrators
LAB\Domain Admins
```

## ACL finale attesa per `Sales`

```text
SYSTEM                         Full Control
Administrators                 Full Control
LAB\Domain Admins              Full Control
LAB\DL_FS_LAB09_Sales_RW       Modify
```

## ACL finale attesa per `HR`

```text
SYSTEM                         Full Control
Administrators                 Full Control
LAB\Domain Admins              Full Control
LAB\DL_FS_LAB09_HR_RW          Modify
```

## ACL finale attesa per `Scambio`

```text
SYSTEM                         Full Control
Administrators                 Full Control
LAB\Domain Admins              Full Control
LAB\DL_FS_LAB09_Scambio_RW     Modify
```

## Permesso sulla cartella principale `DatiLab09`

Sulla cartella principale possiamo concedere ai gruppi Domain Local solo la possibilità di raggiungere la radice della share, senza propagare automaticamente il permesso alle sottocartelle:

```text
LAB\DL_FS_LAB09_Sales_RW       Read & execute / List folder contents       This folder only
LAB\DL_FS_LAB09_HR_RW          Read & execute / List folder contents       This folder only
LAB\DL_FS_LAB09_Scambio_RW     Read & execute / List folder contents       This folder only
```

Il valore `This folder only` è essenziale: serve a evitare che il permesso della radice venga ereditato da `Sales`, `HR` e `Scambio`.

## Test atteso da CLIENT1

Da `CLIENT1`, accediamo con `LAB\lab09.sales1` e apriamo:

```text
\\SRV1\DatiLab09
```

Risultato atteso:

```text
Sales      visibile e accessibile
Scambio    visibile e accessibile
HR         non visibile con ABE attiva
```

Se `HR` è ancora visibile, ricontrolliamo l'ACL NTFS di `HR`: quasi certamente è rimasto un gruppo generico con permessi utili.
