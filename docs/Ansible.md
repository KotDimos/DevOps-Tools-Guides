# Ansible

# Содержание

* [Начало работы](#начало-работы)
    * [Установка](#установка)

* [Словарь](#словарь)

* [Структура файлов](#структура-файлов)
    * [ansible.cfg](#ansible.cfg)
    * [Древовидная структура файлов](#древовидная-структура-файлов)

* [Инвентарный файл (hosts)](#инвентарный-файл)
    * [ini](#ini)
    * [yml](#yml)

* [Playbook](#playbook)
    * [Переменные (vars)](#переменные)
    * [Модули](#модули)
    * [Условия (when)](#условия)
    * [Циклы (loop)](#циклы)

* [Роли](#роли)
    * [Создание роли](#создание-роли)
    * [Директории в роли](#директории-в-роли)
    * [Requirements](#requirements)

* [Шаблоны](#шаблоны)
    * [Параметры в файлах](#параметры-в-файлах)
    * [Условие](#условие)
    * [Цикл](#цикл)
    * [Фильтры](#фильтры)

* [Тестирование](#тестирование)
    * [Установка](#установка)
    * [Иницилизация роли](#иницилизация-роли)
    * [Запуск тестирования](#запуск-тестирования)

* [Полезные ссылки](#полезные-ссылки)


# Начало работы

## Установка

[Наверх](#содержание)

Установить Ansible можно 2 способами:

* Через пакетный менеджер Linux.
* Через менеджер python - pip.

Полная
[документация по установке для Linux](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).


# Словарь

[Наверх](#содержание)

* `Hosts` - инвентарный файл, где описываются хосты, группы хостов,
а так же хранятся переменные для этих хостов.
* `Inventories` - директория, где хранятся инвентарные файлы,
а также переменные для групп и хостов.
* `Task` - минимальный элемент описания конфигурации в котором описано состояние объекта.
* `Role` - логически завершенный и обосновано выделенный набор tasks,
который в дальнейшем используется для применения повторяющейся конфигурации в разных playbooks.
* `Playbook` - файл с полным описанием конфигурации для конкретных хоста/группы,
включает в себя tasks и roles.
* `Vars` - переменные, используемые для вариативности параметров конфигурации
в зависимости от хоста/группы/роли/etc.
* `Module` - многоразовый автономный сценарий, который возможно конфигурировать от своих задач.
* `Requirements` - файл с перечнем Ansible-ролей.

Список всех терминов в [документации](http://docs.ansible.com/ansible/devel/glossary.html).


# Структура файлов

[Наверх](#содержание)

* `ansible.cfg` - первоначальная настройка для ansible.
* `playbook-name.yml` - ansible-playbook, в котором описаны задачи.
* `requirements.yml` - файл с перечнем ansible-ролей, отдельно вынесенных задач.
* `inventories` - директория для инвентарей ansible.
* `hosts` - файл инвентаризации для рабочих серверов.


## ansible.cfg

[Наверх](#содержание)

Конфигурационный файл `ansible.cfg` может находиться в таких местах,
и в таком приоритете происходит поиск файла.

* `ANSIBLE_CONFIG` - переменная среды, если установлена.
* `./ansible.cfg` – в текущей директории.
* `~/.ansible.cfg` — в домашней директории пользователя.
* `/etc/ansible/ansible.cfg` — в стандартной директории ansible.

[Полная документация по ansible.cfg](https://docs.ansible.com/ansible/latest/reference_appendices/config.html).

[Полный список параметров](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#common-options).


## Древовидная структура файлов

[Наверх](#содержание)

[Один из вариантов](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html#alternative-directory-layout)
правильной структуры директории.

```
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
```


# Инвентарный файл

[Наверх](#содержание)

Инвентарный файл - хост или перечень хостов, с которыми будет происходить работа.

Инвентарный файл можно описать в формате ini или yml.

## ini

[Наверх](#содержание)

`[]` - В скобки указывается название группы серверов.

    [web]
    web1
    web2

    [db]
    db1
    db2

Частоиспользуемые ключи:

* `ansible_host` - ip адрес.
* `ansible_user` - имя пользователя для подключения на удаленный сервер.
* `ansible_sudo_pass` - Пароль для подключения sudo пользователя.
* `ansible_pass` - Пароль пользователя.
* `ansible_ssh_private_key_file` - путь до закрытого ключа. Для входа без участия пароля.
* `ansible_port` - порт подключения ssh.

*Пример:*

    [example]
    example1
    ansible_host=0.0.0.0
    ansible_user=user
    ansible_ssh_private_key_file=/home/user/.ssh/id_rsa
    ansible_port=2222

Все ключи можно посмотреть в
[документации](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters).


## yml

[Наверх](#содержание)

`hosts` - указываются точки доступа, которые не имеют какой-то группы.

`children` - указываются названия группы серверов и внутри идёт имя группы.
И дальше идёт перечисление названий серверов, и как подключаться.

Пример yml файла.

``` yaml
---
all:
  hosts:
    dev:
  children:
    web:
      hosts:
        web1:
        web2:
    db:
      hosts:
        db1:
        db2:
```

Также можно указывать нужные прараметры как и в ini формате.

*Пример:*

``` yaml
---
all:
  children:
    example:
      example1:
        ansible_host: 0.0.0.0
        ansible_user: user
        ansible_ssh_private_key_file: /home/user/.ssh/id_rsa
        ansible_port: 2222
      example2:
        ansible_host: 0.0.0.0
        ansible_user: user
        ansible_ssh_private_key_file: /home/user/.ssh/id_rsa
        ansible_port: 2222
```


# Playbook

## Переменные

[Наверх](#содержание)

Переменные предназначены для хранения значений, которые могут использоваться в playbook.

Для инициализации или переопределения в плэйбуке переменных, используется `vars`.

``` yaml
vars:
  <variable>: <value>
```

Для использования переменных нужно заключать их в двойные фигурные скобки - `{{ }}`,
и обязательно внутри скобочек переменная должна отделяться пробелами.

Так же хорошей практикой считается,
заключать всю переменную в двойные кавычки, где используется подстановка.

*Примеры:*

``` yaml
file:
  path: "{{ path_to_file }}"

file:
  path: "~/directory/{{ file_name }}.txt"
```

Также переменные можно задавать в отдельном файле.

``` yaml
<variable>: <value>
```

И после этого подключить этот файл к плейбуку. Для этого используется `vars_files`.

``` yaml
vars_files:
  - <paht-to-file>
```

Переменные можно определять как лист.

``` yaml
list:
  - var_1
  - var_2
  - var_3
```

Применение.

``` yaml
"{{ list[0] }}"
```

Переменные можно определять как словарь.

``` yaml
<dictionary>:
  <variable_1>: <value_1>
  <variable_2>: <value_2>
```

Применение.

``` yaml
dictionary['variable_1']
dictionary.variable_1
```

Переменные можно переопределять, и у них есть свои приоритеты.

[Полный приоритет переменных](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#ansible-variable-precedence).


## Модули

[Наверх](#содержание)

Модули отвечают за действия, которые выполняет Ansible.
При этом каждый модуль, как правило, отвечает за свою конкретную и небольшую задачу.

Пример на модуле [file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html).
Модуль file нужен для создания/удаления файла.

``` yaml
- name: create file
  file:
    path: /home/user/file.txt
    owner: user
    group: user
    mode: '0777'
    state: touch
```

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

`shell` - запуск /bin/sh.

`command` - команды не будут обрабатываться через оболочку, поэтому такие переменные, как $HOME,
и такие операции, как <, >, |, ; и & не будут работать.

Почему лучше использовать модули, а не команды shell.
При исполнении сценария, asnible проверяет сделано ли уже это
(скопирован ли файл, установлена ли программа и пр.),
И если уже это сделано, то он ничего не делает.

Есть специальный модуль для отладки плэйбука -
[debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html).


## Условия

[Наверх](#содержание)

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

*Примеры:*

Проверка семейства ОС.

``` yaml
- name: Check OS family
  debug: msg="This is Debian"
  when: ansible_os_family == "Debian"
```

Так же можно сохранить результат выполнения модуля в переменную.
Для этого используется параметр `register`.


``` yaml
- name: Check if admin logged
  command: who
  register: who_check

- name: Print message if user admin not logged
  debug: msg="User admin is not logged on remote host"
  when: not 'admin' in who_check.stdout
```


## Циклы

[Наверх](#содержание)

Для циклов используется специальный параметр `loop`.

В данном примере просто будет выведено сообщение с номерами массива prime.
В параметр `loop`, просто передаётся массив, который требуется обойти,
а дальше с помощью переменной `item` вызывается по одному элементу.

``` yaml
vars:
  prime: [2, 3, 5, 7, 11]
tasks:
  - name: Show first five prime numbers
    debug:
      msg: "{{ item }}"
    loop: "{{ prime }}"
```

Как один из реальных примеров применения циклов.
Создание пользователей из списка.

``` yaml
- name: add users
  ansible.builtin.user:
    name: "{{ item.username }}"
    password: "{{ item.pass }}"
  loop:
    - { username: user1, pass: password1 }
    - { username: user2, pass: password2 }
```


# Роли

## Создание роли

[Наверх](#содержание)

Для создание роли используется ansible galaxy.
И после этой команды будут созданы все стандартные директории для роли.

    ansible-galaxy init <role-name>


## Директории в роли

[Наверх](#содержание)

* `defaults` - содержится перечень всех переменных которые роль будет использовать,
эти переменные могут перезаданы уровнем выше - плейбуком.
* `files` - хранения файлов, которые могут быть скопированы, конкретно для роли.
* `handlers` - специльные обработчки событий, для совершение действий.
* `meta` - информация о роли, создатель, компания, для каких платформ, зависимости, теги.
* `tasks` - директория, где хранятся все таски.
* `templates` - директория, где хранятся шаблоны.
* `tests` - директория для тестов.
* `vars` - перечень переменных, но их нельзя переопределить выше.


## Requirements

[Наверх](#содержание)

Для того чтобы вести зависимости проекта от требуемых ролей
используется файл `requirements.yml`.

Параметры в файле:

* `src` - источник роли, указывается URL-адрес репозитория.
* `scm` - если src является URL-адресом, нужно указать scm.
Поддерживаются только git или hg. По умолчанию git.
* `version` - версия роли для загрузки.
Используется тег, хэш коммита или имя ветки. По умолчанию master.
* `name` - переопределение имени роли. По умолчанию используется имя репозитория.

*Примеры:*

Использование https.

    - src: https://github.com/bennojoy/nginx
      scm: git
      version: master
      name: nginx_role

Использование ssh.

    - src: git+ssh://git@github.com/bennojoy/nginx
      version: master
      name: nginx_role

При установке ролей по стандарту используется директория `~/.ansible/roles`.

Установка из requirements.

    ansible-galaxy install -r requirements.yml

Если требуется изменить путь до директории, то используется флаг `--roles-path`.

    ansible-galaxy install --roles-path "<path>" -r requirements.yml

*Пример:*

    ansible-galaxy install --roles-path ./roles -r requirements.yml


# Шаблоны

[Наверх](#содержание)

`Шаблоны` - это простые текстовые файлы, содержащие параметры конфигурации.
Во время выполнения плейбуков, переменные будут заменены соответствующими значениями.

Кроме замены параметров также есть условные операторы, циклы,
фильтры для преобразования данных, выполнение арифметических вычислений и т.д.


## Параметры в файлах

[Наверх](#содержание)

Файл с шаблоном имеет расширение `.j2` - механизм создания шаблонов Jinja2.

Если шаблоны используется в роли,
тогда создаётся директория `templates`, где и хранятся все шаблоны для нужной роли.

*Пример:*

Пример небольшого шаблона, для подстановки значений
(все переменные должны быть объявлены).

Файл `nginx.conf.j2`.

```
server {
    listen {{ nginx_http_port }}
    server_name {{ server_name }};
    location / {
        proxy_pass http://{{ ip_forward }}:{{ port_forward }};
    }
}
```

Копирование шаблона.

```
- name: Create nginx ssl config
  ansible.builtin.template:
    src: "nginx.conf.j2"
    dest: "/etc/nginx/conf.d/defaults.conf"
    mode: "0644"
```


## Условие

[Наверх](#содержание)

В шаблонах можно использовать условия, для подстановки значений при различных данных.

*Шаблон:*

```
{% if condition %}
{% elif condition %}
{% else %}
{% endif %}
```

*Пример:*

```
server {
    {% if nginx_port == "80" %}
        listen {{ nginx_port }}

    {% elif nginx_port == "443" %}
        listen {{ nginx_port }} ssl;
        server_name example.com;
        ssl_certificate example.com.crt;
        ssl_certificate_key example.com.key;

    {% endif %}

    server_name {{ server_name }};
    location / {
        proxy_pass http://{{ ip_forward }}:{{ port_forward }};
    }
}
```


## Цикл

[Наверх](#содержание)

Цикл предназначен для перебора и обрабоки массива значений.

*Шаблон:*

```
{% for item in array %}
{% endfor %}
```

*Пример:*

```
{% for person in people %}
    {{ person }}
{% endfor %}
```


## Фильтры

[Наверх](#содержание)

Фильтры позволяют преобразовывать данные JSON в данные YAML,
разбивать URL-адреса для извлечения имени хоста,
получать хеш SHA1 строки, добавлять или умножать целые числа и многое другое.

*Примеры:*

Указание значение переменной по умолчанию.

    {{ some_variable | default(5) }}

Вывод списка, только с уникальными значениями.

    {{ some_list | unique }}

Объединение двух списков.

    {{ some_list1 | union(some_list2) }}

Пересечение 2 списков (уникальный список всех элементов в обоих):

    {{ some_list1 | intersect(some_list2) }}

Разница в 2 списках (элементы в 1, которые не существуют во 2):

    {{ some_list1 | difference(some_list2) }}

[Полная документация по фильтрам](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html).


# Тестирование

[Наверх](#содержание)

`Molecule` - фрэймворк, предназначенный для тестирования ролей в Ansible.


## Установка

[Наверх](#содержание)

Установка molecule.

    python3 -m pip install molecule

[Документация по molecule](https://ansible.readthedocs.io/projects/molecule/)


## Иницилизация роли

[Наверх](#содержание)

Molecule использует ansible-galaxy под капотом для создания обычных макетов ролей.
Дополнительно создаётся директория molecule с нужными файлами.

    molecule init role [namespace].[role_name] --driver-name [driver]

Файлы:

* `molecule.yml` - это центральная точка входа.
Можно настроить каждый инструмент, который Molecule будет использовать при тестировании.
* `converge.yml` - это файл playbook, содержащий вызов роли.
* `verify.yml` - это файл Ansible, позволяет писать специальные тесты
для состояния контейнера после завершения выполнения роли.

Также есть специальный файл `INSTALL.rst`, он не является обязательным,
но содержит инструкции, какое дополнительное программное обеспечение
или шаги по настройке нужно предпринять, чтобы взаимодействовать с драйвером.

*Примеры:*

Создание роли `nginx` с использованием драйвера docker.

    molecule init role acme.nginx --driver-name docker


## Запуск тестирования

[Наверх](#содержание)

Тестирование с помощью molecule запускается в корневой директории роли.

Запуск создания экземпляра.

    molecule create

Просмотр создания экзепляра.

    molecule list

Запустить проверку роли.

    molecule converge

Запуск тестирования роли.

    molecule verify

Если требуется проверить экземпляр руками.

    molecule login

Уничтожение экзепляра.

    molecule destroy

Также есть команда, если требуется запустить сразу `create`,
`converge`, `verify` и `destroy` по очереди.

    molecule test


## Тесты для проверки состояния

[Наверх](#содержание)

Для инфраструктуры можно (и нужно) писать тесты, которые проверяют,
что нужный код исполнилняется верно, все нужные сервисы запускаются,
файлы верно настроены и т.д.

Сейчас по стандарту используется проверка инфраструктуры с помощью Ansible.
Раньше было использование с применением testinfra написанная на Python.


# Полезные ссылки

[Наверх](#содержание)

[Специальные переменные от ansible](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html).
