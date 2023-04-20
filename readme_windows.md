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
