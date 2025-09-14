# Vagrant NGINX Provisioning

---
# Úvodní část
## 1. Nasazení webového serveru pomocí Vagrant a Ansible
Projekt slouží k otestování nasazení webového serveru pomocí Ansible v izolovaném prostředí Vagrant. Umožňuje bezpečné testování playbooků bez ovlivnění hlavního repozitáře nebo Codespace konfigurace.

Tento projekt vychází z předchozího repozitáře [ansible-web-wm](https://github.com/Miska296/ansible-web-wm), který sloužil jako základní šablona pro roli `webserver` a strukturu playbooku.

---
## 2. Cíl projektu
Provisioning webového serveru s NGINX pomocí Ansible v lokálním prostředí Vagrant. Webová stránka je generována šablonou a dostupná na portu `80`.

---
## 3. Použité technologie
- Vagrant (virtuální prostředí)
- VirtualBox (poskytovatel VM)
- Ansible (automatizace konfigurace)
- Ubuntu 20.04 (hostovaný OS)
- NGINX (webový server)

---
## 4. Struktura projektu
kořenová složka `vagrant-nginx-provisioning/`:
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
# Testovací scénář
## 5. Lokální testování Ansible playbooku ve Vagrant VM
Tento návod popisuje postup, jak otestovat Ansible playbooky ve Vagrant virtuálním prostředí. Testování probíhá izolovaně, bez narušení funkční konfigurace používané v Codespace.

---
### 5.1 Příprava prostředí
Spusť virtuální stroj pomocí Vagrantu:
  ```bash
  vagrant up
  vagrant ssh
  ```
Ujisti se, že složka `/vagrant` obsahuje:
  - `playbook.yml` — hlavní playbook
  - `roles/webserver/` — role pro konfiguraci webserveru
  - `group_vars/web/vault` — šifrovaný soubor s heslem
  - `inventory/hosts` — vlastní inventář pro testování

---
### 5.2 Inventář pro Vagrant
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
Soubor `inventory/hosts` je klíčový pro správné fungování playbooku a jeho oddělení od Codespace konfigurace.

---
### 5.3 Ansible Vault
Vault soubor se nachází v `group_vars/web/vault` a obsahuje proměnnou:
  ```yaml
  webapp_password: tajneheslo123
  ```
Soubor je šifrován pomocí:
  ```bash
  ansible-vault create group_vars/web/vault
  ```
Při spuštění playbooku je nutné zadat heslo:
  ```bash
  ansible-playbook playbook.yml --ask-vault-pass -i inventory/hosts
  ```
Díky nastavení `ansible_connection=local` v inventáři není nutné přidávat parametr `-c local`.

---
### 5.4 Testování vytvoření uživatele
Po úspěšném spuštění playbooku ověř, že uživatel `webapp` byl vytvořen:
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