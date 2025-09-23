# Manuale per nslookup

## 1. Introduzione a nslookup

### Cos'è nslookup?

**nslookup** (name server lookup) è uno strumento a riga di comando utilizzato per:
- **Interrogare** server DNS
- **Risolvere** nomi di dominio in indirizzi IP
- **Verificare** configurazioni DNS
- **Diagnosticare** problemi di risoluzione nomi
- **Esplorare** la gerarchia DNS

### A cosa serve?

- **Troubleshooting di rete**: Identificare problemi di connettività DNS
- **Verifica configurazioni**: Controllare record DNS dopo modifiche
- **Sicurezza**: Investigare domini sospetti
- **Amministrazione**: Gestione zone DNS
- **Apprendimento**: Comprendere il funzionamento DNS

### Limitazioni note

⚠️ **Nota importante**: `nslookup` è considerato **deprecato** da molti amministratori di sistema in favore di strumenti più moderni come `dig` e `host`. Tuttavia, rimane ampiamente utilizzato per la sua semplicità e disponibilità universale.

---

## 3. Sintassi e modalità d'uso

### Sintassi generale

```bash
nslookup [options] [hostname] [DNS-server]
```

### Modalità operative

#### 1. Modalità non-interattiva (one-shot)
```bash
# Query singola
nslookup google.com

# Con server DNS specifico
nslookup google.com 8.8.8.8
```

#### 2. Modalità interattiva
```bash
# Avvio modalità interattiva
nslookup

# Prompt interattivo
> google.com
> set type=MX
> gmail.com
> exit
```

### Parametri principali

| Parametro | Descrizione | Esempio |
|-----------|-------------|---------|
| `hostname` | Nome o IP da risolvere | `google.com` |
| `DNS-server` | Server DNS da utilizzare | `8.8.8.8` |
| `-type=TYPE` | Tipo di record DNS | `-type=MX` |
| `-debug` | Output debug dettagliato | `-debug` |
| `-timeout=N` | Timeout in secondi | `-timeout=10` |

---

## 4. Query DNS di base

### Risoluzione nome → IP (A record)

```bash
# Query A record (default)
nslookup google.com

# Output tipico:
# Server:  8.8.8.8
# Address: 8.8.8.8#53
#
# Non-authoritative answer:
# Name:    google.com  
# Address: 142.250.191.14
```

### Risoluzione inversa IP → nome (PTR record)

```bash
# Reverse DNS lookup
nslookup 8.8.8.8

# Output:
# Server:  192.168.1.1
# Address: 192.168.1.1#53
#
# Non-authoritative answer:
# 8.8.8.8.in-addr.arpa   name = dns.google.
```

### Utilizzo di server DNS specifico

```bash
# Usa Google DNS
nslookup google.com 8.8.8.8

# Usa Cloudflare DNS  
nslookup google.com 1.1.1.1

# Usa server DNS locale
nslookup google.com 192.168.1.1
```

---

## 5. Tipi di record DNS

### A Record (Address)
**Funzione**: Mappa nome dominio → IPv4

```bash
nslookup -type=A google.com

# Oppure modalità interattiva:
> set type=A  
> google.com
```

### AAAA Record (IPv6 Address)
**Funzione**: Mappa nome dominio → IPv6

```bash
nslookup -type=AAAA google.com

# Output esempio:
# google.com has AAAA address 2a00:1450:4002:809::200e
```

### MX Record (Mail Exchange)
**Funzione**: Server di posta per il dominio

```bash
nslookup -type=MX gmail.com

# Output esempio:
# gmail.com mail exchanger = 5 gmail-smtp-in.l.google.com.
# gmail.com mail exchanger = 10 alt1.gmail-smtp-in.l.google.com.
```

**Interpretazione MX**:
- **Numero** (5, 10): Priorità (più basso = maggiore priorità)
- **Nome server**: Server SMTP di destinazione

### CNAME Record (Canonical Name)
**Funzione**: Alias per altro nome dominio

