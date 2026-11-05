# Lecture 01 — Email Security

**Docente:** Leonardo Querzoni  
**Fonte:** `1 - Email security.pdf` (2026/27), slide **1–33**  
**Stato:** slide 1–33 affrontate; **ripasso del sistema binario e di Base64 ancora da svolgere e verificare**. Gli esempi integrativi sono distinti dalle slide.

## Architettura e flusso (slide 1–6)

L'email usa un modello **store-and-forward**: i server memorizzano e inoltrano i messaggi; mittente e destinatario non devono essere online insieme.

![Email architecture: MUA, MSA, MTA, MDA, Message Store](../assets/email-architecture.svg)

| Componente | Funzione essenziale |
| --- | --- |
| **MUA** (*Mail User Agent*) | Client per comporre e leggere email |
| **MSA** (*Mail Submission Agent*) | Accetta il messaggio in uscita dal client |
| **MTA** (*Mail Transfer Agent*) | Trasferisce email fra server: può essere client e server SMTP |
| **MDA** (*Mail Delivery Agent*) | **Deposita** il messaggio nella mailbox del destinatario |
| **Message Store** | Conserva i messaggi consegnati |
| **MRA** (*Mail Retrieval Agent*) | **Recupera** i messaggi dalla mailbox per renderli disponibili al MUA |

**Flusso:** `MUA → MSA → MTA → [altri MTA] → MDA → mailbox → MRA → MUA`. Sono **ruoli logici**, non necessariamente sistemi separati; l'MRA può essere integrato nel MUA. **SMTP** invia/trasferisce, **POP3/IMAP** permettono l'accesso alla mailbox.

*Precisazione:* la slide 6 descrive MDA; MRA compare nel titolo. La spiegazione specifica dell'MRA è un'integrazione.

## SMTP: envelope ≠ message (slide 7–8)

SMTP (*Simple Mail Transfer Protocol*, RFC 5321) trasferisce email; il messaggio ha un **envelope SMTP** separato da **header e body** (formato RFC 5322).

| Envelope SMTP | Header del messaggio |
| --- | --- |
| `MAIL FROM`: indirizzo di ritorno per errori di consegna | `From:`: mittente **dichiarato**, normalmente visibile nel client |
| `RCPT TO`: destinatari effettivi della consegna | `To:`: destinatari **dichiarati**, visibili nel client |

**Esempio:** `MAIL FROM:<bounce@mailer.example>`, ma `From: Alice <alice@example.com>`. Possono coincidere oppure no. Anche se coincidono, **non costituiscono da soli una prova dell'identità** del mittente. I destinatari effettivi `RCPT TO` possono differire dal campo `To:` (ad esempio Bcc).

**Email spoofing:** un attaccante può inserire un `From:` ingannevole. È il problema di autenticazione che motiva SPF, DKIM e DMARC (da approfondire più avanti).

## Struttura dell'header (slide 9)

L'header è formato da campi `Nome: valore` (RFC 5322) ed è separato dal **body da una riga vuota**.

- **Folding:** un valore può proseguire sulla riga successiva se questa inizia con spazio o tab.
- Gli header tradizionali usano ASCII a 7 bit; **MIME encoded words** possono rappresentare valori non ASCII.
- **MIME** estende inoltre la rappresentazione del contenuto (allegati, media e codifiche).

## Due identificatori diversi (slide 10–11)

| | **Message-ID** | **ENVID** |
| --- | --- | --- |
| A cosa si riferisce | Al messaggio | A una singola operazione di invio/consegna |
| Dove si trova | Campo dell'**header** | Identificatore dell'**envelope** SMTP |
| Assegnato da | MUA o MSA | Primo MTA (come indicato nella slide) |
| Utilità | Thread, duplicati, tracking | Tracciare un ciclo di consegna e le notifiche relative |

**Message-ID**, es. `<20241001102233.12345@example.com>`: il dominio dopo `@` identifica chi assegna l'ID; la parte prima di `@` identifica il messaggio in quell'ambito. È progettato per essere globalmente unico, **non è una prova di autenticità**.

**ENVID** (*Envelope Identifier*): identifica una **specifica operazione di invio e consegna**, non il messaggio in sé. Nel normale trasferimento `MTA 1 → MTA 2 → MTA 3` non viene rigenerato a ogni passaggio; neppure un semplice nuovo tentativo dopo un errore temporaneo costituisce automaticamente un nuovo *posting*. Se invece un sistema **re-immette il messaggio come nuova spedizione** (*re-posting*, per esempio una mailing list che lo redistribuisce), **non riutilizza l'ENVID precedente**, anche se il contenuto resta uguale.

