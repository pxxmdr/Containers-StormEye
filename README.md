# Containers-StormEye
Containerização da API Java, banco de dados MySQL e visualização via PHP utilizando Docker para o projeto StormEye, com deploy em nuvem.

---

## 👨‍💻 Integrantes do Projeto

Desenvolvido por 

Pedro Henrique Martins dos Reis (RM555306)
Thamires Ribeiro Cruz (RM558128)
Adonay Rodrigues da Rocha (RM558782)

para a disciplina **DevOps Tools & Cloud Computing** — FIAP 2025.

---

# 🌪️ StormEye - Deploy em Nuvem com Docker

Este repositório contém o processo completo de **deploy em nuvem** do StormEye — um sistema de alerta de catástrofes naturais. A aplicação foi implantada em uma máquina virtual Linux no Azure utilizando **Docker** para o backend Java (Spring Boot) e banco de dados MySQL.

---

## ⚙️ Tecnologias Utilizadas

- Java 17 + Spring Boot
- MySQL 8.0
- Docker
- Azure VM Linux
- Rede Docker customizada

---

## 🚀 Passo a Passo do Deploy

### 1. Atualização e instalação do Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y
docker --version          # Conferir versão do Docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER  # Remover necessidade do sudo
```

> 🔁 Reinicie a sessão ou execute `newgrp docker` para aplicar o grupo.

---

### 2. Criar estrutura de diretórios na VM

```bash
mkdir -p ~/stormeye/backend
mkdir -p ~/stormeye/database
```

---

### 3. Enviar o .jar da aplicação para a VM (comando local)

```bash
scp "C:\Users\User\Desktop\StormEye\StormEye\target\stormeye-0.0.1-SNAPSHOT.jar" admlnx@<IP_DA_VM>:~/stormeye/backend/stormeye.jar
```

---

### 4. Criar Dockerfile do Backend

```bash
nano ~/stormeye/backend/Dockerfile
```

**Conteúdo do Dockerfile:**

```Dockerfile
FROM openjdk:17-jdk-slim

RUN useradd -m stormeyeuser

WORKDIR /home/stormeyeuser/app

COPY stormeye.jar .

EXPOSE 8080

ENV SPRING_PROFILES_ACTIVE=prod

USER stormeyeuser

CMD ["java", "-jar", "stormeye.jar"]
```

---

### 5. Criar rede Docker

```bash
docker network create stormeye_net
```

---

### 6. Build e execução do container do Backend

```bash
cd ~/stormeye/backend
docker build -t stormeye-api .

docker run -d \
  --name stormeye_api \
  --network stormeye_net \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://stormeye_mysql:3306/stormeyedb \
  -e SPRING_DATASOURCE_USERNAME=stormuser \
  -e SPRING_DATASOURCE_PASSWORD=stormpass \
  stormeye-api
```

---

### 7. Criar Dockerfile do MySQL

```bash
nano ~/stormeye/database/Dockerfile
```

**Conteúdo do Dockerfile:**

```Dockerfile
FROM mysql:8.0

ENV MYSQL_ROOT_PASSWORD=admin
ENV MYSQL_DATABASE=stormeyedb
ENV MYSQL_USER=stormuser
ENV MYSQL_PASSWORD=stormpass

EXPOSE 3306
```

---

### 8. Build e execução do container do Banco de Dados

```bash
cd ~/stormeye/database
docker build -t stormeye-mysql .

docker run -d \
  --name stormeye_mysql \
  --network stormeye_net \
  -e MYSQL_ROOT_PASSWORD=admin \
  -e MYSQL_DATABASE=stormeyedb \
  -e MYSQL_USER=stormuser \
  -e MYSQL_PASSWORD=stormpass \
  -p 3306:3306 \
  -v stormeye_db_data:/var/lib/mysql \
  stormeye-mysql
```
---
### 9. Execução do container do PHPMyAdmin

```bash
docker run -d \
  --name stormeye-phpmyadmin \
  --network stormeye_net \
  -p 8081:80 \
  -e PMA_HOST=stormeye_mysql \
  -e PMA_USER=stormuser \
  -e PMA_PASSWORD=stormpass \
  phpmyadmin
```
---

## ✅ Verificações

- Checar containers:
```bash
docker ps
```

- Logs da aplicação:
```bash
docker logs stormeye_api
```

- A API deve estar disponível em:
```
http://<IP_DA_VM>:8080/catastrofes
```

---

## ♨️ Repositório do projeto JAVA, informações e requisitos completos dentro do README.md.
```
https://github.com/ThamiresRC/StormEye

```
---

## 📸 Evidência

(https://youtu.be/WNzFNy_GQUo)

---

## 📦 Requisitos Entregues

- [x] Código-fonte da aplicação
- [x] Dockerfile do backend
- [x] Dockerfile do banco de dados
- [x] Scripts para subir containers e rede
- [x] Deploy funcional na nuvem Azure


