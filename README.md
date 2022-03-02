### Описание функционала `Kibana role` согласно ДЗ.
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

### Далее показана структура файлов роли с описанием задач, модулей, методов, переменных и их значений, при выполнении которых Ansible на управляемых нодах установит утилиты и настроит соответствующие конфигурации.

1. Файл `ansible/playbook/site.yml` для задания порядка выполнения задач роли.
```yml
  - include_tasks: "download_kibana_rpm.yml"
  - include_tasks: "install_kibana.yml"      
  - import_tasks: "configure.yml"
```
* `- include_tasks: "download_kibana_rpm.yml"` Загружаем пакет с Кибаной
* `- include_tasks: "install_kibana.yml"` Инсталляция Кибаны
* `- import_tasks: "configure.yml"` Конфигурирование Кибаны

2. Файл ` kibana_roles/tasks/download_kibana_rpm.yml`
```yml
- name: "Download Kibana's rpm"
  get_url:
    url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"
    dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
  register: download_kibana
  until: download_kibana is succeeded
```
* `- name: "Download Kibana's rpm"`  Название задачи для понимания ее назначения
* `get_url:`    Ипользовать модуль `get_url`
* ` url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"`    Адрес ссылки для загрузки `Kibana`. Переменая `{{ elk_stack_version }}` указывает на то, какую версию нужно загружать.
* `dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"`  Директория на `manage_node` для сохранения загруженного пакета 
* `register: download_kibana`      В переменную `download_kibana` записываем результат выполнения загрузки.
* `until: download_kibana is succeeded`    Цикл для выполнения условий успешной загрузки пакета.

3. Файл ` kibana_roles/tasks/install_kibana.yml`
```yml
- name: Install Kibana
  become: true
  yum:
    name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
    state: present
  notify: restart kibana
```
* `- name: Install Kibana`      Имя задачи.
* `become: true`      Выполнять с повышением привелегий (под УЗ root по дефолту)
* `yum:`    Запустить модуль `yum` для установки в систему утилиты из пакета RPM
* `name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"`   Место расположения пакета.
* `state: present`    Условие гарантии того, что данный пакет будет установлен.
* `notify: restart kibana`   Выполнить условие - с помощью хендлера с именем `restart kibana`  перезапустить сервис  `Kibana` на `manage_node`

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
* `- name: Configure Kibana`   Имя задачи.
* `become: true`      Выполнять с повышением привелегий (под УЗ root по дефолту)
* `template:`     Обращение к модулю `template` для создания на `manage_node` из файлов с шаблоном конфигурации .j2 файлов конфигураци .yml. Модуль ищет шаблоны  в директории `/template` на `control_node`
* `src: kibana.yml.j2`   Указан файл - источник шаблона
* `dest: /etc/kibana/kibana.yml`    Указано место назначения файла конфигурации.
* `notify: restart kibana`   Выполнить условие - с помощью хендлера с именем `restart kibana`  перезапустить сервис  `Kibana` на `manage_node`

5. Файл ` kibana_roles/defaults/main.yml `
```yml
elk_stack_version: "7.14.0"
```
* `elk_stack_version` - имя переменной, используемой для определения номера версии Кибаны. Совпадает с версией Elasticsearch
*  `"7.14.0"`- значение переменной
6. Файл `  kibana_roles/handlers/main.yml `
```yml
- name: restart kibana
  become: true
  service:
    name: kibana
    state: restarted
```
* `- name: restart kibana`   Имя первого хендлера 
* `become: true`      Выполнять с повышением привелегий (под УЗ root по дефолту)
* `service:`      Указание на то, что нужно обратиться к модулю `ansible.builtin.service`
* `name: kibana`   Имя сервиса
* `state: restarted`      Выполнить команду рестарта

7. Файл ` kibana_roles/templates/kibana.yml.j2 `
```yml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://{{ hostvars['el-instance']['ansible_facts']['default_ipv4']['address'] }}:9200"]
kibana.index: ".kibana"
```
* `server.host: "0.0.0.0"` - дляхоста с любым IP адресом
* `elasticsearch.hosts` - переменная, значение которой сформируется с помощью ключей и значений, взятых из других переменных
* `kibana.index: ".kibana"` - переменная
8. Файл ` kibana_roles/vars/main.yml `
```yml
kibana_version: "7.14.0"
```
* имя переменной и её значение для определения версии Кибаны
