FROM python:3.11-slim

# Install base tools
RUN apt update && apt install -y \
    git curl redis-server mariadb-client gnupg cron\
    && pip install --upgrade pip setuptools wheel

# Install Node.js 18 + Yarn (for asset building)
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g yarn

# Install Frappe bench
RUN pip install frappe-bench

# Create non-root user
RUN useradd -ms /bin/bash frappe
USER frappe
WORKDIR /home/frappe

RUN bench init /home/frappe/frappe-bench --frappe-branch version-15 --skip-redis-config-generation
# TODO: add settings for redis and db connections https://docs.frappe.io/framework/user/en/bench/bench-commands#advanced-setup-based