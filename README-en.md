# Vagrant NGINX Provisioning

---
# Introduction part
## 1. Deploying a web server using Vagrant and Ansible
The project serves to test the deployment of a web server using Ansible in an isolated Vagrant environment. It allows for safe testing of playbooks without affecting the main repository or Codespace configuration.

This project is based on a previous repository [ansible-web-wm](https://github.com/Miska296/ansible-web-wm), which served as a basic template for the `webserver` role and the structure of the playbook.
> This project is also available in the Czech version: [README-en.md](README-en.md)

---
## 2. Project goal
Provisioning a web server with NGINX using Ansible in a local Vagrant environment. The web page is generated from a template and is accessible on port `80`.

---
## 3. Used technologies
- Vagrant (virtual environment)
- VirtualBox (VM provider)
- Ansible (configuration automation)
- Ubuntu 20.04 (hosted OS)
- NGINX (web server)

---
## 4. Project structure
root component `vagrant-nginx-provisioning/`:
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

---
---
# Testing scenario
## 5. Local testing of Ansible playbook in Vagrant VM
This guide describes the procedure for testing Ansible playbooks in a Vagrant virtual environment. The testing is carried out in isolation, without disrupting the functional configuration used in Codespace.

---
### 5.1 Preparation of the environment
Start the virtual machine using Vagrant:
  ```bash
  vagrant up
  vagrant ssh
  ```
Make sure that the `/vagrant` folder contains:
  - `playbook.yml` — main playbook
  - `roles/webserver/` — role for web server configuration
  - `group_vars/web/vault` — encrypted file with a password
  - `inventory/hosts` — own inventory for testing

---
### 5.2 Inventory for Vagrant
File `inventory/hosts` created with the content:
  ```ini
  [web]
  localhost ansible_connection=local
  ```
This ensures that:
- Variables from `group_vars/web` will be loaded even for `localhost`.
- Ansible will not use SSH, but local connection (`-c local`)
- The variables vault will be available for the `webserver` role.
- Testing will take place directly in the Vagrant VM without the need for remote connection  
The file `inventory/hosts` is key for the proper functioning of the playbook and its separation from the Codespace configuration.

---
### 5.3 Ansible Vault
The vault file is located in `group_vars/web/vault` and contains the variable:
  ```yaml
  webapp_password: tajneheslo123
  ```
The file is encrypted using:
  ```bash
  ansible-vault create group_vars/web/vault
  ```
When launching the playbook, it is necessary to enter the password:
  ```bash
  ansible-playbook playbook.yml --ask-vault-pass -i inventory/hosts
  ```
Thanks to the setting `ansible_connection=local` in the inventory, there is no need to add the `-c local` parameter.

---
### 5.4 Testing user creation
After successfully launching the playbook, verify that the user `webapp` has been created:
  ```bash
  id webapp
  getent passwd webapp
  ```
Expected output:
  ```text
  uid=1001(webapp) gid=1002(webapp) groups=1002(webapp)
  webapp:x:1001:1002::/home/webapp:/bin/bash
  ```
The test was successful - the user `webapp` was created with a home directory and shell `/bin/bash`.

---
### 5.5 Testování webového serveru
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
## 6. Technické detaily
### 6.1 Webová služba
Role `webserver` provádí následující kroky:
- Vytvoření uživatele `webapp` se shellem `/bin/bash`
- Instalaci a konfiguraci webserveru NGINX
- Generování statické webové stránky pomocí šablony `index.html.j2` s proměnnými `welcome_message` a `admin_user`
- Uložení souboru `index.html` do složky `/opt/static-sites`, vlastněné uživatelem `webapp`
- Nastavení oprávnění pro přístup skupiny `www-data`
- Ověření dostupnosti webové stránky pomocí modulu `uri`

---
### 6.2 Konfigurace NGINX
- Konfigurace pomocí šablony `nginx.conf.j2`
- Obsah webu uložen v `/opt/static-sites/index.html`
- Vlastníkem obsahu je `webapp`, přístup umožněn skupině `www-data`
- Webová stránka dostupná na portu `80`, ověřena pomocí:
  ```bash
  curl http://localhost
  ```

---
### 6.3 Kroky provisioning skriptu
- Nasazení vlastní konfigurace NGINX (`sites-available/static-site`)
- Aktivace konfigurace pomocí symlinku do `sites-enabled`
- Nastavení oprávnění pro přístup uživatele `www-data` ke složce `static-sites`
- Validace dostupnosti webu pomocí modulu `uri`
> Poznámka: Tyto kroky jsou již podrobně popsány v sekcích 6.1 a 6.2 výše. Tato část slouží jako stručné shrnutí provisioning procesu.

---
---
# Výsledky a ověření
## 7. Výsledek
Webová stránka se úspěšně zobrazuje na portu `80` s obsahem generovaným pomocí Ansible. Funkčnost byla ověřena lokálně ve Vagrant VM.

---
### 7.1 Testování a ověření
Testovací běh proběhl dne **12. září 2025**.  
Výsledky:
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
### 7.2 Doplňkové poznámky a tipy
- Řešení chyby 403 pomocí správného nastavení skupinových práv (`www-data`)
- Vlastní konfigurace NGINX mimo výchozí šablonu (`nginx.conf.j2`)
- Testování v izolovaném prostředí Vagrant bez ovlivnění Codespace
- Všechny kroky a výsledky jsou dokumentovány v tomto `README.md`

---
## 8. Spuštění playbooku v Codespace
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
## 10. Autor
Projekt vypracovala [Michaela Kučerová](https://github.com/Miska296)  
Verze: 1.0  
Datum: září 2025

---
## 11. Licence
Tento projekt je dostupný pod licencí MIT. Podrobnosti viz soubor [LICENSE](LICENSE).