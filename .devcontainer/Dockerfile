FROM docker.io/frappe/bench:latest

USER frappe

RUN bench init /home/frappe/frappe-bench --frappe-branch version-15 --skip-redis-config-generation

ENV REDIS_CACHE=redis://redis-cache:6379
ENV REDIS_QUEUE=redis://redis-queue:6379
ENV REDIS_SOCKETIO=redis://redis-socketio:6379

ENV DB_HOST=mariadb
ENV MARIADB_ROOT_PASSWORD=root
ENV DB_PASSWORD = 123
ENV DB_PORT=3306

ENV SITE_NAME=localhost

WORKDIR /home/frappe/frappe-bench

RUN bench set-mariadb-host ${DB_HOST}
RUN bench set-redis-cache-host ${REDIS_CACHE}
RUN bench set-redis-queue-host ${REDIS_QUEUE}
RUN bench set-redis-socketio-host ${REDIS_SOCKETIO}

RUN bench new-site ${SITE_NAME} --admin-password admin --db-root-password ${MARIADB_ROOT_PASSWORD} --no-mariadb-socket