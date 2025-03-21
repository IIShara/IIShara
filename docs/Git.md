# Git

## Установка
**Linux**
```shell
# Для Debian/Ubuntu 
apt-get install git

# Для Fedora 
yum install git

# Для OpenSUSE
zypper install git
```

## Настройка
Имя и фамилию нужно писать латиницей, через пробел, в кавычках:
```shell
git config --global  user.name "Name Surname"
```
Почту записываем в кавычках:
```shell
git config --global  user.email "your@email"
```
Опционально можно настроить автоматическую поддержку цветов. Эта опция сделает так, чтобы разные типы файлов различались цветами в репозитории:
```shell
git config --global color.ui auto
```
Осталось убедиться, что данные добавлены и корректно отображаются:
```shell
git config --list
```
В ответ на запрос командная строка выведет настройки вашего профиля.

## Создание репозитория
```
git init
```
После исполнения команды появится сообщение об инициализации репозитория. Оно означает, что Git начал отслеживать файлы проекта и будет записывать изменения в скрытую папку .git.

## Рабочий процесс

### git add: добавление файлов в индекс
```shell
# Добавляем в индекс один файл
git add file_name
# Добавляем в индекс несколько файлов 
git add file_name_1 file_name_2 file_name_3
# Добавляем в индекс все изменённые файлы 
git add .
```

### git status: проверка статуса репозитория
Команда `git status` даёт представление о текущем состоянии репозитория. Она показывает, какие неотслеживаемые файлы попали в проект, какие файлы находятся в индексе и какие сохранённые файлы вы изменили в репозитории.

```shell
$ git status   # Запрашиваем текущее состояние репозитория
# Видим файлы, которые находятся в индексе и подготовлены для коммита
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   index.html
        modified:   styles.css
# Видим неотслеживаемые файлы, которые только попали в проект
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        script.js   # Файл script.js не отслеживается Git
# Видим изменённые файлы репозитория, которые ещё не добавлены в индекс 
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        README.md   # Файл README.md был изменён, но не добавлен в индекс
```
### git commit: добавление файлов в репозиторий

