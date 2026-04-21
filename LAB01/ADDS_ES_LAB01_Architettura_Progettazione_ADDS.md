# ADDS_ES_LAB01 - Esercitazione autonoma architettura Active Directory e progettazione ambiente

## Traccia individuale di consolidamento

---

# 1. Obiettivo dell’esercitazione

Questa esercitazione autonoma serve a verificare che tu abbia compreso la struttura di Active Directory e che tu sia in grado di tradurre i concetti in un piccolo progetto tecnico di laboratorio.

Non devi ancora installare AD DS.

Devi però dimostrare che sai:

- usare i termini corretti
- classificare i componenti logici e fisici
- definire il dominio del laboratorio
- associare le VM ai ruoli previsti
- impostare un piano IP coerente
- spiegare il ruolo del DNS

---

# 2. Scenario

Sei incaricato di preparare la documentazione iniziale di un laboratorio Windows che, nelle sessioni successive, verrà usato per:

- creare un dominio AD DS
- joinare client al dominio
- gestire utenti, gruppi e OU
- applicare Group Policy
- configurare servizi di rete
- arrivare a un piccolo scenario cluster

Hai a disposizione l’ambiente predisposto nel LAB00 con:

- Hyper-V
- Windows Server 2022
- Windows 11 client
- rete di laboratorio `192.168.50.0/24`

---

# 3. Risultato atteso

Al termine dell’esercitazione devi consegnare un file di evidenza che contenga:

- definizione del dominio
- classificazione logica/fisica di elementi AD
- tabella ruoli VM
- piano IP
- spiegazione tecnica del ruolo del DNS
- tabella FSMO sintetica
- schema testuale della topologia

---

# 4. Vincoli obbligatori

Usa questi nomi standard:

- dominio: `lab.local`
- NetBIOS: `LAB`
- `DC1`
- `SRV1`
- `CLIENT1`
- `CLU1`
- `CLU2`

Usa queste reti:

- `192.168.50.0/24`
- `10.10.10.0/24` per la seconda rete cluster

---

# 5. Attività da svolgere

## Attività 1 - Definisci con parole tue che cos’è AD DS

Scrivi un testo breve di 6-10 righe in cui spieghi:

- che cos’è AD DS
- a cosa serve in una rete Windows
- differenza tra autenticazione e autorizzazione

### Evidenza richiesta

Inserisci il testo nel report finale.

---

## Attività 2 - Classifica i concetti

Compila la tabella seguente:

| Elemento | Logico o fisico? | Motivo |
|---|---|---|
| domain |  |  |
| forest |  |  |
| tree |  |  |
| OU |  |  |
| site |  |  |
| Domain Controller |  |  |
| subnet |  |  |
| utenti |  |  |
| gruppi |  |  |
| replica |  |  |

### Evidenza richiesta

Completa la tabella nel report.

---

## Attività 3 - Assegna i ruoli alle VM

Compila la tabella:

| VM | Ruolo previsto | Motivo |
|---|---|---|
| DC1 |  |  |
| SRV1 |  |  |
| CLIENT1 |  |  |
| CLU1 |  |  |
| CLU2 |  |  |

### Evidenza richiesta

Spiega in almeno una riga perché `DC1` sarà la macchina più importante delle prime sessioni.

---

## Attività 4 - Definisci il piano IP del laboratorio

Compila la tabella:

| Nodo | Rete | IP previsto |
|---|---|---|
| DC1 | vLab-ADDS |  |
| SRV1 | vLab-ADDS |  |
| CLIENT1 | vLab-ADDS |  |
| CLU1 | vLab-ADDS |  |
| CLU1 | vLab-CLUSTER |  |
| CLU2 | vLab-ADDS |  |
| CLU2 | vLab-CLUSTER |  |

### Evidenza richiesta

Aggiungi due righe per spiegare perché i nodi cluster hanno una seconda rete dedicata.

---

## Attività 5 - Spiega il ruolo del DNS

Rispondi alle domande:

1. Perché il Domain Controller ospiterà anche DNS?
2. Perché `CLIENT1` non deve usare un DNS pubblico come DNS primario?
3. Che tipo di problemi potresti vedere se il DNS è errato?

### Evidenza richiesta

Scrivi almeno 8-10 righe complessive.

---

## Attività 6 - Tabella FSMO sintetica

Compila:

| Ruolo FSMO | Ambito | Descrizione sintetica | Nodo iniziale nel lab |
|---|---|---|---|
| Schema Master |  |  |  |
| Domain Naming Master |  |  |  |
| RID Master |  |  |  |
| PDC Emulator |  |  |  |
| Infrastructure Master |  |  |  |

### Evidenza richiesta

Indica quale ruolo ti sembra più “critico” in un dominio molto piccolo e perché.

---

## Attività 7 - Verifica minima della tua infrastruttura

Accendi solo:

- `DC1`
- `SRV1`
- `CLIENT1`

Verifica su almeno due macchine:

```powershell
hostname
ipconfig /all
Test-Connection 192.168.50.10 -Count 2
```

### Evidenza richiesta

Riporta:

- hostname
- IPv4
- DNS configurato
- esito del ping

---

## Attività 8 - Disegna la topologia in formato testuale

Crea uno schema come questo:

```text
Forest: lab.local
Domain: lab.local

Rete principale: 192.168.50.0/24
  DC1      192.168.50.10
  SRV1     192.168.50.20
  CLIENT1  192.168.50.100
  CLU1     192.168.50.31
  CLU2     192.168.50.32

Rete cluster: 10.10.10.0/24
  CLU1     10.10.10.31
  CLU2     10.10.10.32
```

### Evidenza richiesta

Inserisci lo schema nel report finale.

---

# 6. File da consegnare

Crea il file:

```text
ADDS_evidence_es_lab01.md
```

Usa questa struttura:

```md
# Evidence esercitazione autonoma LAB01

## 1. Che cos’è AD DS

## 2. Classificazione logica/fisica

## 3. Ruolo delle VM

## 4. Piano IP

## 5. DNS e AD

## 6. FSMO overview

## 7. Verifica rete macchine

## 8. Schema testuale della topologia

## 9. Commento finale
- che cosa mi è chiaro
- che cosa devo ancora consolidare prima del LAB02
```

---

# 7. Criteri di autovalutazione

Considera l’esercitazione riuscita se:

- distingui senza incertezza domain e Domain Controller
- non confondi OU e struttura fisica
- sai spiegare il ruolo del DNS in AD
- hai un piano IP coerente
- colleghi correttamente le VM ai moduli successivi del corso

Se invece hai ancora dubbi su questi punti, il problema non è grave, ma va risolto prima della promozione di `DC1` a Domain Controller.

---

# 8. Conclusione

Questa esercitazione non serve a “fare qualcosa di spettacolare”.

Serve a evitare il tipo di errore didattico più comune nei laboratori infrastrutturali: installare prima di aver progettato.

Nel LAB02 userai esattamente queste decisioni per creare il dominio vero del laboratorio.
