ARG MYSQL_VERSION=8.0
FROM mysql:${MYSQL_VERSION}

COPY my.cnf /etc/mysql/conf.d/my.cnf

RUN mkdir -p /docker-entrypoint-initdb.d

EXPOSE 3306
CMD ["mysqld"]
