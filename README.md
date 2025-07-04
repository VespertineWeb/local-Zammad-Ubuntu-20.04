
# Instalação do Zammad com Docker no Ubuntu 20.04

Este guia passo a passo mostra como instalar o Docker, Docker Compose e executar o [Zammad](https://zammad.org/) com Docker no Ubuntu 20.04.

## 1. Instalar Docker

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Remover versões antigas do Docker (se houver)
sudo apt remove docker docker-engine docker.io containerd runc

# Instalar dependências
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Adicionar chave GPG oficial do Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicionar repositório do Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Atualizar lista de pacotes
sudo apt update

# Instalar Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Iniciar e habilitar Docker
sudo systemctl start docker
sudo systemctl enable docker

# Adicionar seu usuário ao grupo docker (para não precisar usar sudo)
sudo usermod -aG docker $USER
```

## 2. Reiniciar sessão

```bash
# Aplicar mudanças do grupo (ou faça logout/login)
newgrp docker

# Verificar se Docker está funcionando
docker --version
docker run hello-world
```

## 3. Instalar Docker Compose (caso necessário)

```bash
# Verificar se o plugin Docker Compose está disponível
docker compose version
```

Se **não estiver disponível**, instale manualmente:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 4. Instalar Zammad com Docker

```bash
# Criar diretório para o Zammad
mkdir -p ~/zammad-docker
cd ~/zammad-docker

# Baixar docker-compose oficial do Zammad
curl -o docker-compose.yml https://raw.githubusercontent.com/zammad/zammad-docker-compose/master/docker-compose.yml
```

### (Opcional) Criar arquivo `.env`

```bash
cat > .env << 'EOF'
POSTGRES_USER=zammad
POSTGRES_PASS=zammad_password_123
POSTGRES_DB=zammad_production
MEMCACHE_SERVERS=zammad-memcached:11211
REDIS_URL=redis://zammad-redis:6379
EOF
```

## 5. Iniciar Zammad

```bash
# Subir todos os serviços
docker compose up -d

# Verificar se está rodando
docker compose ps
```

## 6. Aguardar inicialização

```bash
# Acompanhar logs (Ctrl+C para sair)
docker compose logs -f

# Ou acompanhar apenas o Zammad
docker compose logs -f zammad-railsserver
```

## 7. Verificar funcionamento

```bash
# Ver status dos containers
docker compose ps

# Testar acesso à aplicação
curl -I http://localhost:8080

# Ver logs específicos
docker compose logs zammad-railsserver
```

### Acesso ao Zammad

- **URL:** http://localhost:8080  
- **Primeira configuração:** criar usuário administrador via interface web

---

## Comandos úteis

```bash
# Ver containers em execução
docker compose ps

# Parar Zammad
docker compose down

# Iniciar Zammad
docker compose up -d

# Reiniciar serviço específico
docker compose restart zammad-railsserver

# Ver logs em tempo real
docker compose logs -f

# Fazer backup do banco de dados
docker compose exec zammad-postgresql pg_dump -U zammad zammad_production > backup_$(date +%Y%m%d_%H%M%S).sql

# Acessar terminal do Zammad
docker compose exec zammad-railsserver bash

# Ver uso de recursos
docker stats
```

---

## Solução de problemas

```bash
# Se der erro de permissão com Docker
sudo systemctl restart docker
newgrp docker

# Se não conseguir baixar imagens
sudo systemctl restart docker

# Verificar logs
docker compose logs
```

---

> Execute os comandos na ordem e me avise se aparecer algum erro!  
> O processo todo leva cerca de 5 a 10 minutos dependendo da sua conexão.
