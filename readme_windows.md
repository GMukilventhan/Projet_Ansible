# Projet Ansible - exploration de l'automatisation Windows
## By Maxime

### Fonctionnement

Ansible utilise WinRM (Windows Remote Management) pour piloter les serveurs Windows, qui est le protocole de gestion à distance de Microsoft.
Il utilise les ports :
HTTP : 5985
HTTPS : 5986

### Pré-requis hôtes Windows

- Powershell 3.0
- .NET Framework 4.0
- 1 utilisateur local (authentification basique)
- Règles de firewall autorisant les ports 5985 ou 5986 (WinRM)
- WinRM activé et son service démarré

### Pré-requis serveur Ansible

- Python 2.7

### Configuration hôtes Windows

S'assurer que Powershell est en version 3.0 au minimum
```bash
Get-Host | Select-Object Version
```

S'assurer que .NET Framework est en version 4.0 au minimum
```bash
Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP' -Recurse | Get-ItemProperty -Name version -EA 0 | Where { $_.PSChildName -Match '^(?!S)\p{L}'} | Select PSChildName, version
```

S'assurer que WinRM n'est pas encore configuré
```bash
winrm get winrm/config/Service
```
Configurer WinRM à l'aide d'un [script](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1)

```bash
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file
```

On s'assure que WinRM a été configuré correctement par le script (commandes de diagnostic) :

Vérification de la configuration du service WinRM
```bash
winrm get winrm/config/Service
```

Vérification de la configuration de Winrs
```bash
winrm get winrm/config/Winrs
```

Vérification de la configuration des "listeners" WinRM
```bash
winrm enumerate winrm/config/Listener
```

Créer un user local dédié pour Ansible
```bash
net user USER_NAME PASSWORD /add
```

Ajout cet user au groupe Administrateurs du poste
```bash
net localgroup administrators USER_ACCOUNT /add
```
### Configuration serveur Ansible

S'assurer qu'une version récente de Python est installée
```bash
python3 --version
```

Installer Ansible
```bash
apt install software-properties-common
add-apt-repository --yes --update ppa:ansible/ansible 
apt install ansible
```

Créer un fichier d'inventaire sous /etc/ansible
```bash
touch inventory
```

Y ajouter les informations sur l'hôte ainsi que les variables de configuration
```bash
[windows]
windows10 ansible_host=IP
[windows:vars]
ansible_user=ansible
ansible_password=PASSWORD
ansible_port=5986
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
```

Indiquer dans le fichier de configuration Ansible (ansible.cfg) le chemin du fichier d'inventaire
```bash
[defaults]
inventory =./inventory
```

Définir NANO comme éditeur de texte par défaut :

Ouvrir le fichier ~/.bashrc
```bash
nano ~/.bashrc
```

Ajouter la ligne suivante
```bash
export EDITOR=nano
```

Création d'un fichier chiffré de variables
```bash
ansible-vault create globalvars/all.yml
```

Editer le fichier et ajouter des variables
```bash
ansible-vault edit globalvars/all.yml
```

Ajouter les variables au format YAML
```bash
---
ansible_user: ansible
ansible_password: Azerty@77
ansible_port: 5986
ansible_connection: winrm
ansible_winrm_transport: basic
ansible_winrm_server_cert_validation: ignore
```


Afin d'éviter de taper le pwd vault à chaque fois, créer un fichier de stockage du pwd
(On placera ce fichier dans le répertoire homme de root)
```bash
touch vault_pwd
```

Ajouter le pwd sans aucune synthaxe particulière
```bash
nano vault_pwd
```

Dans le fichier de configurable d'Ansible (ansible.cfg), indiquer le chemin du fichier de stockage du pwd
```bash
vault_password_file = /root/vault_pwd
```

Création d'un rôle pour ping la machine Windows (winping)\n
Tout d'abord, se placer dans le répertoire roles (etc/ansible/roles)

