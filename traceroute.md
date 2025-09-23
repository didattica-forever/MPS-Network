# Manuale traceroute

## 1. Introduzione a traceroute

### Cos'è traceroute?

**Traceroute** è uno strumento di diagnostica di rete che:
- **Traccia il percorso** che i pacchetti seguono per raggiungere una destinazione
- **Misura i tempi di risposta** per ogni hop (salto) lungo il percorso  
- **Identifica router intermedi** attraverso cui passano i dati
- **Rivela problemi di routing** e colli di bottiglia nella rete

### A cosa serve?

- **Troubleshooting connettività**: Individuare dove si interrompe la connessione
- **Analisi performance**: Identificare hop con latenza elevata
- **Mapping della rete**: Visualizzare la topologia fisica della rete
- **Verifica routing**: Controllare che i pacchetti seguano il percorso corretto
- **Sicurezza**: Identificare percorsi anomali o sospetti

### Varianti per sistema operativo

| Sistema | Comando | Note |
|---------|---------|------|
| **Linux/Unix/macOS** | `traceroute` | Versione completa con molte opzioni |
| **Windows** | `tracert` | Versione semplificata |
| **Windows PowerShell** | `Test-NetConnection -TraceRoute` | Versione moderna |

---

## 3. Come funziona traceroute

### Principio di funzionamento

Traceroute sfrutta il campo **TTL (Time To Live)** dell'header IP:

1. **TTL=1**: Primo pacchetto con TTL=1
   - Il primo router decrementa TTL a 0
   - Router invia ICMP "Time Exceeded" 
   - Traceroute registra questo router come Hop 1

2. **TTL=2**: Secondo pacchetto con TTL=2
   - Primo router decrementa TTL a 1, inoltra
   - Secondo router decrementa TTL a 0
   - Secondo router invia ICMP "Time Exceeded"
   - Traceroute registra questo router come Hop 2

3. **Continua** aumentando TTL fino a raggiungere la destinazione

### Diagramma del processo

```
Client ────TTL=1───→ Router1 ────────────→ Router2 ────────→ Destination
       ←─Time Exceeded─                           
       
Client ─────────────→ Router1 ──TTL=2──→ Router2 ────────→ Destination
                                      ←─Time Exceeded─      
                                      
Client ─────────────→ Router1 ─────────→ Router2 ──TTL=3→ Destination
                                                   ←─Reply─
```

### Protocolli utilizzati

#### Linux/Unix traceroute:
- **Default**: UDP con porte alte (33434+)
- **Opzione -I**: ICMP Echo Request
- **Opzione -T**: TCP SYN packets

#### Windows tracert:
- **Default**: ICMP Echo Request

---

## 4. Sintassi e opzioni

### Sintassi base

```bash
# Linux/macOS
traceroute [opzioni] destinazione

# Windows  
tracert [opzioni] destinazione
```

### Opzioni principali Linux/macOS

| Opzione | Descrizione | Esempio |
|---------|-------------|---------|
| `-n` | Non risolve nomi (solo IP) | `traceroute -n google.com` |
| `-w timeout` | Timeout in secondi | `traceroute -w 5 google.com` |
| `-q queries` | Numero probe per hop | `traceroute -q 1 google.com` |
| `-m max_hops` | Massimo numero hop | `traceroute -m 15 google.com` |
| `-p port` | Porta di destinazione | `traceroute -p 80 google.com` |
| `-I` | Usa ICMP invece di UDP | `traceroute -I google.com` |
| `-T` | Usa TCP SYN | `traceroute -T -p 80 google.com` |
| `-f first_ttl` | TTL iniziale | `traceroute -f 3 google.com` |
| `-s source` | Indirizzo sorgente | `traceroute -s 192.168.1.100 google.com` |

### Opzioni principali Windows

| Opzione | Descrizione | Esempio |
|---------|-------------|---------|
| `-d` | Non risolve nomi | `tracert -d google.com` |
| `-h max_hops` | Massimo hop | `tracert -h 15 google.com` |
| `-w timeout` | Timeout in ms | `tracert -w 5000 google.com` |
| `-4` | Forza IPv4 | `tracert -4 google.com` |
| `-6` | Forza IPv6 | `tracert -6 google.com` |

---

