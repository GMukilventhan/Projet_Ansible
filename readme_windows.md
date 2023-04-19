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

S'assurer que Powershell est en version 3.0
```bash
Get-Host | Select-Object Version
```






