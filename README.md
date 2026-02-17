# 4640-lab-wk7

Ansible configuration to set up Nginx on EC2 instances.

## required command(s) to:

create new keys
```bash
ssh-keygen -t ed25519 -f ~/.ssh/aws
```
import public key
```bash
./scripts/import_lab_key ~/.ssh/aws.pub
```
delete key from AWS
```bash
./scripts/delete_lab_key
```
cd to the terraform directory, and run:
```bash
terraform init
terraform validate
terraform apply
```
cd back to ansible and run:
```bash
ansible-playbook --syntax-check playbook.yml
ansible-playbook -i inventory/hosts.yml playbook.yml
```

hosts.yml
```bash
all:
  children:
    web:
      hosts:
        server-one:
          ansible_host: 34.220.157.187
          ansible_user: admin
          ansible_ssh_private_key_file: ~/.ssh/aws
        server-two:
          ansible_host: 52.12.203.232
          ansible_user: admin
          ansible_ssh_private_key_file: ~/.ssh/aws
```
this defines servers' information and ip addresses to manage.

ansible_host: public IP addresses of the EC2 instances.

ansible_user: set to admin for the Debian-based AMI.

ansible_ssh_private_key_file: path to the generated SSH private key for authentication.

playbook.yml
```bash
playbook.yml
- name: Configure web servers
  hosts: all
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
        update_cache: true

    - name: create directory structure for web documents
      ansible.builtin.file:
        path: /web/html
        state: directory
        mode: '0755'

    - name: copy nginx conf file to server
      ansible.builtin.copy:
        src: files/nginx.conf
        dest: /etc/nginx/sites-available/default

    - name: create link to nginx config file to enable it
      ansible.builtin.file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link

    - name: Generate index.html file from template
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: /web/html/index.html

    - name: reload nginx service
      ansible.builtin.service:
        name: nginx
        state: reloaded
        enabled: true
```
playbook defines what to do in the server step by step.

ansible.builtin.package: installs Nginx and updates the package cache.

ansible.builtin.file: creates the /web/html directory and manages the symlink

ansible.builtin.copy: deploys the custom nginx.conf to the server

ansible.builtin.template: generates the index.html file

ansible.builtin.service: reloads and enables the Nginx service

<img width="618" height="536" alt="image" src="https://github.com/user-attachments/assets/a4eb7a0e-6481-4376-a173-f8420e97e24f" />
