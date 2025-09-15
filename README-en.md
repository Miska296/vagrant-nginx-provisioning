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
### 5.5 Testing the web server
If the installation of NGINX is part of the provisioning script:
- Check that NGINX is running:
  ```bash
  sudo systemctl status nginx
  ```
- If it is not running, start it:
  ```bash
  sudo systemctl start nginx
  ```
- Test the availability of the website:
> **Note:** In the Codespace environment, `systemd` is not available, so the command `sudo systemctl status nginx` may not work. Instead, you can use:
  ```bash
  curl http://localhost
  ```
If you have port forwarding in your `Vagrantfile`:
  ```ruby
  config.vm.network "forwarded_port", guest: 80, host: 8080
  ```
Then you can test from the host system:
  ```bash
  curl http://localhost:8080
  ```

---
## 6. Technical details
### 6.1 Web service
The role `webserver` performs the following steps:
- Creating the user `webapp` with the shell `/bin/bash`
- Installation and configuration of the NGINX web server
- Generating a static web page using the template `index.html.j2` with variables `welcome_message` and `admin_user`
- Saving the file `index.html` to the folder `/opt/static-sites`, owned by the user `webapp`
- Setting permissions for the `www-data` group access
- Checking the availability of a website using the `uri` module

---
### 6.2 NGINX Configuration
- Configuration using the template `nginx.conf.j2`
- The content of the website is stored in `/opt/static-sites/index.html`
- The owner of the content is `webapp`, access granted to the group `www-data`
- The website available on port `80`, verified using:
  ```bash
  curl http://localhost
  ```

---
### 6.3 Steps of the provisioning script
- Deployment of the custom NGINX configuration (`sites-available/static-site`)
- Activating the configuration using a symlink to `sites-enabled`
- Setting permissions for user `www-data` access to the `static-sites` folder
- Validation of website availability using the `uri` module
> Note: These steps are already described in detail in sections 6.1 and 6.2 above. This section serves as a brief summary of the provisioning process.

---
---
# Results and verification
## 7. Result
The website is successfully displayed on port `80` with content generated using Ansible. Functionality was verified locally in a Vagrant VM.

---
### 7.1 Testing and verification
The test run took place on **September 12, 2025**.  
Results:
- The playbook `playbook.yml` ran without errors (`ok=16`, `changed=12`, `failed=0`)
- The user `webapp` has been successfully created
- The NGINX web server has been installed, configured, and restarted
- The webpage was generated using the template `index.html.j2`
- The output `curl http://localhost` contained the expected HTML content:
  ```html
  <h1>Hello from Ansible-managed NGINX!</h1>
  <p>Server configured automatically by michaela using Ansible</p>
  ```
This confirms the functionality of the provisioning script in an isolated environment.

---
### 7.2 Supplementary notes and tips
- Resolve a 403 error by setting group rights correctly ('www-data')
- Custom NGINX configuration outside of the default template (`nginx.conf.j2`)
- Testing in an isolated Vagrant environment without affecting Codespace
- All steps and results are documented in this `README.md`

---
## 8. Launching the playbook in Codespace
For local testing in Codespace or Vagrant VM, just run:
  ```bash
  ansible-playbook playbook.yml --ask-vault-pass -i inventory/hosts
  ```
This command:
- Loads the inventory from `inventory/hosts`, which contains `ansible_connection=local`
- The Vault will use the password to decrypt the variables
- It runs tasks directly on the local machine without SSH
- It will automatically find the role `webserver` in the `roles/` folder
- Does not require any `provision.sh` script 

After a successful run, the webpage will appear on port `80`. In Codespace, the port can be opened publicly and a URL in the format e.g.: https://upgraded-space-trout-7vxgjp7x7pv53wpg-80.app.github.dev/

Displayed content:
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