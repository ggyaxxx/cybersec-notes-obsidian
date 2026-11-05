# Lecture 01 — Email Security

**Docente:** Leonardo Querzoni  
**Fonte primaria:** `1 - Email security.pdf`, Sapienza 2026/27  
**Stato:** studiate e sintetizzate le slide **1–8**; il resto della lezione è da completare.  
**Nota sulle fonti:** i chiarimenti e gli esempi aggiuntivi sono distinti dal contenuto presentato nelle slide.

## Panorama e idea fondamentale (slide 1–3)

La lezione introduce l'architettura email e, in seguito, SMTP, POP/IMAP, MIME, minacce e meccanismi di sicurezza. Distinguere la sicurezza del **trasporto** da quella del **contenuto end-to-end**.

**Store-and-forward:** un server riceve, memorizza temporaneamente e inoltra o consegna un messaggio. Mittente e destinatario **non devono essere online contemporaneamente**.

## Architettura email (slide 4–6)

![Email architecture: MUA, MSA, MTA, MDA, Message Store](../assets/email-architecture.svg)

| Componente | Cosa fa |
| --- | --- |
| **MUA** — Mail User Agent | Client con cui l'utente scrive e legge email (es. Thunderbird). |
| **MSA** — Mail Submission Agent | Riceve dal MUA un'email in uscita per avviarne la spedizione. |
| **MTA** — Mail Transfer Agent | Riceve e inoltra messaggi tra server; può operare sia come **server SMTP** sia come **client SMTP**. |
| **MDA** — Mail Delivery Agent | **Consegna:** prende il messaggio arrivato tramite l'MTA e lo deposita nella mailbox del destinatario. |
| **Message Store / Mailbox** | Conserva le email già consegnate. |
| **MRA** — Mail Retrieval Agent | **Recupero:** permette di prelevare/accedere alle email nella mailbox per renderle disponibili al MUA, normalmente con funzionalità POP3/IMAP. |

**Percorso completo (ruoli logici):**

`MUA mittente → MSA → MTA mittente → MTA destinatario → MDA → Mailbox → MRA → MUA destinatario`

Il diagramma sopra illustra i componenti principali e il recupero POP/IMAP; **l'MRA può essere integrato nel MUA**, senza essere una macchina o un'applicazione distinta.

> **MDA vs MRA:** l'MDA **deposita** il messaggio nella mailbox; l'MRA lo **recupera** dalla mailbox. Non confondere queste operazioni con il trasferimento fra server, compito degli MTA.

**Fonte:** le slide 4–6 illustrano l'architettura e MDA; la distinzione operativa MRA/POP/IMAP è un **chiarimento integrativo**: nella slide 6 l'MRA compare nel titolo ma non viene sviluppato in dettaglio.

## SMTP e struttura di un'email (slide 7–8)

**SMTP (Simple Mail Transfer Protocol)** si occupa della **submission** e del **trasferimento** dei messaggi email. Non è il protocollo con cui normalmente il destinatario legge i messaggi già depositati: per l'accesso alla mailbox si usano **IMAP** o **POP3**.

Una singola email ha **due livelli distinti**:

1. **SMTP envelope:** dati usati dal sistema SMTP per inoltro e gestione delle consegne, in particolare `MAIL FROM` (indirizzo per gli eventuali messaggi di mancata consegna, *reverse-path*) e `RCPT TO` (destinatario di trasporto).
2. **Message (header + body):** contenuto trasmesso dopo il comando SMTP `DATA`. L'**header** contiene campi come `From:`, `To:`, `Subject:`, `Date:`; una **riga vuota** lo separa dal **body**.

### Esempio: envelope e header possono differire

```text
SMTP envelope:
MAIL FROM:<bounce@mailer.example>
RCPT TO:<bob@example.org>

Message header:
From: Alice <alice@example.com>
To: Bob <bob@example.org>
Subject: Riunione

Ciao Bob, ci vediamo domani.
```

Il `From:` e il `To:` normalmente mostrati da Gmail, Outlook o Thunderbird sono **campi dell'header**, **non** i comandi `MAIL FROM` e `RCPT TO` dell'envelope.

- Il mittente visibile (`From:`) e il mittente dell'envelope (`MAIL FROM`) **possono** coincidere, ma **non sono tenuti a coincidere**.
- Anche se coincidono, la sola uguaglianza **non autentica** l'identità del mittente.
- Analogamente, il `To:` visibile può differire dai destinatari effettivi `RCPT TO`, per esempio con i destinatari in Bcc.

### Implicazione di sicurezza: email spoofing

**Esempio integrativo (non dimostrazione di un attacco nella slide):** un messaggio può dichiarare nell'header `From: amministrazione@azienda.example` senza che tale dichiarazione provi l'origine reale. Ecco perché il semplice `From:` mostrato dal client **non è prova di autenticità**.

Questo problema motiva i successivi meccanismi di autenticazione della posta, come **SPF, DKIM e DMARC** (ancora da studiare): questi non vanno interpretati come una semplice verifica che i due campi `From` siano uguali.

### Message format e MIME

La **RFC 5322** descrive l'Internet Message Format: **header**, riga vuota, **body**. **MIME** estende la rappresentazione dei contenuti per supportare, fra l'altro, allegati, diversi tipi di media e codifiche.

## Domande rapide di verifica

1. Perché il modello *store-and-forward* non richiede che mittente e destinatario siano online insieme?
2. Qual è la differenza tra **MDA** e **MRA**?
3. Perché un **MTA** può essere sia client sia server SMTP?
4. Il `From:` mostrato dal client è l'indirizzo `MAIL FROM` dell'envelope?
5. Se `From:` e `MAIL FROM` coincidono, questo autentica necessariamente il mittente?
