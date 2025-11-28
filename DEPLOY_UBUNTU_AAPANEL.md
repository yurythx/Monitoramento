# Deploy Stack Monitoramento - Ubuntu 24.04 (Cloudflare Tunnel)

## Visão Geral
Esta documentação detalha o processo de implantação da Stack de Monitoramento (Zabbix, GLPI, Grafana) em Ubuntu 24.04 com Cloudflare Tunnel.

## Serviços Incluídos
- **Zabbix Server**: Monitoramento de infraestrutura
- **Zabbix Web**: Interface web do Zabbix
- **GLPI**: Sistema de gerenciamento de TI (ITSM)
- **Grafana**: Dashboards e visualização de dados
- **PostgreSQL**: Banco de dados para Zabbix
- **MySQL**: Banco de dados para GLPI
- **Portainer**: Gerenciamento de containers

## Pré-requisitos

### Sistema Operacional
- Ubuntu 24.04 LTS
- IP do servidor: 192.168.0.121 (ajustar conforme necessário)
- aaPanel instalado e configurado

### Dependências
```bash
# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Instalar Docker Compose
sudo apt update
sudo apt install docker-compose-plugin
```

### Configurações do Ubuntu
```bash
# Configurar firewall (Tunnel não requer portas públicas)
sudo ufw allow 22/tcp      # SSH
sudo ufw --force enable

# Criar diretórios para volumes
sudo mkdir -p /var/lib/docker/volumes
sudo mkdir -p /mnt/config
sudo chown -R 1000:1000 /mnt/config
```

## Exposição com Cloudflare Tunnel

### Ingress de exemplo
```yaml
tunnel: <SEU_TUNNEL_ID_OU_NOME>
credentials-file: /etc/cloudflared/<SEU_TUNNEL_ID>.json
ingress:
  - hostname: glpi.seudominio.com
    service: http://127.0.0.1:18080
  - hostname: zabbix.seudominio.com
    service: http://127.0.0.1:18081
  - hostname: grafana.seudominio.com
    service: http://127.0.0.1:13000
  - hostname: portainer.seudominio.com
    service: http://127.0.0.1:9000
  - service: http_status:404
```

## Preparação dos Arquivos

### 1. Ajustar arquivo .env principal
```bash
# Copiar e editar arquivo de ambiente
cp .env .env.backup
```

Editar as seguintes variáveis no `.env`:
```env
# PostgreSQL para Zabbix
POSTGRES_USER=zabbix
POSTGRES_PASSWORD=sua_senha_postgresql_segura
POSTGRES_DB=zabbix

# Zabbix Server
DB_SERVER_HOST=postgres
DB_SERVER_PORT=5432
ZBX_SERVER_HOST=zabbix-server
ZBX_SERVER_PORT=10051

# MySQL para GLPI
MYSQL_ROOT_PASSWORD=sua_senha_mysql_root_segura
MYSQL_DATABASE=glpi
MYSQL_USER=glpi
MYSQL_PASSWORD=sua_senha_glpi_segura

# GLPI
GLPI_DB_HOST=mysql
GLPI_DB_PORT=3306
GLPI_DB_NAME=glpi
GLPI_DB_USER=glpi
GLPI_DB_PASSWORD=sua_senha_glpi_segura
```

### 2. Ajustar arquivo .env do Grafana
```bash
# Editar Grafana/.env
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=sua_senha_grafana_segura
```

## Processo de Deploy

### 1. Preparação
```bash
# Navegar para o diretório do projeto
cd /caminho/para/Monitoramento

# Criar rede Docker
docker network create itsm_shared_net
```

### 2. Iniciar Infraestrutura Base
```bash
# Iniciar bancos de dados primeiro
docker compose -f Zabbix/zabbix.yml up -d zabbix-db
docker compose -f GLPI/glpi.yml up -d glpi-db

# Aguardar inicialização (30-60 segundos)
sleep 60
```

### 3. Iniciar Serviços Principais
```bash
# Iniciar Zabbix Server
docker compose -f Zabbix/zabbix.yml up -d zabbix-server

# Aguardar inicialização do Zabbix Server
sleep 30

# Iniciar Zabbix Web
docker compose -f Zabbix/zabbix.yml up -d zabbix-web

# Iniciar GLPI
docker compose -f GLPI/glpi.yml up -d glpi

# Iniciar Grafana
docker compose -f Grafana/grafana.yml up -d grafana
```

### 4. Iniciar Serviços Auxiliares
```bash
# Iniciar backup (opcional)
docker compose -f backup/backup.yml up -d
```

## Verificação

### 1. Status dos Containers
```bash
docker ps
docker compose logs -f
```

### 2. Testes de Conectividade
```bash
# Testar Zabbix Web
curl -I http://127.0.0.1:18081

# Testar GLPI
curl -I http://127.0.0.1:18080

# Testar Grafana
curl -I http://127.0.0.1:13000

# Testar Portainer (se instalado)
curl -I http://127.0.0.1:9000
```

## Configuração Inicial dos Serviços

### Zabbix
1. Acesse `https://zabbix.seudominio.com`
2. Complete o wizard de instalação
3. Configure conexão com banco PostgreSQL
4. Login: Admin / zabbix (alterar senha)

### GLPI
1. Acesse `https://glpi.seudominio.com`
2. Complete o wizard de instalação
3. Configure conexão com banco MySQL
4. Login: glpi / glpi (alterar senha)

### Grafana
1. Acesse `https://grafana.seudominio.com`
2. Login com credenciais do .env
3. Configure data sources (Zabbix, etc.)

### Portainer
1. Acesse `https://portainer.seudominio.com`
2. Crie conta de administrador
3. Configure ambiente Docker local

## Monitoramento e Manutenção

### Comandos Úteis
```bash
# Ver logs
docker compose logs -f [serviço]

# Reiniciar serviços
docker compose restart [serviço]

# Backup de dados
docker compose -f backup/backup.yml up

# Atualizar imagens
docker compose pull
docker compose up -d
```

### Backup Automático
O serviço de backup está configurado para:
- Backup diário dos bancos de dados
- Retenção de 7 dias
- Armazenamento em volume persistente

## Segurança

### Medidas Implementadas
- Senhas fortes em todos os serviços
- Comunicação interna via rede Docker
- SSL/TLS via Cloudflare (Tunnel)
- Firewall configurado
- Volumes com permissões adequadas

### Recomendações Adicionais
- Alterar senhas padrão imediatamente
- Configurar backup externo
- Monitorar logs regularmente
- Manter sistema atualizado

## Solução de Problemas

### Problemas Comuns
1. **Containers não iniciam**: Verificar logs e dependências
2. **Erro de conexão DB**: Verificar credenciais e rede
3. **Tunnel não conecta**: Verificar credenciais e DNS gerado pelo Tunnel
4. **SSL**: Gerenciado pelo Cloudflare; revisar políticas e domínio

### Logs Importantes
```bash
# Logs do Zabbix
docker logs zabbix-server
docker logs zabbix-web

# Logs do GLPI
docker logs glpi

# Logs do Grafana
docker logs grafana
```

## URLs de Acesso

Após a configuração completa:
- **Zabbix**: https://zabbix.seudominio.com
- **GLPI**: https://glpi.seudominio.com
- **Grafana**: https://grafana.seudominio.com
- **Portainer**: https://portainer.seudominio.com

## Atualizações

Para atualizar os serviços:
```bash
docker compose pull
docker compose up -d
```

---

**Nota**: Substitua `seudominio.com` pelo seu domínio real e ajuste as senhas conforme necessário.
