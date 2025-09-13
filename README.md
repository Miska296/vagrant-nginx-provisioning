# Vagrant NGINX Provisioning

---
---
# Úvodní část
## 1. Nasazení webového serveru pomocí Vagrant a Ansible
Projekt slouží k otestování nasazení webového serveru pomocí Ansible v izolovaném prostředí Vagrant. Umožňuje bezpečné testování playbooků bez ovlivnění hlavního repozitáře nebo Codespace konfigurace.

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
---
# Testovací scénář
## 4. Lokální testování Ansible playbooku ve Vagrant VM
Tento návod popisuje postup, jak otestovat Ansible playbooky ve Vagrant virtuálním prostředí bez narušení funkční konfigurace používané v Codespace.

---
## 5. Příprava prostředí
Spusť virtuální stroj pomocí Vagrantu:
  ```bash
  vagrant up
  vagrant ssh
  ```
Ujisti se, že složka `/vagrant` obsahuje:
  - `users-test.yml` — testovací playbook
  - `roles/users/tasks/main.yml` — role pro vytvoření uživatele
  - `group_vars/web/vault` — šifrovaný soubor s heslem
  - `provision.sh` — provisioning skript
  - `inventory/hosts` — vlastní inventář pro testování ve Vagrantu

---
## 6. Inventář pro Vagrant
Vytvoř soubor `/vagrant/inventory/hosts` s obsahem:
  ```ini
  [web]
  localhost
  ```
Tím zajistíš, že proměnné z `group_vars/web` se načtou i pro `localhost`.

---
## 7. Ansible Vault
Vault soubor se nachází v `group_vars/web/vault` a obsahuje proměnnou:
  ```yaml:
  webapp_password: tajneheslo123
  ```
Soubor je šifrován pomocí:
  ```bash
  ansible-vault create group_vars/web/vault
  ```
Při spuštění playbooku je nutné zadat heslo:
  ```bash
  ansible-playbook users-test.yml --ask-vault-pass -i /vagrant/inventory/hosts
  ```

---
## 8. Testování vytvoření uživatele
Po úspěšném spuštění playbooku ověř, že uživatel byl vytvořen:
  ```bash
  id webapp
  getent passwd webapp
  ```

---
## 9. Testování webového serveru
Pokud je součástí provisioning skriptu instalace NGINX:
- Ověř, že běží:
  ```bash
  sudo systemctl status nginx
  ```
- Pokud neběží, spusť ho:
  ```bash
  sudo systemctl start nginx
  ```
- Testuj dostupnost webu:
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
---
# Technické detaily
## 10. Webová služba
Role webserver provádí:
- Vytvoření uživatele `webapp`
- Instalaci a konfiguraci NGINX
- Klonování statického webu z GitHubu
- Generování `index.html` pomocí šablony `index.html.j2` s proměnnými `welcome_message` a `admin_user`
- Uložení obsahu do `/opt/static-sites`, vlastněného uživatelem `webapp`
- Ověření dostupnosti webu pomocí modulu `uri`

---
## 11. Konfigurace NGINX
- Konfigurace pomocí šablony `nginx.conf.j2`
- Obsah webu uložen v `/opt/static-sites/index.html`
- Vlastníkem obsahu je `webapp`, přístup umožněn skupině `www-data`
- Webová stránka dostupná na portu `80`, ověřena pomocí:
  ```bash
  curl http://localhost
  ```

---
## 12. Kroky provisioning skriptu
- Vytvoření uživatele `webapp` se shellem `/bin/bash`
- Vytvoření složky `/opt/static-sites` s vlastníkem `webapp`
- Instalace a aktivace služby NGINX
- Nasazení vlastní konfigurace NGINX (`sites-available/static-site`)
- Aktivace konfigurace pomocí symlinku do `sites-enabled`
- Generování souboru `index.html` pomocí šablony `index.html.j2`
- Použití proměnných `welcome_message` a `admin_user`
- Nastavení oprávnění pro přístup uživatele `www-data` ke složce `static-sites`
- Validace dostupnosti webu pomocí modulu `uri`
- Ověření výstupu pomocí `curl http://localhost`

---
## 13. Bonus
- Diagnostika chyby 403 a oprava pomocí skupinových práv
- Vlastní konfigurace NGINX mimo výchozí šablonu
- Testování v izolovaném prostředí Vagrant bez ovlivnění Codespace
- Vše dokumentováno v README.md

---
---
# Výsledky a závěr
## 14. Výsledek
Webová stránka se úspěšně zobrazuje na portu `80` s obsahem generovaným pomocí Ansible. Vše je ověřeno lokálně ve Vagrantu.

---
## 15. Poznámky
- Konfigurace pro Codespace zůstává nedotčena
- Lokální inventář slouží pouze pro testování ve Vagrantu
- Vault proměnné se načítají správně díky přiřazení `localhost` do skupiny `web`
- Projekt je izolovaný od Codespace a hlavního GitHub repozitáře
- Vhodné pro testování, výuku nebo demonstraci provisioning procesů

---
## 16. Autor
Projekt vypracovala [Michaela Kučerová](https://github.com/Miska296)  
Verze: 1.0  
Datum: září 2025

---
## 17. Licence
Tento projekt je dostupný pod licencí MIT. Podrobnosti viz soubor [LICENSE](LICENSE).

- Struktura projektu