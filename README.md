# Monitoramento GLPI + Zabbix + Grafana

Este projeto orquestra um ambiente de monitoramento completo utilizando GLPI, Zabbix e Grafana via Docker Compose.

## Estrutura do Projeto
- **GLPI/**: Gestão de ativos e chamados
- **Zabbix/**: Monitoramento de infraestrutura
- **Grafana/**: Visualização de dados

## Pré-requisitos
- Docker
- Docker Compose

## Variáveis de Ambiente
Cada serviço possui um arquivo `.env` próprio com variáveis sensíveis:
- `GLPI/.env`
- `Grafana/.env`
- `Zabbix/.env`

> **Importante:** Altere as senhas padrão antes de subir o ambiente.

## Subindo o ambiente
Na raiz do projeto, execute:

```powershell
docker compose up -d
```

## Acesso aos Serviços
- **GLPI**: http://localhost:18080 (credenciais conforme `GLPI/.env`)
- **Zabbix Web**: http://localhost:18081 (usuário `Admin`, senha conforme `Zabbix/.env`)
- **Grafana**: http://localhost:13000 (usuário/senha conforme `Grafana/.env`)

## Integração
- Todos os serviços compartilham a rede `itsm_shared_net` (criada automaticamente pelo Compose na raiz) para facilitar integrações.
- O Grafana já instala o plugin `grafana-zabbix` para integração com Zabbix.

## Recomendações
- Altere as senhas padrão nos arquivos `.env` antes de subir.
- Não versionar arquivos `.env` com senhas reais.
- Consulte a documentação oficial de cada serviço para configurações avançadas.

---

### Observações de configuração

- O `docker-compose.yml` da raiz orquestra GLPI, Zabbix e Grafana.
- Zabbix utiliza `DB_SERVER_HOST=zabbix-db` para apontar o banco Postgres.
- A rede `itsm_shared_net` é criada automaticamente pela raiz; não é necessário criá-la manualmente.

---

### Estrutura dos arquivos `.env`

**GLPI/.env**
```
MYSQL_ROOT_PASSWORD=defina_um_valor_forte_ou_use_random
MYSQL_DATABASE=glpi
MYSQL_USER=glpi
MYSQL_PASSWORD=glpi
```

**Grafana/.env**
```
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=admin
```

**Zabbix/.env**
```
POSTGRES_USER=zabbix
POSTGRES_PASSWORD=zabbix
POSTGRES_DB=zabbix_db
```
## Documentação Completa da Stack

### Visão Geral
- Arquitetura híbrida: GLPI usa MariaDB, Zabbix usa PostgreSQL 15, Grafana armazena localmente.
- Todos os serviços compartilham a rede `itsm_shared_net` e iniciam via `docker compose` na raiz do projeto.
- Portas não padrão para evitar conflitos: GLPI `18080`, Zabbix Web `18081`, Zabbix Server `11051`, Grafana `13000`.

### Pré-Requisitos
- Windows com Docker Desktop e Docker Compose (v2+).
- Portas livres: `18080`, `18081`, `11051`, `13000`.
- Permissões para criar volumes Docker.

### Estrutura de Pastas
```
Monitoramento-GLPI-Zabbix-Grafana/
├── GLPI/
│   ├── .env
│   └── docker-compose.yml
├── Grafana/
│   ├── .env
│   ├── docker-compose.yml
│   └── provisioning/
│       └── datasources/
├── Zabbix/
│   ├── .env
│   └── docker-compose.yml
└── docker-compose.yml
```

### Serviços e Portas
- GLPI (Gestão de Serviços): `http://localhost:18080/`
- Zabbix Web (Frontend): `http://localhost:18081/`
- Zabbix Server (porta de agentes/trappers): `11051/tcp`
- Grafana (Dashboards): `http://localhost:13000/`

### Configuração (.env)
- Defina credenciais fortes antes de subir os serviços.
- Arquivos `.env` por serviço:

GLPI/.env
```
MYSQL_ROOT_PASSWORD=sua_senha_root_glpi
MYSQL_DATABASE=glpi_db
MYSQL_USER=glpi_user
MYSQL_PASSWORD=sua_senha_glpi_db

# Variáveis lidas pelo contêiner glpi/glpi
GLPI_DB_HOST=glpi-db
GLPI_DB_PORT=3306
GLPI_DB_NAME=glpi_db
GLPI_DB_USER=glpi_user
GLPI_DB_PASSWORD=sua_senha_glpi_db
```

Zabbix/.env
```
POSTGRES_USER=zabbix_user
POSTGRES_PASSWORD=sua_senha_zabbix_db
POSTGRES_DB=zabbix_db
```

Grafana/.env
```
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=sua_senha_grafana
```

### Orquestração (Compose)
- Use apenas o `docker-compose.yml` da raiz para orquestrar toda a stack.
- Rede `itsm_shared_net` é criada automaticamente pela raiz. Se usar compose por serviço (GLPI/Zabbix/Grafana), crie manualmente: `docker network create itsm_shared_net`.
- Volumes persistem dados:
  - `glpi_db_data`, `glpi_files`
  - `zabbix_db_data`
  - `grafana_data`

### Subir, Parar e Status
- Subir: `docker compose up -d`
- Parar: `docker compose down`
- Status: `docker compose ps`
- Logs:
  - GLPI: `docker logs glpi --tail 200`
  - Zabbix Web: `docker logs zabbix-web --tail 200`
  - Zabbix Server: `docker logs zabbix-server --tail 200`
  - MariaDB: `docker logs glpi-db --tail 200`
  - PostgreSQL: `docker logs zabbix-db --tail 200`

### Primeira Execução
- GLPI:
  - Acesse `http://localhost:18080/`.
  - Login padrão: usuário `glpi`, senha `glpi`.
  - Caso peça instalador: siga o assistente e confirme as variáveis `GLPI_DB_*` do `.env`.
- Zabbix:
  - Acesse `http://localhost:18081/`.
  - Login padrão: usuário `Admin`, senha `zabbix`.
- Grafana:
  - Acesse `http://localhost:13000/`.
  - Login conforme `Grafana/.env`.
  - Instalação do plugin Zabbix já declarada em `GF_INSTALL_PLUGINS`. Configure o Data Source Zabbix com host `zabbix-server` e porta `10051` dentro da rede.

### Backups
- Crie a pasta `backups/` na raiz para guardar dumps.
- Backup MariaDB (GLPI):
  - `docker exec glpi-db mysqldump -u"$env:MYSQL_USER" -p"$env:MYSQL_PASSWORD" "${env:MYSQL_DATABASE}" > backups/glpi_$(Get-Date -Format yyyyMMdd_HHmm).sql`
- Backup PostgreSQL (Zabbix):
  - `docker exec zabbix-db pg_dump -U zabbix_user -d zabbix_db > backups/zabbix_$(Get-Date -Format yyyyMMdd_HHmm).sql`
- Restauração MariaDB:
  - `docker exec -i glpi-db mysql -u"$env:MYSQL_USER" -p"$env:MYSQL_PASSWORD" "${env:MYSQL_DATABASE}" < backups/glpi_<data>.sql`
- Restauração PostgreSQL:
  - `docker exec -i zabbix-db psql -U zabbix_user -d zabbix_db < backups/zabbix_<data>.sql`

### Upgrade de Banco (Zabbix PostgreSQL)
- Ao atualizar Postgres (ex.: 13 → 15), o diretório de dados pode ser incompatível.
- Passos seguros:
  - Faça backup (`pg_dump`).
  - `docker compose down`.
  - Remova o volume antigo: `docker volume rm monitoramento-glpi-zabbix-grafana_zabbix_db_data` (nome pode variar).
  - Suba novamente: `docker compose up -d`.
  - Restaure o dump se necessário.

### Segurança
- Troque todas as senhas padrão nos `.env` por valores fortes.
- Limite exposição de portas a redes internas ou use um reverse proxy com TLS (Traefik/Nginx).
- Defina senhas únicas para DB e apps; evite reuso.
- Faça backups periódicos e teste restauração.

### Solução de Problemas
- Login GLPI falhou:
  - Use `glpi/glpi` e não `root`. Verifique `GLPI/.env`.
  - Se instalador aparecer, complete o assistente.
- Zabbix Web inacessível ou login falhado:
  - Confira saúde do `zabbix-db` e `zabbix-server`.
  - Reset de senha do Admin (exemplo): `docker exec -it zabbix-db psql -U zabbix_user -d zabbix_db -c "UPDATE users SET passwd=md5('zabbix') WHERE alias='Admin';"`
- Rede/bancos indisponíveis:
  - Cheque `docker compose ps` e `docker logs`.
  - Verifique colisão de portas no host.

### Comandos Úteis
- `docker compose up -d` — sobe tudo em segundo plano
- `docker compose down` — derruba os serviços
- `docker compose ps` — vê status e health
- `docker logs <container> --tail 200` — últimos logs
- `docker volume ls` — lista volumes
- `docker network ls` — lista redes

### Referências
- Zabbix com PostgreSQL 15 (compatibilidade 10–15 em versões 6.4+).
- GLPI oficial: imagens `glpi/glpi` e suporte a MariaDB.
- Grafana: plugin Zabbix `alexanderzobnin-zabbix-app`.
