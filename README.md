containers - содержит информацию для развёртывания контейнеров (jenkins, nexus, stage, app), с использованием docker-compose
jobs - выгрузка информации о jenkins джобах, из контейнера без изменений
CI.log - журнал выполнения джоба CI без персонифицированной информации (IP адреса, названия креденшиалов)
CD.log - журнал выполнения джоба CD без персонифицированной информации (IP адреса, названия креденшиалов)
CI_job.txt - jenkins job, загружает исходный код из github, выполняет тесты, загружает артефакт в nexus  
CD_job.txt - jenkins job, запускает ansible playbook
download.yml - ansible playbook, скачивает артефакт из nexus репозитория и запускает приложение.
screenshot.png - результат запуска приложения
