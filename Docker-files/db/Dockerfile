FROM MYSQL:8.0.33
LABEL "project"="vprofile"
LABEL "author"="Adenle"

ENV MYSQL_ROOT_PASSWORD="adenle"
ENV MYSQL_DATABASE="accounts"

ADD db_backup.sql /docker-entrypoint-initdb.d/db_backup.sql

EXPOSE 3306