# Травицкий Сергей

# Домашнее задание к занятию «Резервное копирование баз данных»

**Домашнее задание выполните в Google Docs или в md-файле в вашем репозитории GitHub.** 

Для оформления вашего решения в GitHub можете воспользоваться [шаблоном](https://github.com/netology-code/sys-pattern-homework).

Название файла должно содержать номер лекции и фамилию студента. Пример названия: «12.8. Резервное копирование баз данных — Александр Александров».

Перед тем как выслать ссылку, убедитесь, что её содержимое не является приватным, то есть открыто на просмотр всем, у кого есть ссылка. Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Любые вопросы по решению задач задавайте в чате учебной группы.

---

### Задание 1. Резервное копирование

### Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования. 

Необходимо описать, какие варианты резервного копирования подходят в случаях: 

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

*Приведите ответ в свободной форме.*

<details>
<summary>ОТВЕТ</summary>  

1.1. Я думаю что полный бэкап базы раз в неделю для начала. Потом раз в день диференциальный бэкап, который охватывает все изменения с момента последнего бэкапа.


1.2. Полный бэкап я думаю раз в день в период неактивности базы если возможно, если нет возможности делать каддый день полный бэкап, тогда можно реже, но время на восстановление может значительно увеличится. В течение дня инкрементальное копирование (более рационально для экономии ревурсов) с временными метками для простоты автоматической обработки.


1.3.* master-master или  master-slave, даст возможность быстро переключиться на работающую базу.


</details>

### Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

*Приведите ответ в свободной форме.*

<details>
<summary>ОТВЕТ</summary>  

2.1. Для резервирования базы данных используется команда в стандартном варианте: `pg_dump [параметры подключения] [параметры создания резервной копии] [имя БД]` 
 
 примеры:  

 Создание резервной копии `pg_dump dbname > /tmp/dbname.sql`  

 Создание резервной копии отдельной таблицы `pg_dump -t example_table dbname > /tmp/dbname_example_table.sql`  

 Создание резервной копии  базы данных в архивном формате `pg_dump -Fc dbname > /tmp/dbname.bak`  
 
Утилита pg_restore позволяет восстанавливать данные из резервных копий. `pg_restore [параметры подключения] [параметры восстановления резервной копии][файл резервной копии]`
 
 примеры:  

 Восстановление  базы данных: `pg_restore -d dbname /tmp/dbname.bak` 
 
 Восстановление отдельной таблицы базы данных: `pg_restore -a -t example_table -d dbname /tmp/dbname.bak`

2.1.* Я думаю что можно создать bash скрипт и реализовать при помощи планировщика `cron`, в интернете много информации на эту тему.  

 Вот один из примеров.  
 
```
#!/bin/sh
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
PGPASSWORD=password
export PGPASSWORD
pathB=/backup
dbUser=dbuser
database=db
find $pathB \( -name "*-1[^5].*" -o -name "*-[023]?.*" \) -ctime +61 -delete
pg_dump -U $dbUser $database | gzip > $pathB/pgsql_$(date "+%Y-%m-%d").sql.gz
unset PGPASSWORD
```
*Для запуска резервного копирования по расписанию, сохраняем скрипт в файл, например, /scripts/postgresql_dump.sh и создаем задание в планировщике:*

</details>


---

### Задание 3. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL. 

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

*Приведите ответ в свободной форме.*

<details>
<summary>ОТВЕТ</summary>  

Исполбзуется опция `--incremental-base=history:last_full_backup.`  

Несколько примеров:  

 --incremental-base=history:last_backup используется опция, при которой mysqlbackup извлекает LSN последней успешной (не TTS) полной или частичной резервной копии из mysql.backup_history  выполняет инкрементное резервное копирование на его основе.   

```
mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
  --incremental --incremental-base=history:last_backup \
  --backup-dir=/home/dbadmin/temp_dir \
  --backup-image=incremental_image1.bi \
   backup-to-image
```

В следующем примере выполняется инкрементное резервное копирование, аналогичное предыдущему примеру, но оптимистичное по своей природе.   

```
mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
  --incremental=optimistic --incremental-base=history:last_backup \
  --backup-dir=/home/dbadmin/temp_dir \
  --backup-image=incremental_image1.bi 
   backup-to-image
```

резервная копия сохраняется в месте, указанном параметром : --incremental-base=dir:directory_path--incremental-backup-dir  

```
mysqlbackup --defaults-file=/home/dbadmin/my.cnf --incremental \
  --incremental-base=dir:/incr-backup/wednesday \
  --incremental-backup-dir=/incr-backup/thursday \
  backup

```

Так же используя специальную утилиту xtrabackup, которая позволяет делать бэкапы на горячую.   

</details>

---

Задания, помеченные звёздочкой, — дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.