## 5. Interpretazione dei risultati

### Output tipico Linux

```bash
$ traceroute google.com
traceroute to google.com (142.250.191.14), 30 hops max, 60 byte packets
 1  router.local (192.168.1.1)  1.234 ms  1.456 ms  1.123 ms
 2  10.0.0.1 (10.0.0.1)  5.678 ms  5.432 ms  5.890 ms
 3  isp-gateway.net (203.0.113.1)  15.234 ms  14.567 ms  15.678 ms
 4  * * *
 5  google-router.net (172.217.164.1)  25.123 ms  24.567 ms  26.234 ms
 6  142.250.191.14 (142.250.191.14)  26.789 ms  25.234 ms  27.123 ms
```

### Output tipico Windows

```cmd
C:\> tracert google.com
Tracciamento instradamento verso google.com [142.250.191.14] su un massimo di 30 punti di passaggio:
  1     1 ms     1 ms     2 ms  192.168.1.1
  2     5 ms     6 ms     5 ms  10.0.0.1
  3    15 ms    14 ms    16 ms  203.0.113.1
  4     *        *        *     Richiesta scaduta.
  5    25 ms    24 ms    26 ms  172.217.164.1
  6    26 ms    25 ms    27 ms  google.com [142.250.191.14]
```

### Interpretazione dei campi

#### Colonna 1: Hop number
```
1, 2, 3, 4, 5, 6...
```
- Numero sequenziale del router nel percorso
- Inizia da 1 (primo router dopo il client)

#### Colonna 2: Indirizzo IP e hostname
```
router.local (192.168.1.1)
10.0.0.1 (10.0.0.1)
```
- **Nome risolto**: Se disponibile reverse DNS
- **IP tra parentesi**: Indirizzo IP del router
- **Solo IP**: Se reverse DNS non disponibile o -n usato

#### Colonne 3-5: Tempi di risposta
```
1.234 ms  1.456 ms  1.123 ms
```
- **Tre misurazioni**: RTT (Round Trip Time) in millisecondi
- **Tre probe**: Per ogni hop vengono inviati 3 pacchetti
- **Variabilità**: Differenze indicano jitter/instabilità

#### Asterischi: Timeout
```
4     * * *
```
- **Significato**: Router non ha risposto entro il timeout
- **Cause possibili**:
  - Firewall blocca ICMP
  - Router configurato per non rispondere
  - Congestione di rete
  - Pacchetti persi

---

## 6. Opzioni avanzate

### Controllo del protocollo

#### UDP traceroute (default Linux)
```bash
# Default: UDP con porte 33434+
traceroute google.com

# Specifica porta UDP
traceroute -p 53 google.com
```

#### ICMP traceroute  
```bash
# Usa ICMP Echo Request (come Windows)
traceroute -I google.com

# Più probabile che attraversi firewall
sudo traceroute -I google.com
```

#### TCP traceroute
```bash
# TCP SYN a porta specifica
sudo traceroute -T -p 80 google.com
sudo traceroute -T -p 443 google.com
sudo traceroute -T -p 22 google.com
```

### Controllo timing

#### Timeout personalizzato
```bash
# Timeout 10 secondi (default 5)
traceroute -w 10 google.com

# Windows: timeout in millisecondi
tracert -w 10000 google.com
```

#### Numero di probe
```bash
# Solo 1 probe per hop (più veloce)
traceroute -q 1 google.com

# 5 probe per hop (più accurato)  
traceroute -q 5 google.com
```

### Controllo del percorso

#### Limitare numero hop
```bash
# Massimo 10 hop
traceroute -m 10 google.com

# Inizia dal 3° hop (salta i primi 2)
traceroute -f 3 google.com
```

#### Indirizzo sorgente specifico
```bash
# Usa interfaccia specifica
traceroute -s 192.168.1.100 google.com

# Utile per multi-homed hosts
traceroute -s eth1 google.com
```

### IPv6 traceroute

```bash
# Linux
traceroute6 google.com
traceroute -6 google.com

# Windows
tracert -6 google.com
```

---

## 7. Troubleshooting con traceroute

### Identificare problemi comuni

#### 1. Timeout completo su un hop
```
 8     * * *
 9    45 ms   46 ms   44 ms   router.next.com
```

