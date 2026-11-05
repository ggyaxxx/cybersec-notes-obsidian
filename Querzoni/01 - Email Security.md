# Lecture 01 — Email Security

**Docente:** Leonardo Querzoni  
**Fonte:** `1 - Email security.pdf`, Sapienza 2026/27  
**Stato:** appunti progressivi; **studiate finora le slide 1–4**. Le sezioni successive verranno aggiunte durante la lezione.

## Obiettivo della lezione (slide 1–2)

Capire architettura e funzionamento dell'email prima di affrontare le minacce e le protezioni. La panoramica del docente comprende SMTP, POP, IMAP, MIME, email threats, SPF/DKIM/ARC/DMARC e protezione end-to-end con PGP/S/MIME (questi ultimi sono **argomenti annunciati**, non ancora studiati qui).

**Distinzione guida:** sicurezza dell'infrastruttura di recapito e sicurezza *end-to-end* del contenuto non sono lo stesso problema.

## Store-and-forward (slide 3)

I sistemi email moderni usano un modello **store-and-forward**: un server riceve e conserva temporaneamente un messaggio prima di inoltrarlo o consegnarlo. Mittente e destinatario **non devono essere online simultaneamente**.

## Internet email architecture (slide 4)

![Email architecture: MUA, MSA, MTA, MDA, Message Store](../assets/email-architecture.svg)

| Componente | Ruolo |
| --- | --- |
| **MUA** — Mail User Agent | Client con cui si scrivono e leggono messaggi |
| **MSA** — Mail Submission Agent | Riceve dal client il messaggio da spedire |
| **MTA** — Mail Transfer Agent | Trasferisce i messaggi tra server via SMTP |
| **MDA** — Mail Delivery Agent | Consegna il messaggio alla casella del destinatario |
| **MS** — Message Store | Memorizza i messaggi recapitati |

**Percorso tipico:** mittente (MUA) → MSA → MTA → [altri MTA] → MDA → Message Store → MUA del destinatario.

**Protocolli:** **SMTP** per submission e trasferimento; **POP/IMAP** per l'accesso ai messaggi conservati nella casella del destinatario.

> I componenti rappresentano **ruoli logici**, non necessariamente macchine fisiche distinte.

**Fonte del disegno:** schema originale ricostruito per questi appunti sulla base della figura della **slide 4** e dell'architettura citata **RFC 5598**; non è una copia della figura della dispensa.

## Domande di verifica (quando riprenderemo)

- Perché email non richiede che entrambi gli utenti siano online?
- Che differenza c'è fra MSA, MTA e MDA?
- Perché SMTP non sostituisce IMAP?
