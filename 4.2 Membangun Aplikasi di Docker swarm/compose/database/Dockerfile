FROM postgres:14

COPY ./musicapp.sql /docker-entrypoint-initdb.d/create_tables.sql

EXPOSE 5432