*Precisazione tecnica esterna:* ENVID è un parametro **opzionale** dell'estensione SMTP DSN (RFC 3461), quindi non tutte le email lo usano. Il suo scopo è associare le notifiche di consegna alla corretta operazione di invio; il re-posting è distinto dal semplice inoltro tra MTA.

## Campi e destinatari effettivi (slide 12)

La slide cita `From:`, `To:`, `Date:`, `Message-ID:`, `Subject:` e li presenta come obbligatori. **Attenzione rispetto allo standard:** nella RFC 5322 non tutti lo sono (per esempio `Subject:` è facoltativo).

**Idea chiave della slide:** `To:` non determina necessariamente a chi SMTP consegna il messaggio; è `RCPT TO` a specificarlo. Analogamente, `From:` non prova chi abbia realmente inviato l'email.

## Da saper spiegare senza appunti

Perché **MDA** e **MRA** svolgono compiti diversi? Quale differenza esiste fra `MAIL FROM` e `From:`? Perché un `Message-ID` con un dominio apparentemente affidabile non autentica il messaggio?

## Slide 13–22 — SMTP e routing

I campi Cc, Bcc, Reply-To, Sender, In-Reply-To, References, Return-Path e Received completano l'header; alcuni aiutano a ricostruire destinatari, conversazioni e percorso, ma non garantiscono da soli autenticità.

Per consegnare a un altro dominio, l'MTA consulta i record DNS MX e poi contatta il mail exchanger via SMTP. La preferenza MX numericamente minore ha priorità.

Nella conversazione SMTP (slide 19), MAIL FROM e RCPT TO definiscono **l'envelope** e vengono trasmessi **prima di DATA**. Solo dopo DATA vengono inviati gli header del messaggio (From, To, Subject), una riga vuota e il body. Il punto su una riga separata termina i dati. Un codice 250 dopo DATA indica accettazione dal server, non lettura o consegna finale.

ESMTP usa EHLO per scoprire estensioni quali STARTTLS (protezione della connessione), AUTH (autenticazione del client), DSN (notifiche di consegna), SIZE, 8BITMIME, PIPELINING, CHUNKING e SMTPUTF8. TLS sul collegamento non implica cifratura end-to-end.

## MIME: formato, contenuto e codifica (slide 23–33)

**MIME (Multipurpose Internet Mail Extensions)** estende il formato email per gestire caratteri non ASCII, allegati e messaggi composti da più parti. **Non sostituisce SMTP**: SMTP trasporta, MIME descrive come interpretare i contenuti.

| Header MIME | Scopo |
| --- | --- |
| `MIME-Version: 1.0` | Versione MIME |
| `Content-Type: text/plain; charset=UTF-8` | Tipo del contenuto e, per il testo, interpretazione dei byte |
| `Content-Transfer-Encoding: base64` | Come sono codificati i byte per il trasporto |
| `Content-ID`, `Content-Description` | Identificazione e descrizione di una parte |

Esempi di tipi: `text/plain`, `image/jpeg`, `application/pdf`, `multipart/mixed`. **Charset ≠ transfer encoding**: UTF-8 trasforma caratteri in byte; Base64 trasforma byte in caratteri ASCII trasportabili.

**Non tutto il body deve essere Base64.** Un messaggio di solo testo può usare `7bit` (testo direttamente leggibile), mentre uno stesso testo può anche essere codificato in Base64. In un messaggio `multipart/mixed`, il testo può rimanere leggibile mentre **solo la parte PDF** usa `Content-Transfer-Encoding: base64`: ogni parte può avere il proprio Content-Type e la propria codifica.

**Base64, idea essenziale:** suddivide **3 byte = 24 bit** in **4 gruppi da 6 bit**; ciascuno seleziona uno dei 64 simboli (`A–Z`, `a–z`, `0–9`, `+`, `/`). Quando necessario usa `=` come padding. Esempio: `Man` → `TWFu`. È una **codifica reversibile, non cifratura**: non garantisce confidenzialità o autenticità.

> **Da ripassare prima di proseguire:** conversione decimale ↔ binario, byte e bit, potenze di 2, raggruppamento 8→6 bit, corrispondenza indice→simbolo Base64 e padding `=`. La meccanica del calcolo **non è ancora stata verificata con un esercizio autonomo**.
