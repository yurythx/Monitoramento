# 🔍 Stack de Monitoramento - GLPI + Zabbix + Grafana

Esta stack oferece uma solução completa de monitoramento e gestão de TI, integrando três ferramentas poderosas em um ambiente Docker orquestrado.

## 📋 Visão Geral

### Componentes da Stack
- **🎫 GLPI**: Sistema de gestão de ativos, inventário e chamados (ITSM)
- **📊 Zabbix**: Plataforma de monitoramento de infraestrutura em tempo real
- **📈 Grafana**: Ferramenta de visualização e dashboards para análise de dados

### Arquitetura
- **Rede**: Todos os serviços compartilham a rede local `monitoramento_default`
- **Volumes**: Dados persistidos em volumes locais Docker
- **Bancos de Dados**: 
  - GLPI utiliza MariaDB 10.11
  - Zabbix utiliza PostgreSQL 15
  - Grafana armazena dados localmente

## 🚀 Início Rápido

### Pré-requisitos
- ✅ Docker Desktop (Windows) ou Docker Engine (Linux)
- ✅ Docker Compose v2+
- ✅ Portas disponíveis: 18080, 18081, 11051, 13000

### Instalação e Execução
1. **Clone ou baixe o projeto**
2. **Configure as variáveis de ambiente** (veja seção abaixo)
3. **Execute a stack**:
```powershell
docker compose up -d
```

### Verificação do Status
```powershell
# Verificar containers em execução
docker compose ps

# Verificar logs de um serviço específico
docker compose logs glpi --tail 50
```

## ⚙️ Configuração de Variáveis de Ambiente

### Estrutura de Arquivos .env
Cada serviço possui seu próprio arquivo `.env` com configurações específicas:

#### 📁 GLPI/.env
```env
# Configurações do Banco MariaDB
MYSQL_ROOT_PASSWORD=SuaSenhaRootSegura123!
MYSQL_DATABASE=glpi_db
MYSQL_USER=glpi_user
MYSQL_PASSWORD=SuaSenhaGLPISegura123!

# Configurações do GLPI
GLPI_DB_HOST=glpi-db
GLPI_DB_PORT=3306
GLPI_DB_NAME=glpi_db
GLPI_DB_USER=glpi_user
GLPI_DB_PASSWORD=SuaSenhaGLPISegura123!
```

#### 📁 Zabbix/.env
```env
# Configurações do PostgreSQL
POSTGRES_USER=zabbix_user
POSTGRES_PASSWORD=SuaSenhaZabbixSegura123!
POSTGRES_DB=zabbix_db
```

#### 📁 Grafana/.env
```env
# Configurações do Grafana
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=SuaSenhaGrafanaSegura123!
```

> ⚠️ **IMPORTANTE**: Sempre altere as senhas padrão por valores seguros antes de executar a stack!

## 🌐 Acesso aos Serviços

| Serviço | URL Local | Porta | Credenciais Padrão |
|---------|-----------|-------|-------------------|
| **GLPI** | http://localhost:18080 | 18080 | `glpi` / `glpi` |
| **Zabbix Web** | http://localhost:18081 | 18081 | `Admin` / `zabbix` |
| **Grafana** | http://localhost:13000 | 13000 | Conforme `.env` |
| **Zabbix Server** | localhost:11051 | 11051 | Para agentes externos |

### Primeiro Acesso

#### 🎫 GLPI
1. Acesse http://localhost:18080
2. Se aparecer o instalador, siga o assistente
3. Use as credenciais: `glpi` / `glpi`
4. **Altere a senha padrão imediatamente**

#### 📊 Zabbix
1. Acesse http://localhost:18081
2. Login: `Admin` / `zabbix`
3. **Altere a senha padrão imediatamente**

#### 📈 Grafana
1. Acesse http://localhost:13000
2. Use as credenciais definidas em `Grafana/.env`
3. O plugin Zabbix já está instalado automaticamente

## 🔧 Gerenciamento da Stack

### Comandos Básicos
```powershell
# Iniciar todos os serviços
docker compose up -d

# Parar todos os serviços
docker compose down

# Ver status dos containers
docker compose ps

# Ver logs de um serviço específico
docker compose logs [serviço] --tail 100

# Atualizar imagens e reiniciar
docker compose pull && docker compose up -d
```

### Estrutura de Volumes
Os dados são persistidos em volumes locais Docker:
- `monitoramento_glpi_db_data` - Dados do MariaDB (GLPI)
- `monitoramento_glpi_files` - Arquivos do GLPI
- `monitoramento_zabbix_db_data` - Dados do PostgreSQL (Zabbix)
- `monitoramento_grafana_data` - Dados do Grafana

