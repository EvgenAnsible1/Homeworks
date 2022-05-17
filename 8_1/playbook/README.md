# Домашнее задание к занятию "08.01 Введение в Ansible"

## Задача 0.
Подготовка к выполнению
Установите ansible версии 2.10 или выше.
```
ansible [core 2.12.5]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/student/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/student/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
  jinja version = 2.10.1
  libyaml = True
```
Создайте свой собственный публичный репозиторий на github с произвольным именем.
```
https://github.com/EvgenAnsible1/Homeworks
```
Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.
```
https://github.com/EvgenAnsible1/Homeworks/tree/master/8_1/playbook
```
## Задача 1.
Попробуйте запустить playbook на окружении из test.yml, зафиксируйте какое значение имеет факт some_fact для указанного хоста при выполнении playbook'a.
```
student@student-virtual-machine:~/AH/8_1/playbook$ ansible-playbook site.yml -i inventory/test.yml 

PLAY [Print os facts] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] ********************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
## Задача 2.
Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.
```
student@student-virtual-machine:~/AH/8_1/playbook$ nano group_vars/all/examp.yml 
-----------------------------------------------------------------------------------------------
---
  some_fact: 'all default fact'
-----------------------------------------------------------------------------------------------
student@student-virtual-machine:~/AH/8_1/playbook$ ansible-playbook site.yml -i inventory/test.yml 

PLAY [Print os facts] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] ********************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
## Задача 3.
Воспользуйтесь подготовленным (используется docker) или создайте собственное окружение для проведения дальнейших испытаний.
```
student@student-virtual-machine:~/AH/8_1/playbook$ docker ps
CONTAINER ID   IMAGE      COMMAND       CREATED          STATUS         PORTS     NAMES
cb6dacbab219   centos:7   "/bin/bash"   2 minutes ago    Up 6 seconds             centos7
28706ea571c5   ubuntu     "/bin/bash"   22 minutes ago   Up 3 minutes             ubuntu
```
## Задача 4.
Проведите запуск playbook на окружении из prod.yml. Зафиксируйте полученные значения some_fact для каждого из managed host.
```
tudent@student-virtual-machine:~/AH/8_1/playbook$ sudo ansible-playbook site.yml -i inventory/prod.yml 

PLAY [Print os facts] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ********************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
## Задача 5.
Добавьте факты в group_vars каждой из групп хостов так, чтобы для some_fact получились следующие значения: для deb - 'deb default fact', для el - 'el default fact'.
```
student@student-virtual-machine:~/AH/8_1/playbook$ nano group_vars/el/examp.yml 
----------------------------------------------------------------------------------------------------
---
  some_fact: 'el default fact'

----------------------------------------------------------------------------------------------------
student@student-virtual-machine:~/AH/8_1/playbook$ nano group_vars/deb/examp.yml 
----------------------------------------------------------------------------------------------------
---
  some_fact: 'deb default fact'

----------------------------------------------------------------------------------------------------
```
## Задача 6.
Повторите запуск playbook на окружении prod.yml. Убедитесь, что выдаются корректные значения для всех хостов.
```
student@student-virtual-machine:~/AH/8_1/playbook$ sudo ansible-playbook site.yml -i inventory/prod.yml 

PLAY [Print os facts] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ********************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
## Задача 7.
При помощи ansible-vault зашифруйте факты в group_vars/deb и group_vars/el с паролем netology.
```
student@student-virtual-machine:~/AH/8_1/playbook$ ansible-vault encrypt group_vars/deb/examp.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
student@student-virtual-machine:~/AH/8_1/playbook$ ansible-vault encrypt group_vars/el/examp.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
student@student-virtual-machine:~/AH/8_1/playbook$ 
```
## Задача 8.
Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь в работоспособности.
```
student@student-virtual-machine:~/AH/8_1/playbook$ ansible-playbook --ask-vault-pass  site.yml -i inventory/prod.yml 
Vault password: 

PLAY [Print os facts] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ********************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
## Задача 9.
Посмотрите при помощи ansible-doc список плагинов для подключения. Выберите подходящий для работы на control node.
```
Не нашел ничего подходящего, возможно не знаю, что искать =)
Согласно документации (https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html) любой ПК с установленным ansible подходит для работы как control node.
```
## Задача 10.
В prod.yml добавьте новую группу хостов с именем local, в ней разместите localhost с необходимым типом подключения.
```
student@student-virtual-machine:~/AH/8_1/playbook$ nano inventory/prod.yml 
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local
```
## Задача 11.
Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь что факты some_fact для каждого из хостов определены из верных group_vars.
```
tudent@student-virtual-machine:~/AH/8_1/playbook$ ansible-playbook --ask-vault-pass  site.yml -i inventory/prod.yml 
Vault password: 

PLAY [Print os facts] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ********************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
## Задача 12.
Заполните README.md ответами на вопросы. Сделайте git push в ветку master. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым playbook и заполненным README.md.
```