```bash
nslookup -type=CNAME www.github.com

# Output esempio:
# www.github.com canonical name = github.com.
```

### NS Record (Name Server)
**Funzione**: Server DNS autoritativi per il dominio

```bash
nslookup -type=NS google.com

# Output esempio:
# google.com nameserver = ns1.google.com.
# google.com nameserver = ns2.google.com.  
# google.com nameserver = ns3.google.com.
# google.com nameserver = ns4.google.com.
```

### TXT Record
**Funzione**: Record di testo per vari scopi

```bash
nslookup -type=TXT google.com

# Esempi di utilizzo TXT:
# - Verifica proprietà dominio
# - SPF (anti-spam email)
# - DKIM (firma digitale email)
# - DMARC (policy email)
```

### SOA Record (Start of Authority)
**Funzione**: Informazioni sulla zona DNS

```bash
nslookup -type=SOA google.com

# Output esempio:
# google.com
#     origin = ns1.google.com
#     mail addr = dns-admin.google.com  
#     serial = 2023102501
#     refresh = 7200
#     retry = 1800
#     expire = 1209600
#     minimum = 300
```

### PTR Record (Pointer)
**Funzione**: Reverse DNS (IP → nome)

```bash
nslookup -type=PTR 8.8.8.8

# Equivalente a:
nslookup 8.8.8.8
```

---

## 6. Modalità interattiva

### Avvio e comandi base

```bash
# Avvio modalità interattiva
nslookup

# Prompt nslookup
>
```

### Comandi essenziali

#### Set commands (configurazione)

```bash
# Imposta tipo di query
> set type=MX
> set type=NS  
> set type=A

# Imposta server DNS
> server 8.8.8.8
> server 1.1.1.1

# Timeout query (secondi)  
> set timeout=10

# Debug mode
> set debug
> set nodebug

# Query ricorsiva/non ricorsiva
> set recurse
> set norecurse
```

#### Comandi di query

```bash
# Query dominio
> google.com

# Query con server specifico
> google.com 8.8.8.8

# Mostra impostazioni correnti
> set all

# Help
> help

# Exit
> exit
```

### Esempio sessione interattiva completa

```bash
$ nslookup
> set type=MX
> gmail.com
Server:     8.8.8.8
Address:    8.8.8.8#53

Non-authoritative answer:
gmail.com   mail exchanger = 5 gmail-smtp-in.l.google.com.
gmail.com   mail exchanger = 10 alt1.gmail-smtp-in.l.google.com.

> set type=NS
> google.com  
Server:     8.8.8.8
Address:    8.8.8.8#53

Non-authoritative answer:
google.com  nameserver = ns2.google.com.
google.com  nameserver = ns1.google.com.
google.com  nameserver = ns4.google.com.  
google.com  nameserver = ns3.google.com.

> exit
```

---

## 7. Opzioni avanzate

### Debug mode

```bash
# Modalità debug dettagliata
nslookup -debug google.com

# Output dettagliato mostra:
# - Query packet inviato
# - Response packet ricevuto  
# - Parsing dei campi DNS
```

### Timeout e retry

```bash
# Timeout personalizzato (secondi)
nslookup -timeout=5 google.com

# Modalità interattiva
> set timeout=10
> set retry=3
```

### Query non ricorsiva

```bash
# Query diretta (no ricorsione)
nslookup -norecurse google.com

# Utile per testare server specifici senza cache
```

### Query di tutti i record

```bash
# Tutti i record disponibili
nslookup -type=ANY google.com

# ⚠️ Nota: Molti server moderni limitano query ANY
```

### Query con port specifica

```bash
# Modalità interattiva - porta custom
> set port=5353
> google.com
```

---

## 8. Troubleshooting DNS

### Problemi comuni e soluzioni

#### 1. "Server can't find domain: NXDOMAIN"

**Significato**: Dominio non esiste

**Cause**:
- Dominio scritto male
- Dominio scaduto
- Problemi propagazione DNS

