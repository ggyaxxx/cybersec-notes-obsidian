# Lecture 01 — Introduction to Cybersecurity

**Docente:** Riccardo Lazzeretti  
**Fonte:** `01 introduction.pdf`, Sapienza 2026/27  
**Stato:** sintesi dei concetti affrontati durante il nostro ripasso; non sostituisce le slide.

## Fondamenti (slide 3–20)

- **Security is a process, not a product:** firewall, cifratura e altri strumenti funzionano soltanto all'interno di una gestione coerente della sicurezza.
- **No system is 100% secure:** il livello di protezione richiede una valutazione di rischi, costi e conseguenze.
- **Weakest link:** una componente vulnerabile può compromettere il sistema anche quando altre componenti sono robuste.
- **Security by obscurity non è una garanzia:** la sicurezza non dovrebbe dipendere dalla segretezza del progetto.
- **Cryptography is not enough:** la solidità dell'algoritmo non elimina errori di configurazione, furto di chiavi o compromissione degli endpoint.

### CIA triad

| Proprietà | Domanda essenziale | Esempio |
| --- | --- | --- |
| **Confidentiality** | Chi può conoscere le informazioni? | Impedire letture non autorizzate |
| **Integrity** | Chi può modificare dati e sistemi? | Rilevare alterazioni non autorizzate |
| **Availability** | Il servizio è utilizzabile quando serve? | Resistere a interruzioni e DDoS |

La slide 18 richiama **FIPS 199** e i livelli di impatto *low*, *moderate* e *high*. Proprietà ausiliarie (slide 19): authentication, authorization, non-repudiation, auditing, attack tolerance e disaster recovery.

## Esempi di minacce: DDoS e BGP (slide 33–40)

**DDoS / Mirai:** dispositivi IoT compromessi possono formare una botnet; il caso Dyn (2016) mostra l'effetto di un attacco sul DNS e quindi sulla disponibilità dei servizi.

**BGP:** protocollo di routing *inter-domain*: gli Autonomous Systems (AS) annunciano quali prefissi IP risultano raggiungibili e attraverso quali percorsi.

- **Stub AS:** non offre transito tra AS terzi; può avere più di un collegamento a provider.
- **Transit AS:** trasporta anche traffico fra AS esterni (tipicamente un ISP).
- **Prefix hijacking:** un AS origina un annuncio per un prefisso che non è autorizzato ad annunciare. Un router dell'AS che origina l'annuncio può comunicarlo a vicini mediante **eBGP**; questi devono accettare la rotta perché possa propagarsi.
- **Caso Pakistan Telecom / YouTube, febbraio 2008:** venne annunciato `208.65.153.0/24`, più specifico del prefisso `/22` usato da YouTube nelle slide. Per un indirizzo coperto da entrambe le rotte, il forwarding IP usa il **longest prefix match**, quindi la `/24` se installata. Il traffico fu deviato e YouTube risultò irraggiungibile per una parte degli utenti.

> **Distinzione chiave:** un prefisso **più specifico** ha più bit di rete (`/24` > `/22`), quindi copre meno indirizzi. Non significa percorso fisico più breve o più veloce.

### Precisazione aggiuntiva (non sviluppata nelle slide)

RPKI e Route Origin Validation sono strumenti moderni per verificare l'autorizzazione all'origine dei prefissi; non risolvono da soli ogni forma di hijacking.

## Misure e principi di progettazione (slide 41–45)

- **Security mechanisms:** random numbers, encryption, hash, MAC/HMAC, signatures, key exchange/distribution, timestamping.
- **Requisiti funzionali:** misure *tecniche*, *organizzative* e *miste* (es. configuration management e incident response).
- **Security design principles:** least privilege, fail-safe defaults, complete mediation, open design, separation of privilege, isolation, layering; gli altri principi sono elencati nella slide 44.
- **Distinzioni da ricordare:** *least privilege* limita i diritti assegnati; *separation of privilege* richiede più condizioni per certe operazioni; *complete mediation* richiede controlli sugli accessi alle risorse protette.
- **Security strategy (slide 45):** policy (*cosa richiedere*), mechanisms (*come attuarlo*), assurance (*come verificare che funzioni*).

## Richiamo rapido

1. Come può essere compromessa la confidenzialità senza violare matematicamente AES?
2. Perché un annuncio BGP più specifico può attirare traffico?
3. Qual è la differenza tra *least privilege* e *complete mediation*?
