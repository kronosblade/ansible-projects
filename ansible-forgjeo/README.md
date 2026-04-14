# ansible-forgjeo — Deploy Forgejo su Debian

Playbook Ansible che esegue il deploy di un'istanza [Forgejo](https://forgejo.org/) su un server remoto con Debian.

## Struttura del Progetto

```
ansible-forgjeo/
├── ansible.cfg              # Configurazione Ansible
├── hosts.ini                # File inventario
├── playbooks/
│   └── deploy_forgejo.yml   # Playbook principale
├── roles/
│   ├── update/
│   │   ├── tasks/main.yml   # Aggiornamento pacchetti apt
│   │   ├── handlers/main.yml
│   │   ├── vars/main.yml
│   │   └── meta/main.yml
│   ├── security/
│   │   ├── tasks/main.yml   # fail2ban e unattended-upgrades
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   └── jail.local.j2
│   │   ├── vars/main.yml
│   │   └── meta/main.yml
│   └── forgejo/
│       ├── tasks/main.yml   # Installazione Forgejo e dipendenze
│       ├── handlers/main.yml
│       ├── vars/main.yml
│       └── meta/main.yml
└── README.md
```

## Prerequisiti

- Ansible >= 2.9
- Accesso SSH al server Debian di destinazione con privilegi root
- Python installato sull'host di destinazione (richiesto da Ansible)

## Setup

1. Copiare l'inventario di esempio e inserire i dati del proprio server:

```bash
cp hosts.ini.example hosts.ini
```

2. Modificare `hosts.ini` sostituendo l'indirizzo IP, l'utente SSH, la porta e il percorso della chiave secondo la propria configurazione.

## Utilizzo

Eseguire il playbook:

```bash
ansible-playbook -i hosts.ini playbooks/deploy_forgejo.yml
```

Dry run (nessuna modifica applicata):

```bash
ansible-playbook -i hosts.ini playbooks/deploy_forgejo.yml --check
```

## Cosa fa il Playbook

### Role: update

Aggiorna il sistema prima del deploy:

1. **Aggiorna la cache apt**: scarica gli indici aggiornati dei pacchetti da tutti i repository configurati.
2. **Upgrade di tutti i pacchetti (dist-upgrade)**: aggiorna ogni pacchetto installato all'ultima versione disponibile, gestendo anche dipendenze nuove o rimosse.
3. **Rimuove i pacchetti non più necessari**: elimina le dipendenze orfane, inclusi i relativi file di configurazione.
4. **Pulisce la cache apt**: rimuove i file `.deb` obsoleti da `/var/cache/apt/archives/` per liberare spazio.
5. **Verifica se è necessario un riavvio**: controlla `/var/run/reboot-required` e notifica l'operatore se il server necessita di un reboot.

### Role: security

Hardening del server tramite fail2ban e unattended-upgrades.

**fail2ban** protezione da attacchi brute-force:

1. **Installa fail2ban**: installa il pacchetto tramite apt.
2. **Deploya la configurazione jail.local**: copia il template `jail.local.j2` in `/etc/fail2ban/jail.local`. La configurazione di default banna un host per 1 ora dopo 5 tentativi falliti in 10 minuti. La jail SSH è abilitata.
3. **Abilita e avvia fail2ban**: avvia il demone e lo abilita all'avvio del sistema.
4. **Verifica lo stato**: esegue `fail2ban-client status` e mostra l'output per confermare che il servizio è attivo e le jail sono caricate.

**unattended-upgrades**: aggiornamenti di sicurezza automatici:

1. **Installa unattended-upgrades**: installa il pacchetto tramite apt.
2. **Abilita la configurazione**: esegue `dpkg-reconfigure` per attivare gli aggiornamenti automatici.
3. **Abilita e avvia il demone**: avvia il servizio e lo abilita all'avvio del sistema.
4. **Verifica la configurazione**: controlla che `APT::Periodic::Unattended-Upgrade` sia attivo.

### Role: forgejo

Installa il binario Forgejo, le sue dipendenze e prepara le directory di lavoro:

1. **Installa git e git-lfs**: installa i pacchetti necessari al funzionamento di Forgejo tramite apt.
2. **Confronto versioni**: verifica se il binario esiste già e ne estrae la versione installata. Mostra il confronto tra versione installata e versione desiderata (definita in `vars/main.yml`).
3. **Conferma interattiva**: chiede all'operatore il permesso prima di procedere al download. Il messaggio varia a seconda che si tratti di una nuova installazione, un aggiornamento o un re-download della stessa versione.
4. **Scarica il binario Forgejo**: scarica la versione desiderata da Codeberg in `/usr/local/bin/forgejo` con `chmod 755`. Il download avviene solo se l'operatore conferma.
5. **Verifica l'installazione**: esegue `forgejo --version` e mostra l'output.
6. **Crea il gruppo e l'utente di sistema `git`**: utente di sistema con shell `/bin/bash` e home `/home/git`, senza password.
7. **Crea la directory dati `/var/lib/forgejo`**: proprietà `git:git`, permessi `750`.
8. **Crea la directory configurazione `/etc/forgejo`**: proprietà `root:git`, permessi `770`.
