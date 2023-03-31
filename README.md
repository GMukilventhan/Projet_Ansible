
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
vault_password_file = /root/.ansible
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

- Créer un fichier main.yml dans le dossier apache2
```bash
vim roles/apache2/tasks/main.yml
```
- Ajouter les tâches
```bash
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

- name: "Vérifier que l’URL répond"
  ansible.builtin.uri:
    url: http://localhost
    status_code: 200
    return_content: yes
  register: result
  ignore_errors: yes
  failed_when: false
  tags: debug

- name: "debug message"
  debug:
    msg: "L’URL répond"
  when: result.status == 200
  tags: debug

- name: "Eror message"
  debug:
    msg: "L’URL ne répond pas "
  when: result.status != 200
  tags: debug

#bonus redemémarrer le service en cas de down
- name: "Restart service after URL ko"
  service:
    name: apache2
    state: restarted
  when: result.status != 200
  tags: debug
```




## Utilisation 
- Pour lancer le playbook, il faut se placer dans le dossier du projet et lancer la commande suivante:
```bash
ansible-playbook projet.yml
```


## Authors

- [GEORGE Mukilventhan](https://github.com/GMukilventhan)
- [PAYEN Théo](https://github.com/theo-payen)
- [WAZANE Mohamed](https://github.com/mowazane)
- [HABERMANN Maxime](https://github.com/MaximeHab)


