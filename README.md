# Zabbix-deployer

O [Zabbix](https://www.zabbix.com/) é um sistema de monitoramento open-source que monitora: Servidores, aplicações, serviços, banco de dados, redes e etc. Ele possui uma interface web para administração e pode ser integrado com outras aplicações para geração de dashboards, como o [Grafana](https://grafana.com/).

## Sumário

- [Zabbix-deployer](#zabbix-deployer)
  - [Sumário](#sumário)
  - [1. Requisitos e Dependências](#1-requisitos-e-dependências)
  - [2. Instalação](#2-instalação)
    - [2.1 Diretórios](#21-diretórios)
      - [2.1.1 Banco de dados](#211-banco-de-dados)
      - [2.1.2 Aplicação Grafana](#212-aplicação-grafana)
    - [2.2 Docker-Compose](#22-docker-compose)
      - [2.2.1 Portas](#221-portas)
      - [2.2.2 Volumes](#222-volumes)
      - [2.2.3 Variáveis de ambiente (Environment)](#223-variáveis-de-ambiente-environment)
      - [2.2.4 Rede](#224-rede)
      - [2.2.5 Proxy Reverso](#225-proxy-reverso)
    - [2.3 Usando Docker-Compose (Swarm Mode)](#23-usando-docker-compose-swarm-mode)
      - [2.3.1 Variáveis de Ambiente (Environment)](#231-variáveis-de-ambiente-environment)
      - [2.3.2 Docker Secrets](#232-docker-secrets)
    - [2.4 Construindo](#24-construindo)
      - [2.4.1 Docker-Compose](#241-docker-compose)
      - [2.4.2 Swarm Mode](#242-swarm-mode)

  
## 1. Requisitos e Dependências

- [Docker e Docker-Compose](https://docs.docker.com/)
- Versões testadas: ***Zabbix 6.4.4*** e ***Grafana 10.0.2***

## 2. Instalação

### 2.1 Diretórios

#### 2.1.1 Banco de dados
```bash
# Crie os diretórios

# Dir.de dados
$ mkdir $(pwd)/db
```

#### 2.1.2 Aplicação Grafana
```bash
# Crie os diretórios

# Dir.dados
$ mkdir $(pwd)/data
```
Sugestão (no linux):

- Dir.db: /var/lib/zabbix-db
- Dir.data: /var/lib/grafana-data

### 2.2 Docker-Compose

#### 2.2.1 Portas

```yml
# (docker-compose|stack.docker-compose).yml (Em "services.frontend_app", "services.grafana_app")

# Descomente (e/ou altere) as portas/serviços que você deseja oferecer.

ports:
# Porta Web (HTTP) zabbix-frontend
  - 8080:8080

ports:
# Porta Web (HTTP) grafana
  - 3000:3000
```

> Obs: não recomendado o uso da **Porta Web** via HTTP, use um proxy reverso no local com HTTPS, como: [Nginx](https://github.com/nutecuneal/proxy-manager-deployer) ou [Traefik](https://doc.traefik.io/traefik/). Por isso, só descomente essa instrução se realmente souber o que está fazendo.

#### 2.2.2 Volumes

```yml
# (docker-compose|stack.docker-compose).yml (Em "services.db", "services.grafana_app")

# Aponte para as pastas criadas anteriormente.

# Antes

# Zabbix-db
volumes:
  - /$(pwd)/zabbix-db:/var/lib/postgresql/data

# Grafana
volumes:
  - /$(pwd)/grafana-data:/var/lib/grafana

# Depois (exemplo)

# Zabbix-db
volumes:
  - /var/lib/zabbix-db:/var/lib/postgresql/data

# Grafana
volumes:
  - /var/lib/grafana-data:/var/lib/grafana
```

#### 2.2.3 Variáveis de ambiente (Environment)

```yml
# docker-compose.yml
# Em "services.server-app", "services.frontend-app", "services.db"

environment:
# Nome do Host do zabbix-server.
  - ZBX_SERVER_HOST=

# Nome do Host do zabbix-db.
  - DB_SERVER_HOST=

# Nome do usuário no banco.
  - POSTGRES_USER=

# Senha do usuário acima.
  - POSTGRES_PASSWORD=

# Nome do banco de dados criado.
  - POSTGRES_DB=

# Configura o Timezone do frontend-app.
  - PHP_TZ=
```

#### 2.2.4 Rede

```yml
# (docker-compose|stack.docker-compose).yml (Em networks.zabbix-server-net.ipam)

# Altere o valor caso necessário. 

config:
# Endereço da rede
  - subnet: '172.18.0.0/28'
```

#### 2.2.5 Proxy Reverso

```yml
# (docker-compose|stack.docker-compose).yml (Em networks)

# Ajuste o campo "name" para ingressar na rede do seu proxy reverso.

reverse-proxy:
  name: 'reverse-proxy'
  external: true
```

### 2.3 Usando Docker-Compose (Swarm Mode)

#### 2.3.1 Variáveis de Ambiente (Environment)

```yml 
# stack.docker-compose.yml
# Em "services.server_app", "services.frontend_app", "services.db"

environment:
# Nome do Host do zabbix_server.
  - ZBX_SERVER_HOST=

# Nome do Host do zabbix_db.
  - DB_SERVER_HOST=

# Nome do banco de dados criado.
  - POSTGRES_DB=

# Configura o Timezone do frontend_app.
  - PHP_TZ=
```

#### 2.3.2 Docker Secrets

> Crie dois **secrets**, um para armazenar o nome do usuário e o outro para senha de usuário do banco, com os seguintes nomes: **zbxsec-db-user** e **zbxsec-dbuser-passwd**. Consulte a documentação do Docker caso necessário. 


### 2.4 Construindo 

#### 2.4.1 Docker-Compose

```bash
# Execute
$ docker compose up

# OU 
$ docker compose -f docker-compose.yml up -d
```

#### 2.4.2 Swarm Mode

```bash
# Execute
$ docker stack deploy --compose-file stack.docker-compose.yml {STACK_NAME}
```