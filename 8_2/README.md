# Домашнее задание к занятию "08.02 Работа с Playbook"

## Задача 1. Приготовьте свой собственный inventory файл prod.yml.
```
student@student-virtual-machine:~/AH/8_2/playbook_vector$ cat inventory/prod.yml 
---
vector:
  hosts:
    vector-test:
      ansible_connection: docker
```
## Задача 2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector.
## Задача 3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.
## Задача 4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.
```
Вообще, vector имеет rpm и deb пакеты, что существенно упрощает установку с помощью ansible.
Но так как, есть задача 3, то пользуем архив для "неизвестного linux".
```
```
student@student-virtual-machine:~/AH/8_2/playbook_vector$ cat site.yml 
---
- name: Install Vector
  hosts: vector
  tasks:
    - name: Get Vector distrib
      ansible.builtin.get_url:
        url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{
            vector_version }}-x86_64-unknown-linux-gnu.tar.gz"
        dest: "./vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz"
        mode: 0755
        timeout: 90
        force: true
	  tags: download
    - name: Create directory for Vector
      ansible.builtin.file:
        state: directory
        path: "{{ vector_dir }}"
        mode: 0755
      tags: CD
    - name: Extract Vector
      ansible.builtin.unarchive:
        copy: false
        src: "/vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz"
        dest: "{{ vector_dir }}"
        extra_opts: [--strip-components=2]
        creates: "{{ vector_dir }}/bin/vector"
      tags: extract
    - name: environment Vector
      ansible.builtin.template:
        src: templates/vector.sh.j2
        dest: /etc/profile.d/vector.sh
        mode: 0755
      tags: env


```
```		
student@student-virtual-machine:~/AH/8_2/playbook_vector$ cat templates/vector.sh.j2 
#!/usr/bin/env bash
export VECTOR_DIR={{ vector_dir }}
export PATH=$PATH:$VECTOR_DIR/bin
```
```
student@student-virtual-machine:~/AH/8_2/playbook_vector$ cat group_vars/vector/vars.yml 
---
vector_version: "0.21.0"
vector_dir: "/etc/vector"

```
## Задача 5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.
```
Было много синтаксических ошибок и опечаток. Все исправил.

student@student-virtual-machine:~/AH/8_2/playbook_vector$ ansible-lint site.yml 
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml

```
## Задача 6. Попробуйте запустить playbook на этом окружении с флагом --check.
```
student@student-virtual-machine:~/AH/8_2/playbook$ ansible-playbook v.yml -i inventory/prod.yml --check

PLAY [Install Vector] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get Vector distrib] **********************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Create directory for Vector] *************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Extract Vector] **************************************************************************************************************************************************
skipping: [clickhouse-01]

TASK [environment Vector] **********************************************************************************************************************************************
changed: [clickhouse-01]

PLAY RECAP *************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
## Задача 7. Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.
```
student@student-virtual-machine:~/AH/8_2/playbook$ ansible-playbook v.yml -i inventory/prod.yml --diff

PLAY [Install Vector] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get Vector distrib] **********************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Create directory for Vector] *************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Extract Vector] **************************************************************************************************************************************************
skipping: [clickhouse-01]

TASK [environment Vector] **********************************************************************************************************************************************
--- before
+++ after: /home/student/.ansible/tmp/ansible-local-1335193ql8yo3u9/tmp1h6gu25b/vector.sh.j2
@@ -0,0 +1,4 @@
+#!/usr/bin/env bash
+export VECTOR_DIR=/etc/vector
+export PATH=$PATH:$VECTOR_DIR/bin

changed: [clickhouse-01]

PLAY RECAP *************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
## Задача 8. Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.