## 💾 Backup e Restauração

### Backup dos Bancos de Dados

#### MariaDB (GLPI)
```powershell
# Criar backup
docker exec glpi-db mysqldump -u glpi_user -p glpi_db > backup_glpi_$(Get-Date -Format "yyyyMMdd_HHmm").sql

# Restaurar backup
docker exec -i glpi-db mysql -u glpi_user -p glpi_db < backup_glpi_YYYYMMDD_HHMM.sql
```

#### PostgreSQL (Zabbix)
```powershell
# Criar backup
docker exec zabbix-db pg_dump -U zabbix_user -d zabbix_db > backup_zabbix_$(Get-Date -Format "yyyyMMdd_HHmm").sql

# Restaurar backup
docker exec -i zabbix-db psql -U zabbix_user -d zabbix_db < backup_zabbix_YYYYMMDD_HHMM.sql
```

## 🔗 Integração entre Serviços

### Conectar Grafana ao Zabbix
1. No Grafana, vá em **Configuration > Data Sources**
2. Adicione um novo **Zabbix Data Source**
3. Configure:
   - **URL**: `http://zabbix-web:8080/api_jsonrpc.php`
   - **Username**: `Admin` (ou usuário criado no Zabbix)
   - **Password**: Senha do usuário Zabbix

### Rede Interna
Todos os serviços se comunicam através da rede `monitoramento_default`:
- `glpi` - Container do GLPI
- `glpi-db` - MariaDB do GLPI
- `zabbix-web` - Interface web do Zabbix
- `zabbix-server` - Servidor Zabbix
- `zabbix-db` - PostgreSQL do Zabbix
- `grafana` - Container do Grafana

## 🛠️ Solução de Problemas

### Problemas Comuns

#### GLPI não carrega
```powershell
# Verificar logs
docker compose logs glpi --tail 100

# Verificar banco de dados
docker compose logs glpi-db --tail 100
```

#### Zabbix Web inacessível
```powershell
# Verificar se o servidor Zabbix está rodando
docker compose logs zabbix-server --tail 100

# Verificar banco PostgreSQL
docker compose logs zabbix-db --tail 100
```

#### Reset de senha do Zabbix Admin
```powershell
docker exec -it zabbix-db psql -U zabbix_user -d zabbix_db -c "UPDATE users SET passwd=md5('zabbix') WHERE alias='Admin';"
```

### Verificação de Saúde
```powershell
# Verificar se todos os containers estão saudáveis
docker compose ps

# Verificar uso de recursos
docker stats

# Verificar volumes
docker volume ls | findstr monitoramento
```

## 🔒 Segurança

### Recomendações de Segurança
- ✅ Altere **todas** as senhas padrão
- ✅ Use senhas fortes (mínimo 12 caracteres)
- ✅ Não exponha as portas diretamente na internet
- ✅ Use um reverse proxy com SSL (Nginx/Traefik)
- ✅ Faça backups regulares
- ✅ Mantenha as imagens Docker atualizadas

### Exposição Segura
Para expor os serviços na internet, use um reverse proxy:
```nginx
# Exemplo de configuração Nginx
server {
    listen 443 ssl;
    server_name glpi.seudominio.com;
    
    location / {
        proxy_pass http://127.0.0.1:18080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 📚 Recursos Adicionais

### Documentação Oficial
- [GLPI Documentation](https://glpi-project.org/documentation/)
- [Zabbix Documentation](https://www.zabbix.com/documentation)
- [Grafana Documentation](https://grafana.com/docs/)

### Estrutura do Projeto
```
Monitoramento/
├── docker-compose.yml          # Orquestração principal
├── README.md                   # Esta documentação
├── GLPI/
│   ├── .env                   # Configurações GLPI
│   ├── .env.example          # Template de configuração
│   └── glpi.yml              # Compose específico do GLPI
├── Zabbix/
│   ├── .env                  # Configurações Zabbix
│   ├── .env.example         # Template de configuração
│   └── zabbix.yml           # Compose específico do Zabbix
├── Grafana/
│   ├── .env                 # Configurações Grafana
│   ├── .env.example        # Template de configuração
│   ├── grafana.yml         # Compose específico do Grafana
│   └── provisioning/       # Configurações automáticas
└── backup/
    ├── backup.yml          # Serviço de backup automático
    └── backup_script.sh    # Script de backup
```

---

> 💡 **Dica**: Esta stack foi configurada para usar volumes e redes locais, garantindo isolamento e facilidade de deploy. Todos os dados são persistidos automaticamente nos volumes Docker.
