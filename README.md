
# Projet Ansible
## Sujet 
## Description
- Créer un fichier d’inventaire et mettre l’IP d’une machine distante
- Copier la clé SSH du serveur Ansible sur la machine distante (/root/.ssh/authorized_keys)
- Créer un fichier de variable vault et le charger dans le playbook ci-dessous
- Créer un playbook avec les rôles suivants :
- Installer « apache2 » sur le serveur distant
- Copier l’image esgi.jpg dans le dossier /var/www/html
- Copier la template index.j2 dans le chemin /var/www/html/index.html
- Installer le service « ntp »
- Créerlefichier/etc/ntp.conf
- Ajouter les serveurs NTP avec une liste et boucle dans /etc/ntp.conf
- Autoriser le port 80 sur le firewall (iptables)
- Créer une liste d’utilisateurs avec leur mot de passe et faire une boucle dessus pour
la création de ces derniers
- [BONUS] Vérifier que l’URL répond et mettre un message (debug/fail) en fonction du résultat de la vérification

## Documentation

[Documentation Ansible](https://docs.ansible.com/)

## Prérequis
- python
Ansible nécessite une version de python supérieure à 2.6
- Installation de python
```bash
  sudo apt install python
``` 

- Installation d'Ansible

```bash
apt install software-properties-common
 add-apt-repository --yes --update ppa:ansible/ansible 
 apt install ansible
```
- Afin de pouvoir communiquer avec des machines, Ansible a besoin des clefs SSH 
```bash
ssh-keygen -t rsa
```
- Copier la clé publique sur la machine distante
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@ip
```
- Vérifier que la connexion est bien établie
```bash
ssh root@ip
```
- Créer un fichier d'inventaire
```bash
nano /etc/ansible/inventory
```

- Ajouter l'adresse IP de la machine distante
```bash
[client]
ip
```

- Créer un fichier de configuration
```bash
vim ansible.cfg
[defaults]
inventory =./inventory
log_path =./logfile.log
vault_password_file = /root/ansible
#remote_user = root
```

- Attention sur la machine client faut faire la modification suivante
```bash
vim /etc/ssh/sshd_config
PasswordAuthentication yes
```
- relancer le service ssh
```bash
service ssh restart
```

- Créer un fichier de variable vault
```bash
mkdir globalvars
ansible-vault create globalvars/all.yml
```
```bash
touch vault_password
 vim vault_password
mdp
```
- Ajouter les variables
```bash
touch projet.yml
vim projet.yml
```

- Ajouter les roles
```bash
- hosts: CLIENT
  vars_files:
   - "{{ playbook_dir }}/globalvars/all.yml"
```

## Creation de roles
- Créer un dossier apache2
```bash
ansible-galaxy init roles/apache2
```
- Créer un dossier copy
```bash
ansible-galaxy init roles/copy
```
- Créer un dossier ntp
```bash
ansible-galaxy init roles/ntp
```
- Créer un dossier firewall
```bash
ansible-galaxy init roles/firewall
```
- Créer un dossier user
```bash
ansible-galaxy init roles/user
```

## Creation de tâches
## Apache2 + Bonus
- Ouvrir le fichier main.yml dans le dossier apache2
```bash
vim roles/apache2/tasks/main.yml
```
- Ajouter les tâches
```bash
---
# tasks file for roles/apache2

- name: Installation Apache2
  ansible.builtin.apt:
    name: apache2
    state: present
  register: result_apache2

- name: show result_apache2
  debug:
    var: result_apache2

- name: Already installed
  debug:
    msg: "Apache2 is already installed"
  when: result_apache2.changed == false

- name: Restart Apache2
  service:
    name: apache2
    state: restarted
  when: result_apache2.changed == true

- name: "Check that the URL responds"
  ansible.builtin.uri:
    url: http://localhost
    status_code: 200
    return_content: yes
  register: result
  ignore_errors: yes
  failed_when: false

- name: "debug message"
  debug:
    msg: "The URL answers"
  when: result.status == 200

- name: "Error message"
  debug:
    msg: "The URL does not respond"
  when: result.status != 200

    #bonus redemémarrer le service en cas de down
- name: "Restart service after URL ko"
  service:
    name: apache2
    state: restarted
  when: result.status != 200

```
## Copy
Attention il faut créer les fichiers esgi.jpg et index.j2 dans le dossier templates

- Ouvrir le fichier main.yml dans le dossier copy
```bash
vim roles/copy/tasks/main.yml
```
- Ajouter les tâches
```bash
---
# tasks file for roles/copy
#
- name: Copy the esgi.jpg image to the /var/www/html folder
  copy:
    src: /etc/ansible/roles/copy/templates/esgi.jpg
    dest: /var/www/html

- name: Copy index.j2
  template:
    src: /etc/ansible/roles/copy/templates/index.j2
    dest: /var/www/html/index.html
```
- Ajouter les variables
```bash
ansible-vault edit globalvars/all.yml
```
```bash
classe: 4SRC2
groupe: 5
```

## NTP
- Ouvrir le fichier main.yml dans le dossier ntp
```bash
vim roles/ntp/tasks/main.yml
```
- Ajouter les tâches
```bash
---
# tasks file for roles/ntp

- name: Installation of ntp
  ansible.builtin.apt:
    name: ntp
    state: present
  register: result_ntp

- name: show result_ntp
  debug:
    var: result_ntp

- name: Already installed
  debug:
    msg: "ntp is already installed"
  when: result_ntp.changed == false

- name: check file exist
  stat:
    path: "{{ ntp_file_config }}"
  register: check_file

- name: Creation of the ntp file
  file:
    dest: "{{ ntp_file_config }}"
    state: touch
    mode: 0644
  when: not check_file.stat.exists

- name: Add NTP servers with a list and loop in the file
  lineinfile:
    path: "{{ ntp_file_config }}"
    line: "server {{ item }}"
    state: present
  loop: "{{ ntp_server }}"
  register: update_ntp

- name: check  file update          
  set_fact:
    change_status: true
  loop: "{{ update_ntp.results }}"
  when: item.changed

- name: Restart ntp
  service:
    name: ntp
    state: restarted
  when: result_ntp.changed == true or change_status == true or not change_status is defined

```
- Ajouter les variables
```bash
ansible-vault edit globalvars/all.yml
```
```bash
ntp_server:
  - 30.30.30.30
  - 14.14.14.14
  - 5.5.5.5
  - 8.8.8.8
```
- Ajouter le chemin du fichier 
```bash
vim vars/main.yml
```
- Chemin du fichier 
```bash
ntp_file_config: /etc/ntp.conf
```

## Firewall
- Ouvrir le fichier main.yml dans le dossier firewall
```bash
vim roles/firewall/tasks/main.yml
```
- Ajouter les tâches
```bash
---
# tasks file for roles/firewall

- name: Installation of iptables
  ansible.builtin.apt:
    name: iptables
    state: present
  register: result_iptables

- name: show result_iptables
  debug:
    var: result_iptables

- name: Already installed
  debug:
    msg: "iptables is already installed"
  when: result_iptables.changed == false

- name: Allow port 80 on the firewall
  iptables:
    chain: INPUT
    protocol: tcp
    destination_port: 80
    jump: ACCEPT
```

## User

- Ouvrir le fichier main.yml dans le dossier user
```bash
vim roles/user/tasks/main.yml
```

- Ajouter les tâches
```bash
---
# tasks file for roles/users
#
- name: Create users
  user:
    name: "{{ item.Name }}"
    password: "{{ item.Password }}"
    state: present
  loop: "{{ users }}"
  no_log: true

```
- Ajouter les variables
```bash
ansible-vault edit globalvars/all.yml
```
```bash
users:
  - { Name: Mukil, Password: mdp }
  - { Name: Theo, Password: mdp }
  - { Name: Mohamed, Password: mdp }
  - { Name: Maxime, Password: mdp }
```

## Playbook
- Ouvrir le fichier projet.yml et ajouter les roles
```bash
- hosts: CLIENT
  vars_files:
   - "{{ playbook_dir }}/globalvars/all.yml"
  roles:
   - {role: apache2, tags: apache2}
   - {role: copy, tags: copy}
   - {role: ntp, tags: ntp}
   - {role: firewall, tags: firewall}
   - {role: users, tags: users}
```


## Utilisation 
- Pour lancer le playbook, il faut se placer dans le dossier du projet et lancer la commande suivante:
```bash
ansible-playbook projet.yml
```
- Pour lancer un role en particulier, il faut se placer dans le dossier du projet et lancer la commande suivante:
```bash
ansible-playbook projet.yml --tags "nom_du_role"
```
## Repertoire
```bash
/etc/ansible 

├── ansible.cfg 
├── globalvars
│   └── all.yml
├── hosts 
├── inventory 
├── projet.yml 
├── roles 
│   ├── apache2
│   │   └── tasks
│   │       └── main.yml
│   ├── copy
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── esgi.jpg
│   │       └── index.j2
│   ├── firewall
│   │   └── tasks
│   │       └── main.yml
│   ├── ntp
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── vars
│   │       └── main.yml
│   └── users
│       └── tasks
│           └── main.yml
└── logfile.log



```
## Authors
- [GEORGE Mukilventhan](https://github.com/GMukilventhan)
- [PAYEN Théo](https://github.com/theo-payen)
- [WAZANE Mohamed](https://github.com/mowazane)
- [HABERMANN Maxime](https://github.com/MaximeHab)