**Diagnosi**:
```bash
# Verifica con più server DNS
nslookup nonexistentdomain123.com 8.8.8.8
nslookup nonexistentdomain123.com 1.1.1.1

# Verifica server autoritativi
nslookup -type=NS parentdomain.com
```

#### 2. "Request timed out"

**Significato**: Server DNS non risponde

**Cause**:
- Firewall blocca porta 53
- Server DNS down
- Problemi di rete

**Diagnosi**:
```bash
# Test con server diversi
nslookup google.com 8.8.8.8
nslookup google.com 1.1.1.1

# Aumenta timeout
nslookup -timeout=10 google.com

# Verifica connettività
ping 8.8.8.8
```

#### 3. "Non-authoritative answer"

**Significato**: Risposta da cache, non dal server autoritativo

**Normale**: Indica funzionamento corretto del caching

**Ottenere risposta autoritativa**:
```bash
# Trova server autoritativi
nslookup -type=NS google.com

# Query diretta al server autoritativo
nslookup google.com ns1.google.com
```

### Processo di troubleshooting sistematico

#### Step 1: Verifica DNS locale
```bash
# Test server DNS configurato
nslookup google.com

# Verifica impostazioni DNS locali
# Windows: ipconfig /all
# Linux: cat /etc/resolv.conf
```

#### Step 2: Test server DNS pubblici
```bash
# Google DNS
nslookup google.com 8.8.8.8

# Cloudflare DNS
nslookup google.com 1.1.1.1

# Quad9 DNS
nslookup google.com 9.9.9.9
```

#### Step 3: Verifica gerarchia DNS
```bash
# Root servers
nslookup -type=NS .

# TLD servers (.com)
nslookup -type=NS com.

# Domain servers
nslookup -type=NS google.com
```

#### Step 4: Controllo record specifici
```bash
# A record
nslookup -type=A google.com

# CNAME chains
nslookup -type=CNAME www.google.com

# MX records
nslookup -type=MX google.com
```

---

## 9. Esempi pratici

### Scenario 1: Verifica configurazione sito web

```bash
# 1. Risoluzione base
nslookup www.example.com

# 2. Verifica se è un CNAME
nslookup -type=CNAME www.example.com

# 3. IP finale
nslookup -type=A example.com

# 4. Reverse lookup dell'IP
nslookup 93.184.216.34
```

### Scenario 2: Diagnosi problemi email

```bash
# 1. Record MX
nslookup -type=MX company.com

# 2. A record dei mail server
nslookup mail.company.com

# 3. SPF record (anti-spam)
nslookup -type=TXT company.com | grep -i spf

# 4. DMARC policy
nslookup -type=TXT _dmarc.company.com
```

### Scenario 3: Investigazione dominio sospetto

```bash
# 1. Informazioni base
nslookup suspicious-domain.com

# 2. Server DNS autoritativi
nslookup -type=NS suspicious-domain.com

# 3. Informazioni registrazione
nslookup -type=SOA suspicious-domain.com

# 4. Record TXT (possibili indicatori)
nslookup -type=TXT suspicious-domain.com
```

### Scenario 4: Test after DNS changes

```bash
# 1. Verifica propagazione globale
# Test da più server DNS geograficamente distribuiti

# US East Coast
nslookup newsite.com 8.8.8.8

# Europe  
nslookup newsite.com 1.1.1.1

# Asia
nslookup newsite.com 208.67.222.222

# 2. Flush DNS cache locale
# Windows: ipconfig /flushdns
# Linux: sudo systemctl restart systemd-resolved

# 3. Test propagazione completa
for server in 8.8.8.8 1.1.1.1 9.9.9.9; do
    echo "Testing $server:"
    nslookup newsite.com $server
    echo "---"
done
```

### Scenario 5: Analisi CDN configuration

```bash
# 1. Record principale
nslookup www.site.com

# 2. Verifica CNAME chains
nslookup -type=CNAME www.site.com

# 3. Geolocation testing
# Test da diverse location per vedere CDN endpoints diversi

# 4. Trace CDN edge servers
nslookup -type=CNAME edge.site.com
```

