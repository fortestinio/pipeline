<details>
<summary>Содержимое</summary>
  
- containers - содержит информацию для развёртывания контейнеров (jenkins, nexus, stage, app), с использованием docker-compose
- jobs - выгрузка информации о jenkins джобах, из контейнера без изменений
- CI.log - журнал выполнения джоба CI без персонифицированной информации (IP адреса, названия креденшиалов)
- CD.log - журнал выполнения джоба CD без персонифицированной информации (IP адреса, названия креденшиалов)
- CI_job.txt - jenkins job, загружает исходный код из github, выполняет тесты, загружает артефакт в nexus  
- CD_job.txt - jenkins job, запускает ansible playbook
- download.yml - ansible playbook, скачивает артефакт из nexus репозитория и запускает приложение.
- screenshot.png - результат запуска приложения
  
</details>

<details>
<summary>Последовательность действий</summary>
  
- Создал репозиторий на github для кода приложения, создал token для доступа. 
- По совету знакомого java программиста, создал java web app при помощи https://start.spring.io/ , приложение отображает сообщение Hello по ссылке localhost:9093/test . Проверил, собрав приложение из исходного кода на локальной системе через maven clean package в IDE intellij idea. Для pom.xml отредактировал поле version (13 строка). Само приложение будет работать, пока его не остановить, например прекратить выполнение pipeline в jenkins.
- Загрузил код в репозиторий github при помощи IDE intellij idea , в этой функции IDE дублирует установку git и выполнение команд git init , git add ., git commit -m "" , git remote add origin <URL> , git branch -M <name> , git push -u origin <name>.
- Развернул jenkins, nexus, контейнеры, имитирующие stage и app сервера. Реализовал на docker при помощи docker-compose (подготовил dockerfile для каждого контейнера, docker-compose.yaml). Автоматизировал часть шагов по подготовке контейнеров через dockerfile (установка пакетов, плагинов jenkins, работа с пользователями).
- Настроил jenkins - создал нового локального пользователя, внёс credentials для nexus, github и подключения агентов, прописал информацию об используемом maven, подключил ноды контейнеров stage и app серверов.
- Настроил nexus - создал нового локального пользователя, репозиторий с возможностью анонимного подключения.
- Создал jenkins job тип pipeline для скачивания исходного кода из github, тестирования и сборки при помощи maven, загрузки получившегося артефакта в nexus репозиторий.
- Создал jenkins job тип pipeline с запуском ansible playbook для загрузки приложения на app сервер и его запуска. Ansible playbook выполняется от анонимного пользователя, если репозиторий в nexus был настроен для доступа только именованных пользователей, необходимо добавить в playbook поля для аутентификации.
- Рассмотрел возможность добавления автоматизации через webhook - в моём случае конвеер реализован на локальной системе, для подключения webhook потребуется предоставить доступ к локальному jenkins из интернет или использовать ПО, которое симулирует этот процесс.

- Для jenkins можно автоматизировать процессы создания локального пользователя, credentials, нод, через скрипты, например при помощи jenkins-cli и соответствующих файлов конфигурации.
- Для nexus можно автоматизировать создание репозитория при помощи nexus-cli.
  
</details>

<details>
<summary>Пошаговая инструкция для развёртывания на localhost с использованием ОС Windows</summary>

- 1 Установить git 
- 2 В каком-либо каталоге инициализировать репозиторий и выгрузить конвеер ( git init , git remote add origin "<_URL>" , git branch -M "<_name>" , git pull origin "<_name>" )
- 3 Установить wsl (в окне powershell выполнить команду wsl --install)
- 4 Установить docker desktop
- 5 Перейти в каталог \contaneirs
- 6 Изменить в каждом dockefile "<_username>" , "<_password>" , "<home_path>" на необходимые для каждого из серверов. Изменить в docker-compose.yaml "<_path>" для каждого контейнера на актуальные.
- 7 Выполнить в консоли docker-compose up из каталога с файлом docker-compose.yaml
- 8 Дождаться развёртывания контейнеров
- 9 Узнать IP адреса контейнеров при помощи команды docker inspect "<container_ID>" | findstr "IPAddress" выполненной в консоли cmd или powershell, "<container_ID>" можно узнать при помощи команды docker ps выполненной в консоли cmd или powershell
- 10 Для настройки nexus перейти по http://127.0.0.1:8081/ ввести пароль, как предложено, изменить пароль, при необходимости создать нового пользователя, создать репозиторий "<repository_name>"
- 11 Для настройки jenkins перейти по http://localhost:8080/ ввести пароль, как предложено, установить плагины по умолчанию, при необходимости создать нового пользователя.
- 12 В jenkins добавить информацию о credential для github (если используется приватный репозиторий; для доступа можно использовать token), агентах (можно использовать данные из dockefile указанные в ходе выполнения шага 6 или создать новых пользователей), nexus (можно использовать информацию из шага 11) Dashboard - Настроить Jenkins - Credentials - System - Global credentials (unrestricted)
- 13 Создать ноды в jenkins используя credentials, указанные в 12 шаге, и IP адреса контейнеров, как показано в шаге 9 , в качестве удалённой директории можно использовать "<home_path>" , указанную в шаге 6 или каталог /tmp . Для Host Key Verification Strategy выбрать из выпадающего списка Non verifying Verification strategy. После создания ноды выполнить перезапуск агента, кнопка Relauch agent
- 14 Создать информацию о maven (Dashboard - Настроить Jenkins - Tool; версию можно узнать в контейнере jenkins командой mvn --version), в качестве названия можно использовать maven
- 15 Создать объект pipeline с названием CI . В разделе Pipeline, в поле Definition вставить текст из CI_job.txt , заменив "<_IP>" , "<_repository_name>" , "<_nexus_ID>" , "<_repository_name>" на актуальные. В случае, если бы использовался приватный репозиторий раскомментировать строку //credentialsId: '<github_ID>', указав актуальный "<github_ID>"
- 17 Сохранить
- 18 Запустить pipeline CI и дождаться его выполнения. Проверить наличие артефакта в репозитории "<repository_name>" nexus.
- 19 Создать объект pipeline с названием CD . В разделе Pipeline, в поле Definition вставить текст из CD , заменив "<agent_ID>" на актуальные credentials для ноды. 
- 20 Изменить в контейнере stage "<home_path>"/download.yml" IP адрес контейнера с nexus в строке с url на актуальный, как показано в шаге 9
- 21 В случае, если для хранилища артефактов nexus не предусмотрен анонимный доступ, необходимо в контейнере stage изменить содержимое файла "<home_path>"/download.yml" добавив поля username: password: после поля dest:
- 22 Сохранить
- 23 Запустить pipeline CD
- 24 В браузере перейти по ссылке http://localhost:9093/test

</details>
