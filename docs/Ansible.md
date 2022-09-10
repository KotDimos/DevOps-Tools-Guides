# Ansible

# Содержание

* [Основы](#основы)
> * [Установка](#установка)
> * [Терминология](#терминология)
> * [Структура файлов](#структура-файлов)
> * [Hosts](#hosts)
> * [Переменные](#переменные)
> * [Модули](#модули)
> * [Условия](#условия)
> * [Циклы](#циклы)

* [Роли](#роли)
> * [Создание роли](#создание-роли)
> * [Директории в роли](#директории-в-роли)

* [Полезные ссылки](#полезные-ссылки)


# Основы

## Установка

Установить Ansible можно 2 способами:

* Через пакетный менеджер Linux.
* Через менеджер python - pip.

Полная
[документация по установке](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).


## Терминология

* *Inventory* — инвентарный файл. В этом файле описываются хосты,
группы хостов, а также могут быть созданы переменные.
* *Task* - минимальный элемент описания конфигурации в котором описано состояние объекта.
* *Role* - логически завершенный и обосновано выделенный набор tasks,
который в дальнейшем используется для применения повторяющейся конфигурации в разных playbooks.
* *Playbook* - файл с полным описанием конфигурации для конкретных хоста/группы,
включает в себя tasks и roles.
* *Vars* - переменные, используемые для вариативности параметров конфигурации
в зависимости от хоста/группы/роли/etc.
* *Module* — модуль Ansible. Реализует определенные функции.

Список всех терминов в [документации](http://docs.ansible.com/ansible/devel/glossary.html).


## Структура файлов

* ansible.cfg - первоначальная настройка для ansible.
* playbook-name.yml - ansible-playbook, в котором описаны задачи.
* requirements.yml - файл с перечнем ansible-ролей, отдельно вынесенных задач.
* inventorys - директория для инвентарей ansible.
* hosts - файл инвентаризации для рабочих серверов.

Конфигурационный файл `ansible.cfg` может находиться в таких местах,
и в таком приоритете происходит поиск файла.
* ANSIBLE_CONFIG - переменная среды, если установлена.
* ./ansible.cfg – в текущем каталоге.
* ~/.ansible.cfg — в домашнем каталоге.
* /etc/ansible/ansible.cfg — в стандартной директории.

[Полная документация по ansible.cfg](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)

[Один из вариантов](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html#alternative-directory-layout)
правильной структуры директории.

    inventories/
       production/
          hosts                 # файл инвентаризации для рабочих серверов
          group_vars/
             group1.yml         # назначение переменных определенным группам
             group2.yml
          host_vars/
             hostname1.yml      # назначение переменных определенным системам
             hostname2.yml

       development/
          hosts
          group_vars/
             group1.yml
             group2.yml
          host_vars/
             hostname1.yml
             hostname2.yml

    library/                    # если есть пользовательские модули (опционально)
    module_utils/               # если какие-либо пользовательские утилиты для поддержки модулей (опционально)
    filter_plugins/             # если какие-либо настраиваемые плагины фильтров (опционально)

    site.yml                    # playbooks
    webservers.yml
    dbservers.yml

    roles/
        role1/
        role2/
        role3/
        role4/


## Hosts

`hosts` - хост или перечень хостов, с которыми будет происходить работа.

`[ ]` - В скобки указывается название группы серверов.

    # Пример
    [dev]

ansible_host - ip адрес.

    # Пример
    ansible_host=0.0.0.0

ansible_user - имя пользователя для подключения на удаленный сервер.

    # Пример
    ansible_user=user

ansible_pass - Пароль пользователя.

    # Пример
    ansible_pass=your_password

ansible_sudo_pass - Пароль для подключения sudo пользователя.

    # Пример
    ansible_sudo_pass=your_password

ansible_ssh_private_key_file - путь до закрытого ключа. Для входа без участия пароля.

    # Пример
    ansible_ssh_private_key_file=/home/user/.ssh/id_rsa

ansible_port - порт подключения ssh.

    # Пример
    ansible_port=1234

Данные ключи основные для работы. Все ключи можно посмотреть в
[документации](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters).


## Переменные

В системе управления конфигурациями Ansible переменные хранят значения,
которые могут использоваться в наборах инструкций (playbook).

Для инициализации или переопределения в плэйбуке переменных, используется `vars`.

    vars:
      <variable>: <value>

Для использования переменных нужно заключать их в двойные фигурные скобки - `{{ }}`,
и обязательно внутри скобочек переменная должна отделяться пробелами.
Так же хорошей практикой считается, что заключать где используется переменная в двойные кавычки.

    # Пример
    file:
      path: "{{ path_to_file }}"

    file:
      path: "~/directory/{{ file_name }}.txt"

Переменные можно задавать в отдельном файле.

    <variable>: <value>

И для того чтобы подключить этот файл к плейбуку используется `vars_files`.

    vars_files:
      - <paht-to-file>

Переменные можно определять как лист.

    list:
     - var_1
     - var_2
     - var_3

Применение.

    "{{ list[0] }}"

Переменные можно определять как словарь.

    <dictionary>:
      <variable_1>: <value_1>
      <variable_2>: <value_2>

Применение.

    dictionary['variable_1']
    dictionary.variable_1

Переменные можно переопределять, и у них есть свои приоритеты.
[Полный приоритет переменных](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#ansible-variable-precedence).


## Модули
Модули отвечают за действия, которые выполняет Ansible.
При этом каждый модуль, как правило, отвечает за свою конкретную и небольшую задачу.

Пример на модуле [file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html).
Модуль file нужен для создания/удаления файла.

    - name: create file
      file:
        path: /home/user/file.txt
        owner: user
        group: user
        mode: '0777'
        state: touch

* `name` - описание модуля.
* `file` - имя модуля, в версиях выше 2.9 он называется ansible.builtin.file.
* `path` - путь к файлу.
* `owner` - владелец файла.
* `group` - группа файла.
* `mode` - права файла.
* `state` - что требуется сделать. touch - создать файл.

Каждый модуль имеет свои параметры и не имеет смысла расписывать их, и все модули можно найти на
[официальном сайте Ansible](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html).

Есть специальные модули
[shell](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html) и
[command](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html),
их нужно использовать только в самый последний момент, когда нет вообще такого модуля.
Как пример, когда вышла новое приложение, и под неё ещё не написали модули.

`Shell` -  запуск /bin/sh.

`Command` - команды не будут обрабатываться через оболочку, поэтому такие переменные, как $HOME,
и такие операции, как <, >, |, ; и & не будут работать.

Почему лучше использовать модули, а не команды shell.
При исполнении сценария, asnible проверяет сделано ли уже это
(скопирован ли файл, установлена ли программа и пр.),
И если уже это сделано, то он ничего не делает.

Есть специальный модуль для отладки плэйбука -
[debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html).


## Условия

Для проверки условия используется специальный параметр `when`.

В условиях в Ansible для сравнения используются:
* `==` - равно.
* `!=` - не равно.
* `>` больше.
* `<` меньше.
* `>=` больше равно.
* `<=` меньше равно.

Можно указать несколько условий с помощью операторов `and` и `or`.

Для проверки вхождения символа или подстроки в строку используются операторы `in` и `not`.


Пример:

    # Проверка семейства ОС
    - name: Check OS family
      debug: msg="This is Debian"
      when: ansible_os_family == "Debian"

Так же можно сохранить результат выполнения модуля в переменную.
Для этого используется параметр `register`.

Пример:

    - name: Check if admin logged
      command: who
      register: who_check

    - name: Print message if user admin not logged
      debug: msg="User admin is not logged on remote host"
      when: not 'admin' in who_check.stdout


## Циклы

Для циклов используется специальный параметр `loop`.

В данном примере просто будет выведено сообщение с номерами массива prime.
В параметр `loop`, просто передаётся массив, который требуется обойти,
а дальше с помощью переменной `item` вызывается по одному элементу.

    vars:
      prime: [2, 3, 5, 7, 11]
    tasks:
      - name: Show first five prime numbers
        debug:
          msg: "{{ item }}"
        loop: "{{ prime }}"

Как один из реальных примеров применения циклов.
Создание пользователей из списка.

    - name: add users
      ansible.builtin.user:
        name: "{{ item.username }}"
        password: "{{ item.pass }}"
      loop:
        - { username: user1, pass: password1 }
        - { username: user2, pass: password2 }


# Роли

## Создание роли

Для создание роли используется ansible galaxy.
И после этой команды будут созданы все стандартные директории для роли.

    ansible-galaxy init <role-name>

## Директории в роли

* `defaults` - содержится перечень всех переменных которые роль будет использовать,
эти переменные могут перезаданы уровнем выше - плейбуком.
* `files` - хранения файлов, которые могут быть скопированы, конкретно для роли.
* `handlers` - специльные обработчки событий, для совершение действий.
* `meta` - информация о роли, создатель, компания, для каких платформ, зависимости, теги.
* `tasks` - директория, где хранятся все таски.
* `templates` - директория, где хранятся шаблоны.
* `tests` - директория для тестов.
* `vars` - перечень переменных, но их нельзя переопределить выше.


# Полезные ссылки
[Специальные переменные от ansible](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)
