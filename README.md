Kibana role
=========

Роль для установки Kibana на хостах с ОС: Debian, Ubuntu, CentOS, RHEL.

Requirements
------------

Поддерживаются только ОС семейств debian и EL.

Role Variables
--------------

| Variable name | Default | Description |
|-----------------------|----------|-------------------------|
| kibana_version | "7.14.0" | Параметр, который определяет какой версии Kibana будет установлен |

Example Playbook
----------------

    - hosts: all
      roles:
         - { role: kibana_roles }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
- Zakharov Sergey, Netology student of DevOps engeneers course.
- This README.md ver.0.0.02
---
### Описание функционала `ansible-playbook` согласно ДЗ.
* Показана структура файлов роли с описанием задач, модулей, методов, переменных и их значений, при выполнении которых Ansible на управляемых нодах установит утилиты и настроит соответствующие конфигурации.

1. Файл `ansible/playbook/site.yml`
```yml
  - import_tasks: "configure.yml"
  - include_tasks: "download_kibana_rpm.yml"
  - include_tasks: "install_kibana.yml"          
```
2. Файл ` kibana_roles/tasks/download_kibana_rpm.yml`
```yml
- name: "Download Kibana's rpm"
  get_url:
    url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"
    dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
  register: download_kibana
  until: download_kibana is succeeded
```
3. Файл ` kibana_roles/tasks/install_kibana.yml`
```yml
- name: Install Kibana
  become: true
  yum:
    name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
    state: present
  notify: restart kibana
```

4. Файл ` kibana_roles/tasks/configure_kibana.yml`
```yml
- name: Configure Kibana
    become: true
    template:
      src: kibana.yml.j2
      mode: 0644
      dest: /etc/kibana/kibana.yml
    notify: restart kibana
```
5. Файл ` kibana_roles/defaults/main.yml `
```yml
elk_stack_version: "7.14.0"
```
6. Файл `  kibana_roles/handlers/main.yml `
```yml
- name: restart kibana
  become: true
  service:
    name: kibana
    state: restarted
```
7. Файл ` kibana_roles/templates/kibana.yml.j2 `
```yml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://{{ hostvars['el-instance']['ansible_facts']['default_ipv4']['address'] }}:9200"]
kibana.index: ".kibana"
```
8. Файл ` kibana_roles/vars/main.yml `
```yml
kibana_version: "7.14.0"
```