**Possibili cause:**
- Router non risponde a ICMP/UDP
- Firewall blocca traffico
- Router sovraccarico

**Azioni:**
```bash
# Prova protocolli diversi
traceroute -I google.com
sudo traceroute -T -p 80 google.com

# Verifica se il traffico passa comunque
ping -c 1 google.com
```

#### 2. Latenza improvvisa alta
```
 5    12 ms   11 ms   13 ms   router1.isp.com
 6   156 ms  145 ms  167 ms   router2.isp.com
 7    25 ms   24 ms   26 ms   router3.isp.com
```

**Diagnosi**: Hop 6 ha latenza anomala

**Possibili cause:**
- Collo di bottiglia
- Percorso geografico lungo
- Congestione temporanea

#### 3. Percorso asimmetrico
```bash
# Traceroute di andata
traceroute destination.com

# Traceroute di ritorno (dal destination)
# Spesso mostra percorso diverso
```

#### 4. Loop di routing
```
15   router1.problem.net  25 ms  26 ms  24 ms
16   router2.problem.net  27 ms  28 ms  26 ms  
17   router1.problem.net  25 ms  26 ms  24 ms
18   router2.problem.net  27 ms  28 ms  26 ms
```

### Metodologia di troubleshooting

#### Step 1: Traceroute base
```bash
traceroute target.com
```

#### Step 2: Se timeout, prova ICMP  
```bash
traceroute -I target.com
```

#### Step 3: Se ancora problemi, prova TCP
```bash
sudo traceroute -T -p 80 target.com
sudo traceroute -T -p 443 target.com
```

#### Step 4: Analisi comparativa
```bash
# Confronta con target funzionante
traceroute google.com
traceroute target.com

# Identifica dove divergono
```

#### Step 5: Test bidirezionale
```bash
# Dal tuo host al target
traceroute target.com

# Dal target al tuo host (se possibile)
# Oppure usa online looking glass tools
```

---

## 8. Esempi pratici

### Scenario 1: Diagnostica connessione lenta

```bash
# 1. Test base
traceroute slow-website.com

# Output esempio:
#  1  router.home (192.168.1.1)      2 ms    1 ms    2 ms
#  2  isp.local (10.0.0.1)          8 ms    7 ms    9 ms  
#  3  regional.isp.com (203.0.113.1) 15 ms   14 ms   16 ms
#  4  backbone.net (198.51.100.1)    45 ms   44 ms   46 ms
#  5  overseas.net (192.0.2.1)      156 ms  145 ms  167 ms  ← PROBLEMA
#  6  destination.net (93.184.216.1) 158 ms  147 ms  169 ms

# Analisi: Hop 5 mostra salto latenza significativo
# Probabile collegamento intercontinentale

# 2. Verifica con ping sostenuto
ping -c 20 slow-website.com

# 3. Confronto con sito veloce
traceroute fast-website.com
```

### Scenario 2: Sito irraggiungibile

```bash
# 1. Traceroute base
traceroute unreachable-site.com

# Output:
#  1  router.home (192.168.1.1)     1 ms    2 ms    1 ms
#  2  isp.gateway (10.0.0.1)        5 ms    6 ms    5 ms
#  3  * * *
#  4  * * *
#  5  * * *

# 2. Prova ICMP
sudo traceroute -I unreachable-site.com

# 3. Prova TCP su porte comuni
sudo traceroute -T -p 80 unreachable-site.com  
sudo traceroute -T -p 443 unreachable-site.com

# 4. Verifica DNS prima
nslookup unreachable-site.com

# 5. Test con IP diretto se DNS funziona
traceroute 1.2.3.4
```

### Scenario 3: Analisi prestazioni CDN

```bash
# 1. Traceroute verso CDN
traceroute cdn.example.com

# Output tipico CDN ben configurato:
#  1  home.router (192.168.1.1)     1 ms    1 ms    2 ms
#  2  isp.local (10.0.0.1)          5 ms    6 ms    5 ms
#  3  regional.isp (203.0.113.1)    12 ms   11 ms   13 ms
#  4  cdn.edge.server (198.51.100.5) 15 ms   14 ms   16 ms  ← Edge vicino

# 2. Confronto con origin server
traceroute origin.example.com

# Output origin server distante:
#  1-3  [stessi hop locali]
#  4-8  [più hop intermedi]
#  9    origin.far.away (192.0.2.100) 85 ms   82 ms   87 ms
```

