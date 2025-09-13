# vagrant-nginx-provisioning
# Nasazení webového serveru pomocí Vagrant a Ansible
---
## Lokální testování Ansible playbooku ve Vagrant VM
Tento návod popisuje postup, jak otestovat Ansible playbooky ve Vagrant virtuálním prostředí bez narušení funkční konfigurace používané v Codespace.

---
## 1. Příprava prostředí
Spusť virtuální stroj pomocí Vagrantu:
  ```bash
  vagrant up
  vagrant ssh
  ```

- Struktura projektu
Ujisti se, že složka `/vagrant` obsahuje:
  - `users-test.yml` — testovací playbook
  - `roles/users/tasks/main.yml` — role pro vytvoření uživatele
  - `group_vars/web/vault` — šifrovaný soubor s heslem
  - `provision.sh` — provisioning skript pro Codespace
  - `inventory/hosts` — vlastní inventář pro testování ve Vagrantu

---
## 2. Ansible Vault
Vault soubor se nachází v `group_vars/web/vault` a obsahuje proměnnou:
  ```yaml:
  webapp_password: tajneheslo123
  ```
Soubor je šifrován pomocí:
  ```bash
  ansible-vault create group_vars/web/vault
  ```
Při spouštění playbooku je nutné zadat heslo:
  ```bash
  ansible-playbook users-test.yml --ask-vault-pass -i /vagrant/inventory/hosts
  ```

---
## 3. Inventář pro Vagrant
Vytvoř soubor `/vagrant/inventory/hosts` s obsahem:
  ```ini
  [web]
  localhost
  ```
Tím zajistíš, že proměnné z `group_vars/web` se načtou i pro `localhost`.

---
## 4. Testování vytvoření uživatele
Po úspěšném spuštění playbooku ověř, že uživatel byl vytvořen:
  ```bash
  id webapp
  getent passwd webapp
  ```

---
## 5. Testování webového serveru
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
## 6. Poznámky
- Konfigurace pro Codespace zůstává nedotčena
- Lokální inventář slouží pouze pro testování ve Vagrantu
- Vault proměnné se načítají správně díky přiřazení `localhost` do skupiny `web`

---
## 7. Webová služba
Role webserver provádí:
- Vytvoření uživatele `webapp`
- Instalaci a konfiguraci NGINX
- Klonování statického webu z GitHubu
- Generování `index.html` pomocí šablony `index.html.j2` s proměnnými `welcome_message` a `admin_user`
- Uložení obsahu do `/opt/static-sites`, vlastněného uživatelem `webapp`
- Ověření dostupnosti webu pomocí modulu `uri`

---
## 8. Konfigurace NGINX
- Konfigurace pomocí šablony `nginx.conf.j2`
- Obsah webu uložen v `/opt/static-sites/index.html`
- Vlastníkem obsahu je `webapp`, přístup umožněn skupině `www-data`
- Webová stránka dostupná na portu `80`, ověřena pomocí:
  ```bash
  curl http://localhost
  ```

---
## 9. Bonus
- Diagnostika chyb 403 a oprava pomocí skupinových práv
- Vlastní konfigurace NGINX mimo výchozí šablonu
- Testování v izolovaném prostředí Vagrant bez ovlivnění Codespace
- Vše dokumentováno v README.md

---
## 10. Cíl projektu
Provisioning webového serveru s NGINX pomocí Ansible v izolovaném prostředí Vagrant. Webová stránka je generována šablonou a dostupná na portu `80`.

---
## 11. Použité technologie
- Vagrant (virtuální prostředí)
- Ansible (automatizace konfigurace)
- Ubuntu 20.04 (hostovaný OS)
- NGINX (webový server)

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
## 13. Výsledek
Webová stránka se úspěšně zobrazuje na portu `80` s obsahem generovaným pomocí Ansible. Vše je ověřeno lokálně ve Vagrantu.

---
## 14. Poznámky
- Projekt je izolovaný od Codespace a hlavního GitHub repozitáře
- Vhodné pro testování, výuku nebo demonstraci provisioning procesů

---
## 15. Autor
Projekt vypracovala [Michaela Kučerová](https://github.com/Miska296)
Verze: 1.0
Datum: září 2025

---
## 20. Licence
Tento projekt je dostupný pod licencí MIT. Podrobnosti viz soubor [LICENSE](LICENSE).
