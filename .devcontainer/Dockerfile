FROM docker.io/frappe/bench:latest

USER frappe

RUN bench init /home/frappe/frappe-bench --frappe-branch version-15 --skip-redis-config-generation

ENV REDIS_CACHE=redis://redis-cache:6379
ENV REDIS_QUEUE=redis://redis-queue:6379
ENV REDIS_SOCKETIO=redis://redis-socketio:6379

WORKDIR /home/frappe/frappe-bench

RUN bench set-redis-cache-host ${REDIS_CACHE}
RUN bench set-redis-queue-host ${REDIS_QUEUE}
RUN bench set-redis-socketio-host ${REDIS_SOCKETIO}