### Scenario 4: Verifica failover/load balancing

```bash
# 1. Multiple traceroute verso stesso target
for i in {1..5}; do
    echo "=== Traceroute $i ==="
    traceroute -q 1 load-balanced.com
    sleep 2
done

# Possibili risultati:
# - Stesso percorso: Single path
# - Percorsi diversi: Load balancing/ECMP
# - IP destinazione diversi: DNS round-robin
```

### Scenario 5: Investigazione sicurezza

```bash
# 1. Traceroute verso IP sospetto
traceroute 1.2.3.4

# 2. Analizza percorso per:
# - Paesi attraversati (geolocalizzazione IP)
# - ISP coinvolti
# - Hop anomali o non risolubili

# 3. Reverse DNS degli hop
for ip in $(traceroute -n target | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'); do
    echo "=== $ip ==="
    nslookup $ip
done
```

---

## 9. Varianti e alternative

### mtr (My TraceRoute)

**Vantaggi**: Combina traceroute e ping con interfaccia continua

```bash
# Installazione Linux
sudo apt-get install mtr
sudo yum install mtr

# Utilizzo
mtr google.com

# Report mode (non interattivo)
mtr --report google.com

# Numero pacchetti specifico
mtr --report --report-cycles 10 google.com
```

**Output mtr:**
```
                     My traceroute  [v0.92]
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                           Packets               Pings
 Host                                    Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. router.home                           0.0%    10    1.2   1.3   1.0   2.1   0.3
 2. isp.gateway                           0.0%    10    5.4   5.6   4.9   7.2   0.8
 3. backbone.net                          0.0%    10   15.2  15.8  14.1  18.9   1.4
 4. google.com                            0.0%    10   25.1  25.9  24.2  28.7   1.2
```

### tcptraceroute

**Scopo**: Traceroute specifico per TCP

```bash
# Installazione
sudo apt-get install tcptraceroute

# Utilizzo
sudo tcptraceroute google.com 80
sudo tcptraceroute gmail.com 443
```

### Visual traceroute tools

#### Applicazioni grafiche:
- **WinMTR** (Windows): Interfaccia grafica per mtr
- **PathPing** (Windows): Versione Microsoft avanzata
- **VisualRoute**: Tool commerciale con mappe geografiche

### Online traceroute services

#### Looking Glass servers:
```
# Esempio URLs (cambiano nel tempo):
# - lg.he.net (Hurricane Electric)
# - lg.cogentco.com (Cogent)  
# - lg.level3.net (Level 3)
```

**Utilità**: Test traceroute da diverse location geografiche

### PowerShell avanzato (Windows)

```powershell
# Test-NetConnection con traceroute
Test-NetConnection -ComputerName google.com -TraceRoute

# Traceroute personalizzato PowerShell
1..30 | ForEach-Object {
    $result = Test-NetConnection -ComputerName google.com -Hops $_
    if ($result.TraceRoute) {
        "$($_): $($result.TraceRoute[-1]) - $($result.PingSucceeded)"
    }
}
```

---

## 10. Best practices

### 1. Scelta del protocollo

```bash
# Per troubleshooting generale
traceroute target.com          # Prova UDP prima

# Se UDP bloccato  
traceroute -I target.com       # ICMP spesso passa

# Per servizi web specifici
sudo traceroute -T -p 80 target.com   # HTTP
sudo traceroute -T -p 443 target.com  # HTTPS
```

### 2. Interpretazione dei timeout

```bash
# Non sempre i * indicano problemi
# Router può essere configurato per non rispondere
# Verifica se il traffico passa comunque:

# Se traceroute mostra:
#  5  * * *
#  6  router.next.com  25ms 24ms 26ms

# Il hop 5 probabilmente funziona ma non risponde
```

### 3. Raccolta dati sistematica

