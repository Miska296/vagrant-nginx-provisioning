# Vagrant NGINX Provisioning

![MIT License](https://img.shields.io/badge/license-MIT-green.svg)
![Build](https://img.shields.io/badge/build-OK-brightgreen)

---
## 1. Úvodní část
Provisioning webového serveru s NGINX pomocí Ansible v izolovaném prostředí Vagrant. 
Projekt umožňuje bezpečné testování playbooků bez ovlivnění hlavního repozitáře nebo Codespace konfigurace.

Tento projekt vychází z předchozího repozitáře [ansible-web-wm](https://github.com/Miska296/ansible-web-wm), který sloužil jako základní šablona pro roli `webserver` a strukturu playbooku.

> Tento projekt je dostupný také v anglické verzi: [README-en.md](README-en.md)

---
## 2. Požadavky
Než projekt spustíte, ujistěte se, že máte nainstalováno:
- [Vagrant](https://www.vagrantup.com/downloads) (virtuální prostředí)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (poskytovatel VM)
- [Ansible](https://docs.ansible.com/) (automatizace konfigurace)
- Ubuntu 20.04 (hostovaný OS)
- NGINX (webový server)

---
## 3. Jak spustit projekt
  ```bash
  git clone https://github.com/Miska296/vagrant-nginx-provisioning.git
  cd vagrant-nginx-provisioning
  vagrant up
  ```
Po dokončení se vytvoří virtuální stroj s nainstalovaným NGINX serverem. Webová stránka bude dostupná na adrese: `http://localhost:8080`

---
## 4. Cíl projektu
Cílem projektu je automatizované nasazení webového serveru s NGINX pomocí Ansible v lokálním prostředí Vagrant.

---
## 5. Struktura projektu
kořenová složka `vagrant-nginx-provisioning/` obsahuje:
- group_vars/web/vault
- inventory/hosts
- roles/webserver/handlers/main.yml
- roles/webserver/tasks/main.yml
- roles/webserver/templates/index.html.j2
- roles/webserver/templates/nginx.conf.j2
- roles/webserver/vars/main.yml
- playbook.yml
- Vagrantfile
- LICENSE
- README.md
- README-en.md

---
---
# Testovací scénář
## 6. Lokální testování Ansible playbooku ve Vagrant VM
Tento návod popisuje postup, jak otestovat Ansible playbooky ve Vagrant virtuálním prostředí. Testování probíhá izolovaně, bez narušení funkční konfigurace používané v Codespace.

---
### 6.1 Příprava prostředí
Spusť virtuální stroj pomocí Vagrantu:
  ```bash
  vagrant up
  vagrant ssh
  ```
Ujisti se, že složka `/vagrant-nginx-provisioning` obsahuje:
  - `playbook.yml` — hlavní playbook
  - `roles/webserver/` — role pro konfiguraci webserveru
  - `group_vars/web/vault` — šifrovaný soubor s heslem
  - `inventory/hosts` — vlastní inventář pro testování

---
### 6.2 Inventář pro Vagrant
Vytvořen soubor `inventory/hosts` s obsahem:
  ```ini
  [web]
  localhost ansible_connection=local
  ```
Tím zajistíš, že:
- Proměnné z `group_vars/web` se načtou i pro `localhost`
- Ansible nebude používat SSH, ale lokální připojení (`-c local`)
- Vault proměnné budou dostupné pro roli `webserver`
- Testování proběhne přímo ve Vagrant VM bez nutnosti vzdáleného připojení
  
Soubor `inventory/hosts` je klíčový pro správné fungování playbooku. Umožňuje oddělení testovací konfigurace od Codespace a zajišťuje, že Ansible použije lokální připojení bez SSH. Díky tomu se proměnné ze skupiny `web` načtou správně i pro `localhost`, včetně Vault proměnných potřebných pro roli `webserver`.
> Po dokončení tohoto kroku budeš mít připravené prostředí pro bezpečné testování Ansible playbooku ve Vagrant VM.

---
### 6.3 Ansible Vault
Soubor Vault se nachází v `group_vars/web/vault` a obsahuje například proměnnou:
  ```yaml
  webapp_password: tajneheslo123
  ```
Soubor vytvoříš pomocí příkazu:
  ```bash
  ansible-vault create group_vars/web/vault
  ```
Při spuštění playbooku je potřeba zadat heslo pro dešifrování:
  ```bash
  ansible-playbook playbook.yml --ask-vault-pass -i inventory/hosts
  ```
Díky nastavení `ansible_connection=local` v souboru `inventory/hosts` není nutné přidávat parametr `-c local`.

---
### 6.4 Testování vytvoření uživatele
Po úspěšném spuštění playbooku ověř, že byl vytvořen uživatel `webapp`:
  ```bash
  id webapp
  getent passwd webapp
  ```
Očekávaný výstup:
  ```text
  uid=1001(webapp) gid=1002(webapp) groups=1002(webapp)
  webapp:x:1001:1002::/home/webapp:/bin/bash
  ```
Test proběhl úspěšně – uživatel `webapp` byl vytvořen s domovským adresářem a shellem `/bin/bash`.

---
### 6.5 Testování webového serveru
Pokud je součástí provisioning skriptu instalace NGINX:
- Ověř, že NGINX běží:
  ```bash
  sudo systemctl status nginx
  ```
- Pokud neběží, spusť ho:
  ```bash
  sudo systemctl start nginx
  ```
- Testuj dostupnost webu:
> **Poznámka:** V prostředí Codespace není dostupný `systemd`, takže příkaz `sudo systemctl status nginx` nemusí fungovat. Místo toho lze použít:
  ```bash
  curl http://localhost
  ```
Pokud máš ve `Vagrantfile` přesměrování portu:
  ```ruby
  config.vm.network "forwarded_port", guest: 80, host: 8080
  ```
Pak můžeš testovat z hostitelského systému:
  ```bash
  curl http://localhost:8080
  ```

---
## 7. Technické shrnutí role `webserver`
Tato část slouží jako rekapitulace hlavních kroků prováděných rolí `webserver`. Podrobnosti najdete v sekci 6.

### 7.1 Webová služba
- Vytvoření uživatele `webapp` se shellem `/bin/bash`
- Instalace a konfigurace NGINX
- Generování statické webové stránky pomocí šablony `index.html.j2`
- Uložení souboru do `/opt/static-sites`, vlastněné uživatelem `webapp`
- Nastavení oprávnění pro skupinu `www-data`
- Ověření dostupnosti pomocí modulu `uri`

---
### 7.2 Konfigurace NGINX
- Použití šablony `nginx.conf.j2`
- Obsah webu v `/opt/static-sites/index.html`
- Web dostupný na portu `80`, ověřen pomocí:
  ```bash
  curl http://localhost
  ```

---
### 7.3 Kroky provisioning skriptu
- Nasazení konfigurace `sites-available/static-site`
- Aktivace pomocí symlinku do `sites-enabled`
- Nastavení oprávnění pro `www-data`
- Validace dostupnosti pomocí modulu `uri`

---
# Výsledky a ověření
## 8. Výsledky a ověření
Po spuštění se otevře virtuální stroj s nainstalovaným NGINX. Webová stránka je dostupná na `http://localhost:8080` a úspěšně se zobrazuje na portu `80` s obsahem generovaným pomocí Ansible. Funkčnost byla ověřena lokálně ve Vagrant VM.

### 8.1 Co projekt dělá
- Vytvoří Ubuntu virtuální stroj pomocí Vagrantu
- Spustí `provision.sh`, který:
  - Nainstaluje NGINX
  - Zkopíruje testovací HTML stránku do `/var/www/html`
  - Spustí službu NGINX

### 8.2 Testování a ověření
Testovací běh proběhl dne **12. září 2025**. Výsledky:
- Playbook `playbook.yml` proběhl bez chyb (`ok=16`, `changed=12`, `failed=0`)
- Uživatel `webapp` byl úspěšně vytvořen
- Webserver NGINX byl nainstalován, nakonfigurován a restartován
- Webová stránka byla vygenerována pomocí šablony `index.html.j2`
- Výstup `curl http://localhost` obsahoval očekávaný HTML obsah:
  ```html
  <h1>Hello from Ansible-managed NGINX!</h1>
  <p>Server configured automatically by michaela using Ansible</p>
  ```
Tím je potvrzena funkčnost provisioning skriptu v izolovaném prostředí.

---
## 9. Doplňkové poznámky
- Řešení chyby 403 pomocí správného nastavení práv (`www-data`)
- Vlastní konfigurace NGINX (`nginx.conf.j2`)
- Testování v izolovaném prostředí Vagrant
- Projekt je oddělený od Codespace konfigurace

---------------
## 10. Spuštění playbooku v Codespace
Pro lokální testování v Codespace nebo Vagrant VM stačí spustit:
  ```bash
  ansible-playbook playbook.yml --ask-vault-pass -i inventory/hosts
  ```
Tento příkaz:
- Načte inventář z `inventory/hosts`, který obsahuje `ansible_connection=local`
- Použije Vault heslo pro dešifrování proměnných
- Spustí úlohy přímo na lokálním stroji bez SSH
- Automaticky najde roli `webserver` ve složce `roles/`
- Nevyžaduje žádný `provision.sh` skript  

Po úspěšném běhu se webová stránka zobrazí na portu `80`. V Codespace lze port otevřít jako veřejný a získat URL ve formátu např.: https://upgraded-space-trout-7vxgjp7x7pv53wpg-80.app.github.dev/

Zobrazený obsah:
  ```html
  <h1>Hello from Ansible-managed NGINX!</h1>
  <p>Server configured automatically by michaela using Ansible</p>
  ```

---
## 9. Poznámky
- Konfigurace pro Codespace zůstává nedotčena
- Lokální inventář slouží pouze pro testování ve Vagrantu
- Vault proměnné se načítají správně díky přiřazení `localhost` do skupiny `web`
- Projekt je izolovaný od Codespace a hlavního GitHub repozitáře
- Vhodné pro testování, výuku nebo demonstraci provisioning procesů

---
## 🛠 Doporučená vylepšení
### Vylepšení `provision.sh`
...
## Vylepšení provision.sh
Můžeme přidat kontrolu, jestli je NGINX už nainstalovaný:
```bash
if ! command -v nginx &> /dev/null; then
  echo "Installing NGINX..."
  apt-get update
  apt-get install -y nginx
else
  echo "NGINX is already installed."
fi
```

### Přidání testovací stránky
...
## Přidání testovací stránky
Součástí projektu je jednoduchý soubor index.html, který se zobrazí po spuštění.  
Vytvoř soubor index.html s jednoduchým obsahem:
```html
<!DOCTYPE html>
<html>
<head>
  <title>Vagrant NGINX</title>
</head>
<body>
  <h1>Hello from Vagrant NGINX provisioning!</h1>
</body>
</html>
```
A uprav provision.sh, aby ho nakopíroval do /var/www/html:
```bash
cp /vagrant/index.html /var/www/html/index.html
```

### Další kroky
...

🛠 Další kroky
Přidat logování do provision.sh.
Přidat další komponenty (např. firewall, fail2ban).
Vytvořit GitHub Pages dokumentaci.

---
## 10. Autor
Projekt vypracovala [Michaela Kučerová](https://github.com/Miska296)  
Verze: 1.0  
Datum: září 2025

---
## 11. Licence
Tento projekt je dostupný pod licencí MIT. Podrobnosti viz soubor [LICENSE](LICENSE).