Когда все файлы подготовлены к сохранению, их можно перенести из индекса в репозиторий. Для этого нужна команда `git commit` с опцией `-m` и сообщением коммита. Сообщение пишется в кавычках и обычно латиницей:
```shell
git commit -m "Commit message"
```
Сообщения обязательны — по ним разработчики ориентируются в проекте. Есть даже специальный документ — [«Соглашение о коммитах»](https://www.conventionalcommits.org/ru/v1.0.0/).

Если убрать опцию `-m`, то после нажатия **Enter** вы попадёте в текстовый редактор. Там вам нужно будет написать сообщение, сохранить его и выйти.

Бывает так: вы закоммитили файл и затем снова его изменяете. В этом случае можно делать новый коммит, минуя индекс. Для этого необходима опция `-a`:
```shell
git commit -am "Commit message"
```
Другая частая ситуация: вы торопились и ошиблись в сообщении коммита. Можно ввести опцию `--amend` и перезаписать сообщение последнего коммита:
```shell
git commit --amend -m "New commit message"
```

### git log: просмотр журнала коммитов
Команда `git log` показывает историю коммитов в обратном хронологическом порядке. Вы можете посмотреть хеш, сообщение, дату и ник автора коммита.
```shell
$ git log   # Запрос на просмотр журнала коммитов

# Информация о третьем сделанном коммите
commit 3f6f9e1f58e30e0d3a0d0ab764c0b30a5b621d4a   # Хеш первого коммита
Author: John Doe <johndoe@example.com>   # Автор первого коммита
Date:   Thu Apr 21 10:26:52 2024 +0300   # Дата первого коммита
    Update README.md   # Сообщение первого коммита

# Информация о втором сделанном коммите
commit acd1e81729dc2ee2dc107ba345fa1ab7e6cfbff9
Author: Jane Smith <janesmith@example.com>
Date:   Wed Apr 20 16:45:39 2024 +0300
    Add new feature

# Информация о первом сделанном коммите
commit 7df1e8c33b0a617b3a72c785a67e45d0d932a180
Author: John Doe <johndoe@example.com>
Date:   Mon Apr 18 09:12:21 2024 +0300
    Initial commit
```
**Запрос на вывод истории коммитов в одну строку**

```shell
$ git log --oneline   # Запрос на вывод истории коммитов в одну строку
3f6f9e1 Update README.md
acd1e81 Add new feature
7df1e8c Initial commit
```

### git show: просмотр коммита

Команда `git show` выводит информацию об одном коммите. Сообщение делится на два блока: часть с метаданными и список изменений, внесённых в коммит.
```shell
$ git show abc12345  # Запрос на просмотр коммита с хешем abc12345

# Метаданные
commit abc12345  # Хеш коммита
Author: John Doe <johndoe@example.com>  # Автор коммита
Date:   Thu Apr 21 10:26:52 2024 +0300  # Дата и время коммита
    Update README.md  # Сообщение коммита

# Список изменений в файле README.md
diff --git a/README.md b/README.md
index abcdef1..1234567 100644
--- a/README.md
+++ b/README.md
@@ -1,3 +1,3 @@
-# My Project  # Старое содержимое строки
+# My Awesome Project  # Новое содержимое строки
```
Если ввести `git show` без хеша, то выведется содержимое последнего коммита.
  
### git diff: просмотр изменений до коммита

Команда `git diff` показывает разницу между последним коммитом и текущим состоянием репозитория. То есть последний коммит сравнивается со всеми неотслеживаемыми файлами, которые ещё не переведены в индекс.

Можно добавить имя файла и сравнить его содержимое с последним коммитом.

Ещё вариант: вместо имени файла можно использовать хеш коммита. Также можно добавить опцию `--staged` и сравнить версию кода после последнего коммита с отслеживаемым состоянием репозитория — со всеми файлами, которые находятся в индексе.

```shell
# Смотрим разницу между последним коммитом и текущим состоянием репозитория
git diff

# Разница между последним коммитом и текущим состоянием файла 
git diff file_name

# Разница между последним коммитом и коммитом с указанным хешем 
git diff commit_hash

# Разница между последним коммитом и отслеживаемым состоянием репозитория 
git diff --staged
```

### git restore: отмена изменений
Команда `git restore` возвращает файл к состоянию последнего коммита. Она отменяет все изменения, если файл не перенесён в индекс. Если файл попал в индекс, то вместе с названием команды нужно использовать опцию `--staged`.
```shell
# Вернуть неотслеживаемый файл к состоянию последнего коммита 
git restore file_name

# Вернуть все файлы из индекса к состоянию последнего коммита
git restore --staged 

# Вернуть указанный файл из индекса к состоянию последнего коммита
git restore --staged file_name
```

### git rm: удаление файлов из индекса
Команда `git rm` позволяет удалить файл, который по ошибке попал в индекс. После выполнения команды файл пропадёт из индекса и из папки на вашем компьютере, в которой хранится проект. Если вы хотите удалить файл только из индекса, то команду `git rm` нужно использовать вместе с опцией `--cached`.
```shell
# Удалить файл из индекса и рабочей директории
git rm file_name

# Удалить файл из индекса и оставить в папке на компьютере 
git rm --cached file_name
```
### git reset: откат коммита
Команда `git reset` позволяет отменить любое количество сделанных коммитов и вернуть проект к какому-то состоянию в прошлом. Команду нужно выполнять с осторожностью, поскольку она может навсегда переписать историю проекта.

На выбор можно использовать три режима: `--soft`, `--mixed` и `--hard`.

В режиме `--soft` проект откатывается к указанному коммиту и переводит все последующие коммиты в индекс. Вы можете сразу сделать новый коммит и перезаписать историю проекта, оставив исходные файлы без изменений.

В режиме `--mixed` откаченные файлы попадают в неотслеживаемую зону. Вы можете эти файлы изменить, удалить или вернуть обратно в индекс.

В режиме `--hard` проект откатывается к указанному коммиту и удаляет все последующие коммиты без возможности их восстановления.
```shell
# Откатываемся и переводим последующие коммиты в индекс
git reset --soft commit_hash

# Откатываемся и переводим последующие коммиты в неотслеживаемую зону
git reset --mixed commit_hash

# Откатываемся и удаляем все последующие коммиты
git reset --hard commit_hash
```
Перед выполнением `git reset` мы рекомендуем всегда делать резервную копию проекта, на случай непредвиденного удаления файлов.

## Ветвление
Вся разработка в Git происходит в ветках. Они сохраняют коммиты и организуют их в цепочку: перемещаясь по ветке от одного коммита к другому можно отследить изменения в проекте. При работе с ветками вы будете часто их создавать, просматривать, переименовывать, переключать, объединять и удалять.

### git branch <branch_name>: создание новой ветки
После первого коммита Git автоматически создаёт первую ветку. Обычно в ней хранят стабильную версию проекта для пользователей продукта. Под остальные задачи разработчики создают отдельные ветки с помощью команды `git branch`:
```shell
git branch branch_name
```
По названию ветки должно быть понятно, что в ней происходит. Например, если в названии упоминается слово `bugfix`, то ветка предназначена для исправления ошибок. Слово `feature` указывает на разработку какой-то функции. А вот случайное название `test10.24` не значит ничего, и таких названий лучше избегать.

Ветку с неудачным названием можно переименовать:
```shell
git branch -m old_branch_name new_branch_name

# old_branch_name — старое имя ветки 
# new_branch_name — новое имя ветки
```
### git branch: просмотр веток
Команда `git branch` позволяет получить список всех доступных веток в проекте. Также она проставляет символ звёздочки слева от текущей активной ветки:
```shell
# Запрашиваем список всех доступных веток 
git branch

# Результат вывода
  bugfix/fix-bug
  * maine
  feature/new-feature
```
### git checkout: переключение между ветками
Команда `git checkout` позволяет переключиться с одной ветки на другую:
```shell
git checkout branch_name
```
Также можно одной командой создать новую ветку и сразу в неё перейти:
```shell
git checkout -b branch_name
```
У команды `git checkout` есть более современная альтернатива:
```shell
git switch branch_name
```
Команда `git switch` безопасней и больше подходит новичкам. Перед каждым переключением она автоматически проверяет рабочую директорию и не срабатывает, если переход на выбранную ветку может привести к потере данных.

### git merge: слияние репозиториев
Команда `git merge` позволяет добавить изменения из одной ветки в другую. Такой процесс называется слиянием, и он завершается появлением общего коммита для объединённых веток. По этому коммиту можно отследить историю каждой ветки.
```shell
# Переключаемся на основную ветку, которая будет принимать изменения 
git checkout main_branch

# Сливаем изменения из второстепенной ветки в основную 
git merge secondary_branch
```
### git branch -d <branch_name>: удаление ветки

После слияния второстепенная ветка больше не нужна и мы её можем удалить.
```shell
# Проверяем текущую ветку
git branch

# Результат вывода
  main_branch
  * secondary_branch

# Переключаемся на основную ветку
git checkout main_branch

# Удаляем второстепенную ветку
git branch -d secondary_branch
```

## Удалённый репозиторий

### git remote add origin url: привязка локального и удалённого репозитория
С помощью командной строки переместитесь в папку с проектом на своём компьютере. Теперь вы можете выполнить команду `git remote add`, которая установит связь между вашим локальным и удалённым репозиторием на GitHub.

К команде нужно добавить два параметра: имя вашего удалённого репозитория и его адрес. Адрес вы найдёте на странице своего профиля во вкладке SSH.
```shell
# Перемещение в папку с проектом
cd путь/к/папке/с/проектом

# Привязка локального репозитория к удалённому на GitHub
git remote add origin git@github.com:ваш_профиль/ваш_репозиторий.git
```
### git remote: просмотр удалённых репозиториев
Если вы часто взаимодействуете с GitHub, то с вашим локальным может быть связано множество удалённых репозиториев. Если ввести команду `git remote`, то можно посмотреть название этих репозиториев и отсортировать все ненужные.
```shell
# Запрашиваем список удалённых репозиториев, которые связаны с локальным
git remote

# Пример вывода: два удалённых репозитория связаны с нашим локальным
  origin
  upstream
```
### git remote — v: просмотр удалённых URL-адресов
Команда `git remote` показывает только названия удалённых репозиториев, которые связаны с вашим локальным. К команде можно добавить опцию `-v` и посмотреть удалённые URL-адреса. По URL-адресам будет видно, какие изменения вы делали.
```shell
# Запрос списка удалённых репозиториев с URL-адресами
git remote -v

# Пример вывода с URL-адресами
  origin  https://github.com/user/repo.git (fetch)
  origin  https://github.com/user/repo.git (push)
  upstream  https://github.com/otheruser/repo.git (fetch)
  upstream  https://github.com/otheruser/repo.git (push)
```
### git push: отправка изменений в удалённый репозиторий
Команда `git push` загружает изменения из локального репозитория в удалённый.

Во время первой загрузки нужно использовать команду с опцией `-u`. Это свяжет локальную и удалённую ветки и синхронизирует их для последующих операций. Для второй и всех последующих загрузок опция `-u` для связанных веток не понадобится.
```shell
# Команда для первой загрузки изменений в удалённый репозиторий: текущая ветка будет связана с веткой main в удалённом репозитории origin 
git push -u origin main

# Команда для второй и последующих загрузок изменений в удалённый репозиторий 
git push
```
### git pull: получение изменений из удалённого репозитория
Команда `git pull` скачивает изменения из удалённого репозитория в локальный.
```shell
# Скачиваем изменения из удалённого репозитория и добавляем их в локальную ветку
git pull
```