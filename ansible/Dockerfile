FROM python:3.12-slim

# Client SSH, nécessaire pour qu'Ansible se connecte au serveur
RUN apt-get update && apt-get install -y --no-install-recommends openssh-client \
    && rm -rf /var/lib/apt/lists/*

# Ansible, avec ses collections (dont community.docker)
RUN pip install --no-cache-dir ansible

WORKDIR /ansible
