# 🐝 Honeypot: Come Costruire un Sistema per Catturare Hacker usando VM Linux
---

## Obiettivi
- Creare un ambiente isolato per attirare attacchi  
- Catturare e registrare attività malevole  
- Analizzare tecniche di intrusione e malware  
- Usare strumenti open source e facilmente accessibili  

---

## Prerequisiti
- PC con almeno 8GB RAM e 50GB spazio libero  
- VirtualBox o VMware  
- ISO Linux (Ubuntu Server o Debian consigliati)  
- Conoscenze base Linux e networking  
- Permessi amministrativi sul PC  

---

## PROCEDIMENTO

### Step 1: Creo la Macchina Virtuale
Creo una nuova VM con almeno 2 CPU e 2GB RAM, installo Linux e aggiorno tutto con:

```bash
sudo apt update && sudo apt upgrade -y
```

Configuro la rete in modalità NAT o Bridge per permettere l’accesso dall’esterno.

---

### Step 2: Installo il Software Honeypot  
Io preferisco **Cowrie**, un honeypot SSH/Telnet molto usato. Puoi installarlo manualmente o tramite uno script che ho preparato per automatizzare tutto:

**Installazione manuale:**

```bash
sudo apt install git python3-virtualenv python3-dev libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind virtualenv -y
git clone https://github.com/cowrie/cowrie.git
cd cowrie
virtualenv --python=python3 cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
cp etc/cowrie.cfg.dist etc/cowrie.cfg
```

**Installazione automatica:**

Scarica e rendi eseguibile lo script `install_cowrie.sh`:

```bash
chmod +x install_cowrie.sh
./install_cowrie.sh
```

Questo script aggiorna il sistema, installa tutte le dipendenze, clona Cowrie e prepara l’ambiente Python.

---

### Step 3: Rendo Sicura la VM  
È fondamentale proteggere la VM per evitare che venga usata come trampolino per attaccare altri sistemi. Ecco cosa faccio:

- Disabilito servizi non necessari per ridurre la superficie d’attacco.
- Configuro un firewall restrittivo (ad esempio con `ufw`) per permettere solo il traffico sulla porta di Cowrie (default 2222).
- Imposto permessi dei file corretti per limitare accessi non autorizzati.
- Attivo logging e backup regolari.

Per facilitare la configurazione firewall puoi eseguire lo script `setup_firewall.sh`:

```bash
chmod +x setup_firewall.sh
./setup_firewall.sh
```

Questo script abilita `ufw`, blocca tutto il traffico in entrata eccetto sulla porta 2222 (porta SSH di Cowrie), e consente il traffico in uscita.

---

### Step 4: Avvio e Monitoraggio
Avvio Cowrie con:

```bash
cd ~/cowrie
source cowrie-env/bin/activate
bin/cowrie start
```

Monitoro i log in tempo reale con:

```bash
tail -f log/cowrie.log
```

Per analisi più avanzate puoi integrare ELK Stack o strumenti di SIEM.

---

### Step 5: Mantengo l’Isolamento  
Per evitare che eventuali compromissioni si propaghino alla rete reale applico:

- Isolamento di rete: uso VLAN o rete NAT isolata nella VM.  
- Snapshot VM: creo snapshot “puliti” prima di ogni test per poter ripristinare rapidamente.  
- Monitoraggio continuo: controllo anomalie e blocco la VM se serve.

---

## Consigli Utili
- Non lasciare mai l’honeypot senza supervisione prolungata  
- Aggiorna sempre sistema e software  
- Usa l’honeypot per ricerca e apprendimento

---

## Risorse
- [Cowrie GitHub](https://github.com/cowrie/cowrie)  
- [Dionaea GitHub](https://github.com/DinoTools/dionaea)  
- [Honeyd GitHub](https://github.com/DataSoft/Honeyd)  
- [VirtualBox](https://www.virtualbox.org/)  

---

## Script di Supporto

### install_cowrie.sh

```bash
#!/bin/bash

set -e

echo "[*] Aggiornamento sistema..."
sudo apt update && sudo apt upgrade -y

echo "[*] Installazione dipendenze..."
sudo apt install -y git python3-virtualenv python3-dev libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind virtualenv

echo "[*] Clono repository Cowrie..."
git clone https://github.com/cowrie/cowrie.git ~/cowrie

cd ~/cowrie

echo "[*] Creo ambiente virtuale Python..."
virtualenv --python=python3 cowrie-env
source cowrie-env/bin/activate

echo "[*] Aggiorno pip e installo dipendenze Python..."
pip install --upgrade pip
pip install -r requirements.txt

echo "[*] Copio file di configurazione..."
cp etc/cowrie.cfg.dist etc/cowrie.cfg

echo "[*] Configuro authbind per permessi porte <1024..."
sudo touch /etc/authbind/byport/22
sudo chown $(whoami) /etc/authbind/byport/22
sudo chmod 755 /etc/authbind/byport/22

echo "[*] Installazione completata."
echo "Avvia Cowrie con:"
echo "    cd ~/cowrie"
echo "    source cowrie-env/bin/activate"
echo "    bin/cowrie start"
```

---

### setup_firewall.sh

```bash
#!/bin/bash

set -e

echo "[*] Abilito ufw..."
sudo ufw enable

echo "[*] Resetto regole correnti..."
sudo ufw reset

echo "[*] Permetto solo SSH su porta 2222..."
sudo ufw allow 2222/tcp

echo "[*] Blocca tutto il resto..."
sudo ufw default deny incoming
sudo ufw default allow outgoing

echo "[*] Stato firewall:"
sudo ufw status verbose

echo "[*] Firewall configurato."
```

---

## Licenza
MIT License — sentiti libero di usare e modificare questo progetto.

Se vuoi che lo carichi tutto in un repo GitHub e ti mando il link, basta chiedere!

