# vagrant-nginx-provisioning
# 🛠 Lokální testování Ansible playbooku ve Vagrant VM
Tento návod popisuje postup, jak otestovat Ansible playbooky ve Vagrant virtuálním prostředí bez narušení funkční konfigurace používané v Codespace.

---
## 📦 1. Příprava prostředí
- Vagrant VM
  - Spusť Vagrant VM pomocí `vagrant up`
  - Připoj se do VM: `vagrant ssh`

- Struktura projektu
Ujisti se, že složka `/vagrant` obsahuje:
  - `users-test.yml` — testovací playbook
  - `roles/users/tasks/main.yml` — role pro vytvoření uživatele
  - `group_vars/web/vault` — šifrovaný soubor s heslem
  - `provision.sh` — provisioning skript pro Codespace
  - `inventory/hosts` — vlastní inventář pro testování ve Vagrantu

---
## 🔐 2. Ansible Vault
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
## 📁 3. Inventář pro Vagrant
Vytvoř soubor `/vagrant/inventory/hosts` s obsahem:
  ```ini
  [web]
  localhost
  ```
Tím zajistíš, že proměnné z `group_vars/web` se načtou i pro `localhost`.

---
## 👤 4. Testování vytvoření uživatele
Po úspěšném spuštění playbooku ověř, že uživatel byl vytvořen:
  ```bash
  id webapp
  getent passwd webapp
  ```

---
## 🌐 5. Testování webového serveru
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
  ```Kód
  http://localhost:8080
  ```

---
## ✅ Poznámky
- Konfigurace pro Codespace zůstává nedotčena
- Lokální inventář slouží pouze pro testování ve Vagrantu
- Vault proměnné se načítají správně díky přiřazení `localhost` do skupiny `web`

---
## Webová služba
Webová služba je nasazena pomocí role webserver, která:
- Vytváří uživatele webapp
- Instaluje a konfiguruje NGINX
- Klonuje statický web z GitHubu
- Generuje `index.html` pomocí šablony `index.html.j2` s proměnnými `welcome_message` a `admin_user`
- Ukládá obsah do `/opt/static-sites`, vlastněného uživatelem `webapp`
- Ověřuje dostupnost webu pomocí modulu `uri`

---
## Webová služba
- Webový server NGINX je nakonfigurován pomocí šablony `nginx.conf.j2`
- Obsah webu je uložen v `/opt/static-sites/index.html`
- Soubor je generován pomocí Ansible šablony `index.html.j2` s proměnnými `welcome_message` a `admin_user`
- Vlastníkem obsahu je uživatel `webapp`, přístup umožněn skupině `www-data`
- Webová stránka je dostupná na portu `80` a ověřena pomocí `curl http://localhost`




---
## 🧠 Bonus
- Diagnostika 403 chyb a oprava pomocí skupinových práv
- Vlastní konfigurace NGINX mimo výchozí šablonu
- Testování v izolovaném prostředí Vagrant bez ovlivnění Codespace
- Vše dokumentováno do README.md


# Nasazení webového serveru pomocí Vagrant a Ansible

## Cíl projektu
Provisioning webového serveru s NGINX pomocí Ansible v izolovaném prostředí Vagrant. Webová stránka je generována šablonou a dostupná na portu 80.

## Použité technologie
- Vagrant (virtuální prostředí)
- Ansible (automatizace konfigurace)
- Ubuntu 20.04 (hostovaný OS)
- NGINX (webový server)

## Kroky provisioning skriptu
1. Vytvoření uživatele `webapp` se shellem `/bin/bash`
2. Vytvoření složky `/opt/static-sites` s vlastníkem `webapp`
3. Instalace a aktivace služby NGINX
4. Nasazení vlastní konfigurace NGINX (`sites-available/static-site`)
5. Aktivace konfigurace pomocí symlinku do `sites-enabled`
6. Generování souboru `index.html` pomocí Ansible šablony `index.html.j2`
7. Použití proměnných `welcome_message` a `admin_user` pro dynamický obsah
8. Nastavení oprávnění pro přístup uživatele `www-data` ke složce `static-sites`
9. Validace dostupnosti webu pomocí modulu `uri`
10. Ověření výstupu pomocí `curl http://localhost`

## Výsledek
Webová stránka se úspěšně zobrazuje na portu 80 s obsahem generovaným Ansiblem. Vše je ověřeno lokálně ve Vagrantu.

## Poznámky
- Projekt je izolovaný od Codespace a hlavního GitHub repozitáře
- Vhodné pro testování, výuku nebo demonstraci provisioning procesů

9. Autor
Projekt vypracovala Michaela Kučerová
Verze: 1.0
Datum: červenec 2025
Poslední aktualizace: September 2025
Build: OK

20. Licence
Tento projekt je dostupný pod licencí MIT. Podrobnosti viz soubor LICENSE.