---

## 10. Alternative e strumenti correlati

### dig (Domain Information Groper)

**Vantaggi**: Output più pulito e opzioni avanzate

```bash
# Equivalenti nslookup vs dig
nslookup google.com          # vs # dig google.com
nslookup -type=MX gmail.com  # vs # dig MX gmail.com  
nslookup -x 8.8.8.8         # vs # dig -x 8.8.8.8
```

### host command

**Vantaggi**: Sintassi semplice per query rapide

```bash
# Esempi host command
host google.com
host -t MX gmail.com
host 8.8.8.8
```

### Online DNS tools

**Utilità**: Test da multiple location

- **DNSChecker.org**: Test propagazione globale
- **MXToolbox.com**: Comprehensive DNS/email testing
- **IntoDNS.com**: DNS health check
- **Whatsmydns.net**: Global DNS propagation

### PowerShell DNS cmdlets (Windows)

```powershell
# Modern PowerShell alternative
Resolve-DnsName google.com
Resolve-DnsName -Name gmail.com -Type MX
Resolve-DnsName -Name 8.8.8.8 -Type PTR
```

---

## Cheat Sheet - Comandi essenziali

### Query veloci
```bash
# Risoluzione base
nslookup domain.com

# Con server specifico  
nslookup domain.com 8.8.8.8

# Reverse lookup
nslookup 1.2.3.4

# Record MX
nslookup -type=MX domain.com

# Record NS
nslookup -type=NS domain.com

# Record TXT
nslookup -type=TXT domain.com
```

### Modalità interattiva
```bash
nslookup
> set type=MX
> gmail.com
> server 8.8.8.8  
> set debug
> google.com
> exit
```

### Troubleshooting rapido
```bash
# Test connettività DNS
nslookup google.com

# Test server specifici
for dns in 8.8.8.8 1.1.1.1 9.9.9.9; do
    echo "=== Testing $dns ==="
    nslookup google.com $dns
done

# Debug dettagliato
nslookup -debug -timeout=10 problematic-domain.com
```

---

## Best Practices

### 1. Server DNS affidabili
```bash
# Public DNS servers consigliati:
# Google: 8.8.8.8, 8.8.4.4
# Cloudflare: 1.1.1.1, 1.0.0.1  
# Quad9: 9.9.9.9, 149.112.112.112
```

### 2. Timeout appropriati
```bash
# Per reti lente
nslookup -timeout=10 domain.com

# Per test rapidi
nslookup -timeout=2 domain.com
```

### 3. Verifica multiple sources
```bash
# Non fidarsi di un solo server DNS
# Test con almeno 2-3 server diversi
# Confronta risultati per inconsistenze
```

### 4. Documentazione risultati
```bash
# Salva output per analisi
nslookup domain.com > dns_results.txt

# Timestamp per troubleshooting
echo "$(date): Testing domain.com" >> dns_log.txt
nslookup domain.com >> dns_log.txt
```

---

## Limitazioni di nslookup

### 1. Output parsing difficulty
- Output non strutturato
- Difficile da usare in script
- Formattazione inconsistente

### 2. Feature limitations
- Meno opzioni rispetto a `dig`
- No support per DNSSEC validation
- Limited batch processing

### 3. Platform differences
- Comportamento diverso Windows/Linux
- Opzioni disponibili diverse
- Error messages diverse

---

Ecco tutti i tipi di record DNS che puoi impostare con `set type` in nslookup:

## Set Type completi per nslookup

### Record DNS Standard