```bash
# Script per multiple misurazioni
#!/bin/bash
TARGET="problematic-site.com"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "=== Traceroute Analysis $TIMESTAMP ===" > trace_$TIMESTAMP.txt
echo "Target: $TARGET" >> trace_$TIMESTAMP.txt
echo "" >> trace_$TIMESTAMP.txt

# Test UDP
echo "=== UDP Traceroute ===" >> trace_$TIMESTAMP.txt
traceroute $TARGET >> trace_$TIMESTAMP.txt 2>&1
echo "" >> trace_$TIMESTAMP.txt

# Test ICMP  
echo "=== ICMP Traceroute ===" >> trace_$TIMESTAMP.txt
sudo traceroute -I $TARGET >> trace_$TIMESTAMP.txt 2>&1
echo "" >> trace_$TIMESTAMP.txt

# Test TCP
echo "=== TCP Traceroute (port 80) ===" >> trace_$TIMESTAMP.txt
sudo traceroute -T -p 80 $TARGET >> trace_$TIMESTAMP.txt 2>&1
```

### 4. Confronto temporale

```bash
# Salva traceroute di baseline quando tutto funziona
traceroute critical-service.com > baseline_trace.txt

# Durante problemi, confronta
traceroute critical-service.com > problem_trace.txt
diff baseline_trace.txt problem_trace.txt
```

### 5. Considerazioni geografiche

```bash
# Per servizi globali, usa mtr per statistiche
mtr --report --report-cycles 20 global-service.com

# Identifica se latenza è geograficamente giustificata:
# - Europa ↔ USA: ~80-120ms normale  
# - Asia ↔ Europa: ~150-200ms normale
# - Transcontinentale: >200ms può essere normale
```

### 6. Sicurezza e privacy

```bash
# Per investigazioni sicurezza, salva tutto
traceroute -n suspicious-host.com > investigation.txt

# Non fare traceroute frequenti verso stesso target
# Può essere interpretato come reconnaissance

# Usa -n per evitare reverse DNS queries che rivelano interesse
```

---

## Troubleshooting Common Issues

### Issue 1: "Operation not permitted"

```bash
# Errore comune su Linux
$ traceroute google.com
traceroute: socket: Operation not permitted

# Soluzioni:
sudo traceroute google.com           # Usa sudo
traceroute -I google.com            # Prova ICMP (potrebbe richiedere sudo)
setcap cap_net_raw=eip /usr/bin/traceroute  # Permessi permanenti
```

### Issue 2: "No route to host"  

```bash
# Verifica connettività di base prima
ping 8.8.8.8                        # Test connessione Internet
ip route show                       # Verifica routing table
traceroute 192.168.1.1              # Test gateway locale
```

### Issue 3: Tutti asterischi

```bash
# Se tutto sono asterischi:
traceroute -I -w 10 target.com      # Più tempo, ICMP
sudo traceroute -T -p 80 target.com # TCP su porta web
mtr --report target.com              # Alternative tool
```

---

## Cheat Sheet - Comandi essenziali

### Quick diagnostics
```bash
# Basic trace
traceroute google.com

# Fast trace (1 probe per hop)
traceroute -q 1 google.com

# No DNS resolution  
traceroute -n google.com

# ICMP trace
traceroute -I google.com

# TCP trace to web server
sudo traceroute -T -p 80 website.com
```

### Windows equivalents
```cmd
tracert google.com
tracert -d google.com                # No DNS
tracert -h 15 google.com            # Max 15 hops
tracert -w 5000 google.com          # 5 second timeout
```

### Advanced analysis
```bash
# MTR continuous monitoring
mtr google.com

# Multiple protocol test
traceroute google.com && traceroute -I google.com && sudo traceroute -T -p 443 google.com

# Path comparison
traceroute site1.com > path1.txt && traceroute site2.com > path2.txt && diff path1.txt path2.txt
```

---

## Limitazioni e considerazioni

### 1. Firewall interference
- Molti firewall bloccano ICMP o UDP
- Router possono essere configurati per non rispondere
- Traceroute TCP spesso più affidabile per servizi web

### 2. Load balancing effects
- Percorsi possono variare tra esecuzioni
- ECMP (Equal Cost Multi-Path) routing
- Geographic load balancing

### 3. Rate limiting
- Router possono limitare risposte ICMP
- Causing artificial timeouts
- Non sempre indica problemi reali

### 4. IPv6 considerations  
- Percorsi IPv4 e IPv6 possono differire
- Dual-stack environments complessi
- Usa traceroute6 o -6 option
