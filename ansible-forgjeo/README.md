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
│   └── fail2ban/
│       ├── tasks/main.yml   # Installazione e configurazione fail2ban
│       ├── handlers/main.yml
│       ├── templates/
│       │   └── jail.local.j2
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

1. **Aggiorna la cache apt** — scarica gli indici aggiornati dei pacchetti da tutti i repository configurati.
2. **Upgrade di tutti i pacchetti (dist-upgrade)** — aggiorna ogni pacchetto installato all'ultima versione disponibile, gestendo anche dipendenze nuove o rimosse.
3. **Rimuove i pacchetti non più necessari** — elimina le dipendenze orfane, inclusi i relativi file di configurazione.
4. **Pulisce la cache apt** — rimuove i file `.deb` obsoleti da `/var/cache/apt/archives/` per liberare spazio.
5. **Verifica se è necessario un riavvio** — controlla `/var/run/reboot-required` e notifica l'operatore se il server necessita di un reboot.

### Role: fail2ban

Installa e configura fail2ban per la protezione da attacchi brute-force:

1. **Installa fail2ban** — installa il pacchetto tramite apt.
2. **Deploya la configurazione jail.local** — copia il template `jail.local.j2` in `/etc/fail2ban/jail.local`. La configurazione di default banna un host per 1 ora dopo 5 tentativi falliti in 10 minuti. La jail SSH è abilitata.
3. **Abilita e avvia fail2ban** — avvia il demone e lo abilita all'avvio del sistema.
4. **Verifica lo stato** — esegue `fail2ban-client status` e mostra l'output per confermare che il servizio è attivo e le jail sono caricate.