| Set Type | Nome Completo | Descrizione | Esempio di Utilizzo |
|----------|---------------|-------------|---------------------|
| **A** | Address | IPv4 address | `set type=A` |
| **AAAA** | IPv6 Address | IPv6 address | `set type=AAAA` |
| **CNAME** | Canonical Name | Alias per altro dominio | `set type=CNAME` |
| **MX** | Mail Exchange | Server di posta | `set type=MX` |
| **NS** | Name Server | Server DNS autoritativi | `set type=NS` |
| **PTR** | Pointer | Reverse DNS (IP→nome) | `set type=PTR` |
| **SOA** | Start of Authority | Informazioni zona DNS | `set type=SOA` |
| **TXT** | Text | Record di testo generico | `set type=TXT` |

### Record DNS Specializzati

| Set Type | Nome Completo | Descrizione | Uso Comune |
|----------|---------------|-------------|------------|
| **SRV** | Service | Localizzazione servizi | `set type=SRV` |
| **HINFO** | Host Information | Info hardware/OS | `set type=HINFO` |
| **MINFO** | Mailbox Information | Info mailbox | `set type=MINFO` |
| **WKS** | Well Known Services | Servizi disponibili | `set type=WKS` |

### Record DNS Sicurezza (DNSSEC)

| Set Type | Nome Completo | Descrizione |
|----------|---------------|-------------|
| **DNSKEY** | DNS Key | Chiavi pubbliche DNSSEC |
| **DS** | Delegation Signer | Firma delegazione |
| **NSEC** | Next Secure | Negative caching sicuro |
| **NSEC3** | NSEC versione 3 | Versione migliorata NSEC |
| **RRSIG** | Resource Record Signature | Firma record DNS |

### Tipi Speciali

| Set Type | Descrizione | Note |
|----------|-------------|------|
| **ANY** | Tutti i record disponibili | ⚠️ Spesso limitato dai server |
| **AXFR** | Zone Transfer completo | ⚠️ Solo per server autoritativi |
| **IXFR** | Zone Transfer incrementale | ⚠️ Raramente accessibile |

## Esempi Pratici di Set Type

### Esempi Base
```bash
nslookup
> set type=A
> google.com

> set type=MX  
> gmail.com

> set type=NS
> google.com

> set type=TXT
> google.com
```

### Esempi Avanzati
```bash
# Record SRV per servizi
> set type=SRV
> _sip._tcp.example.com

# Record DNSKEY per DNSSEC
> set type=DNSKEY
> cloudflare.com

# Tutti i record (se permesso)
> set type=ANY
> example.com
```

### Record TXT Specializzati
I record TXT possono contenere varie informazioni. Esempi comuni:

```bash
# SPF (anti-spam email)
> set type=TXT  
> google.com
# Cerca: "v=spf1..."

# DMARC policy
> set type=TXT
> _dmarc.google.com

# Domain verification
> set type=TXT
> example.com
# Cerca: "google-site-verification=..."
```

## Note Importanti

**⚠️ Limitazioni comuni:**
- **ANY**: Molti server moderni limitano questo tipo di query per prevenire amplificazione attacks
- **AXFR/IXFR**: Zone transfer sono tipicamente ristretti ai server secondari autorizzati
- **Alcuni record** potrebbero non essere supportati da tutti i server DNS

**💡 Suggerimenti:**
- Usa `set type=A` come default (è già il default)
- Per troubleshooting email, inizia sempre con `set type=MX`
- Per investigare gerarchia DNS, usa `set type=NS` e `set type=SOA`
- Per sicurezza/verifiche, controlla `set type=TXT`

Questi sono tutti i principali set type disponibili in nslookup. I più utilizzati nella pratica quotidiana sono A, AAAA, CNAME, MX, NS, PTR, SOA e TXT.
-----------
Certamente! Preparare un manuale per `nslookup` è un'ottima idea, specialmente se vuoi imparare a diagnosticare problemi di rete e DNS. `nslookup` (acronimo di "name server lookup") è uno strumento a riga di comando che permette di interrogare i **server DNS** (Domain Name System) per ottenere informazioni sui nomi di dominio e i loro indirizzi IP. 

Ecco una guida pratica che puoi usare come manuale.

---

## Basi di nslookup

Puoi usare `nslookup` in due modalità: **interattiva** e **non interattiva**. La modalità non interattiva è la più semplice e veloce, mentre quella interattiva è utile per eseguire più query senza dover digitare `nslookup` ogni volta.

* **Modalità non interattiva**: Digita `nslookup` seguito dal nome di dominio che vuoi interrogare.
    * Esempio: `nslookup google.com`

    Questo comando restituirà l'indirizzo IP associato a `google.com` e il server DNS che ha risposto alla tua query.

* **Modalità interattiva**: Digita `nslookup` senza argomenti.
    * Esempio: `nslookup`

    Dopo aver premuto Invio, vedrai un prompt `>`. Da qui, puoi digitare i nomi di dominio o i comandi che ti servono. Per uscire, digita `exit` o `quit`.

---

## Interrogare diversi tipi di record

`nslookup` non si limita a cercare gli indirizzi IP (record A). Puoi specificare il tipo di record DNS che vuoi interrogare usando il comando `set type`. Questo è molto utile per diagnosticare problemi legati a email, record CNAME, o server di nome.

Ecco i tipi di record più comuni che puoi cercare:

* **A**: L'indirizzo IPv4 associato a un nome di dominio. Questo è il tipo di record predefinito.
    * `set type=A`

* **AAAA**: L'indirizzo IPv6 associato a un nome di dominio.
    * `set type=AAAA`

* **MX**: I record **Mail eXchanger**, che indicano i server di posta responsabili della gestione delle email per un dominio.
    * `set type=MX`

* **CNAME**: Il **Canonical Name** di un dominio, ovvero un alias che punta a un altro nome di dominio.
    * `set type=CNAME`

* **TXT**: I record di **testo** che contengono informazioni testuali per il dominio. Spesso usati per la verifica di proprietà del dominio o per record SPF (Sender Policy Framework).
    * `set type=TXT`

* **NS**: I record **Name Server**, che indicano i server DNS autoritativi per un dominio.
    * `set type=NS`

Per usare questi comandi, entra in modalità interattiva (`nslookup`) e poi imposta il tipo di record. Ad esempio, per trovare i record MX di `google.com`:

1.  Digita `nslookup`
2.  Al prompt `>` digita `set type=MX`
3.  Poi digita `google.com`

---

## Esempi pratici e casi d'uso

### Trovare l'IP di un sito web
Il caso d'uso più comune.
* `nslookup www.bing.com`

### Trovare i server di posta per un dominio
Se hai problemi a inviare email, controllare i record MX può aiutarti.
* `nslookup -type=MX google.com`
    * Questo è l'equivalente della modalità non interattiva di `set type=MX` seguito dal dominio.

### Usare un server DNS specifico
A volte è utile interrogare un server DNS diverso da quello predefinito (del tuo provider internet). Ad esempio, puoi usare i server pubblici di Google (8.8.8.8) o di Cloudflare (1.1.1.1).
* `nslookup google.com 8.8.8.8`

Questo comando dirà a `nslookup` di interrogare `google.com` usando il server DNS `8.8.8.8`. È utile per verificare se un problema è legato al server DNS predefinito.

---

## Risoluzione dei problemi comuni

* **Errore "non-existent domain"**: Significa che il nome di dominio non è registrato o è stato digitato in modo errato. Controlla l'ortografia.
* **Tempo scaduto (timeout)**: Il server DNS non ha risposto in tempo. Potrebbe esserci un problema di connettività di rete, un firewall che blocca le query DNS o un server DNS non funzionante.
* **Risposte non autoritative**: Quando `nslookup` restituisce una risposta "non-authoritative", significa che le informazioni non provengono dal server DNS primario del dominio, ma da una cache. È normale e non indica un problema, ma è una cosa da tenere a mente se stai diagnosticando problemi di propagazione dei record DNS.

Spero che questo manuale ti sia utile! `nslookup` è uno strumento fondamentale per chiunque si occupi di amministrazione di rete, sviluppo web o semplicemente vuole capire meglio come funzionano le connessioni a Internet.
