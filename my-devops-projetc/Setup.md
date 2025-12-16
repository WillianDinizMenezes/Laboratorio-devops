# 📘 Guia Completo de Setup - My DevOps Project

> Guia passo a passo detalhado para containerizar e fazer deploy do portfólio na AWS

## 📋 Índice

1. [Pré-requisitos](#-pré-requisitos)
2. [Preparação do Ambiente Local](#-fase-1-preparação-do-ambiente-local)
3. [Containerização com Docker](#-fase-2-containerização-com-docker)
4. [Teste Local](#-fase-3-teste-local-do-container)
5. [Configuração do Amazon ECR](#️-fase-4-configuração-do-amazon-ecr)
6. [Push da Imagem para ECR](#-fase-5-push-da-imagem-para-o-ecr)
7. [Provisionamento da EC2](#️-fase-6-provisionamento-da-instância-ec2)
8. [Deploy na EC2](#-fase-7-deploy-na-ec2)
9. [Verificação e Testes](#-verificação-e-testes)
10. [Troubleshooting](#-troubleshooting)
11. [Limpeza de Recursos](#️-limpeza-de-recursos)

---

## 🔧 Pré-requisitos

### Ferramentas Necessárias

#### 1. Docker Desktop

**Windows/Mac:**
- Baixe em [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- Instale e reinicie o computador se necessário

**Linux:**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

**Verificar instalação:**
```bash
docker --version
docker ps
```

Você deve ver algo como:
```
Docker version 24.0.7, build afdd53b
```

#### 2. AWS CLI

**Windows:**
- Baixe o instalador MSI em [AWS CLI](https://aws.amazon.com/cli/)

**Mac:**
```bash
brew install awscli
```

**Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Verificar instalação:**
```bash
aws --version
```

Saída esperada:
```
aws-cli/2.13.25 Python/3.11.5 Linux/5.15.0 exe/x86_64
```

#### 3. Conta AWS

1. Acesse [aws.amazon.com](https://aws.amazon.com)
2. Clique em "Create an AWS Account"
3. Siga o processo de criação (cartão de crédito necessário)
4. Ative o Free Tier para recursos gratuitos

#### 4. Criar Credenciais AWS (IAM)

1. Acesse o [AWS Console](https://console.aws.amazon.com)
2. Busque por "IAM" na barra superior
3. No menu lateral: **Users** → **Create user**
4. Nome do usuário: `devops-cli-user`
5. Marque: **Provide user access to the AWS Management Console** (opcional)
6. Next → **Attach policies directly**
7. Selecione as políticas:
   - `AmazonEC2FullAccess`
   - `AmazonEC2ContainerRegistryFullAccess`
8. **Create user**
9. Em **Security credentials**, clique em **Create access key**
10. Escolha: **Command Line Interface (CLI)**
11. Copie e guarde:
    - **Access Key ID**: `AKIA...`
    - **Secret Access Key**: `wJalrXUtn...`

⚠️ **IMPORTANTE**: Guarde essas credenciais em local seguro! Não as compartilhe.

#### 5. Configurar AWS CLI

```bash
aws configure
```

Insira as informações:
```
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: wJalrXUtn...
Default region name [None]: us-east-1
Default output format [None]: json
```

**Testar configuração:**
```bash
aws sts get-caller-identity
```

Saída esperada:
```json
{
    "UserId": "AIDA...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/devops-cli-user"
}
```

---

## 📦 Fase 1: Preparação do Ambiente Local

### Passo 1.1: Estrutura do Projeto

Certifique-se de que seu projeto tem a seguinte estrutura:

```
my-devops-project/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── script.js
├── icon/
│   └── (seus ícones)
└── Dockerfile (vamos criar)
```

**Verificar estrutura:**
```bash
cd /caminho/para/my-devops-project
ls -la
```

### Passo 1.2: Testar Website Localmente (Opcional)

Abra o `index.html` no navegador para verificar se está funcionando:

```bash
# Mac
open index.html

# Linux
xdg-open index.html

# Windows (PowerShell)
start index.html
```

---

## 🐳 Fase 2: Containerização com Docker

### Passo 2.1: Criar o Dockerfile

Na raiz do projeto, crie o arquivo `Dockerfile`:

```bash
touch Dockerfile
```

### Passo 2.2: Escrever o Dockerfile

Abra o `Dockerfile` no seu editor favorito e adicione:

```dockerfile
# Usa uma imagem Nginx estável e leve baseada em Alpine Linux
FROM nginx:stable-alpine

# Remove a configuração padrão do nginx (opcional, mas recomendado)
RUN rm -rf /usr/share/nginx/html/*

# Copia todos os arquivos do site para o diretório padrão do nginx
COPY index.html /usr/share/nginx/html/
COPY css/ /usr/share/nginx/html/css/
COPY js/ /usr/share/nginx/html/js/
COPY icon/ /usr/share/nginx/html/icon/

# Expõe a porta 80 para acesso HTTP
EXPOSE 80

# Inicia o nginx em modo foreground
CMD ["nginx", "-g", "daemon off;"]
```

### 📚 Entendendo o Dockerfile

| Linha | Explicação |
|-------|------------|
| `FROM nginx:stable-alpine` | Imagem base: Nginx versão estável em Alpine Linux (~5MB) |
| `RUN rm -rf ...` | Remove arquivos padrão do Nginx |
| `COPY index.html ...` | Copia arquivo principal |
| `COPY css/ ...` | Copia pasta de estilos |
| `COPY js/ ...` | Copia pasta de scripts |
| `COPY icon/ ...` | Copia pasta de ícones |
| `EXPOSE 80` | Documenta que o container usa porta 80 |
| `CMD [...]` | Comando executado quando container inicia |

### Passo 2.3: Criar arquivo .dockerignore (Opcional mas Recomendado)

```bash
echo ".git
.gitignore
README.md
SETUP.md
*.md
node_modules/
.DS_Store
Thumbs.db" > .dockerignore
```

Isso evita copiar arquivos desnecessários para a imagem.

### Passo 2.4: Build da Imagem Docker

```bash
docker build -t my-portfolio:v1.0 .
```

**Explicação do comando:**
- `docker build`: Constrói uma imagem
- `-t my-portfolio:v1.0`: Tag da imagem (nome:versão)
- `.`: Contexto de build (diretório atual)

**Saída esperada:**
```
[+] Building 12.3s (11/11) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:stable-alpine
 => [1/5] FROM docker.io/library/nginx:stable-alpine
 => [internal] load build context
 => [2/5] RUN rm -rf /usr/share/nginx/html/*
 => [3/5] COPY index.html /usr/share/nginx/html/
 => [4/5] COPY css/ /usr/share/nginx/html/css/
 => [5/5] COPY js/ /usr/share/nginx/html/js/
 => [6/5] COPY icon/ /usr/share/nginx/html/icon/
 => exporting to image
 => => naming to docker.io/library/my-portfolio:v1.0
```

### Passo 2.5: Verificar Imagem Criada

```bash
docker images
```

**Saída esperada:**
```
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
my-portfolio    v1.0      1a2b3c4d5e6f   30 seconds ago   23.5MB
nginx           stable    7g8h9i0j1k2l   2 weeks ago      23.2MB
```

---

## 🧪 Fase 3: Teste Local do Container

### Passo 3.1: Executar o Container Localmente

```bash
docker run -d -p 8080:80 --name portfolio-test my-portfolio:v1.0
```

**Explicação do comando:**
- `docker run`: Cria e executa um container
- `-d`: Executa em background (detached mode)
- `-p 8080:80`: Mapeia porta 8080 (host) → 80 (container)
- `--name portfolio-test`: Nome do container
- `my-portfolio:v1.0`: Imagem a ser usada

### Passo 3.2: Verificar se o Container Está Rodando

```bash
docker ps
```

**Saída esperada:**
```
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                  NAMES
abc123def456   my-portfolio:v1.0  "nginx -g 'daemon of…"   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   portfolio-test
```

### Passo 3.3: Testar no Navegador

Abra seu navegador e acesse:

```
http://localhost:8080
```

✅ Seu portfólio deve aparecer funcionando!

### Passo 3.4: Verificar Logs (Opcional)

```bash
docker logs portfolio-test
```

**Comandos Úteis:**
```bash
# Logs em tempo real
docker logs -f portfolio-test

# Últimas 50 linhas
docker logs --tail 50 portfolio-test

# Ver estatísticas de uso
docker stats portfolio-test
```

### Passo 3.5: Parar e Remover o Container de Teste

```bash
# Parar o container
docker stop portfolio-test

# Remover o container
docker rm portfolio-test

# Verificar
docker ps -a
```

---

## ☁️ Fase 4: Configuração do Amazon ECR

### Passo 4.1: Acessar o Console AWS

1. Acesse [console.aws.amazon.com](https://console.aws.amazon.com)
2. Faça login com suas credenciais

### Passo 4.2: Navegar para o ECR

1. Na barra de busca superior, digite: **ECR**
2. Clique em **Elastic Container Registry**
3. Ou acesse diretamente: https://console.aws.amazon.com/ecr

### Passo 4.3: Criar um Repositório

1. Clique em **Get Started** (se primeira vez) ou **Create repository**
2. Configure:
   - **Visibility settings**: `Private`
   - **Repository name**: `my-portfolio`
   - **Tag immutability**: `Disabled` (permite sobrescrever tags)
   - **Scan on push**: `Enabled` ✅ (recomendado para segurança)
   - **KMS encryption**: `Disabled` (para Free Tier)
3. Clique em **Create repository**

### Passo 4.4: Anotar a URI do Repositório

Após criar, você verá algo como:

```
123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio
```

⚠️ **IMPORTANTE**: Copie esta URI! Você precisará em vários comandos.

**Dica**: Salve em um arquivo temporário:
```bash
echo "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio" > ecr-uri.txt
```

---

## 📤 Fase 5: Push da Imagem para o ECR

### Passo 5.1: Obter Credenciais do ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

⚠️ **Substitua**:
- `us-east-1` pela sua região AWS
- `123456789012` pelo seu Account ID

**Saída esperada:**
```
Login Succeeded
```

**Como encontrar seu Account ID:**
```bash
aws sts get-caller-identity --query Account --output text
```

### Passo 5.2: Tagar a Imagem para o ECR

```bash
docker tag my-portfolio:v1.0 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio:v1.0
```

**Verificar tags:**
```bash
docker images | grep my-portfolio
```

Você verá duas entradas:
```
my-portfolio                                          v1.0      1a2b3c4d5e6f   5 minutes ago   23.5MB
123456789012.dkr.ecr.us-east-1.../my-portfolio       v1.0      1a2b3c4d5e6f   5 minutes ago   23.5MB
```

### Passo 5.3: Push da Imagem

```bash
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio:v1.0
```

**Saída esperada:**
```
The push refers to repository [123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio]
5f70bf18a086: Pushed
c2c789d2d3e4: Pushed
e8c96d51d6c7: Pushed
v1.0: digest: sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08 size: 1234
```

### Passo 5.4: Verificar no Console AWS

1. Volte para o console ECR
2. Clique no repositório `my-portfolio`
3. Você deve ver a imagem com tag `v1.0`
4. Status: **Scan complete** (se habilitou scan)

---

## 🖥️ Fase 6: Provisionamento da Instância EC2

### Passo 6.1: Navegar para EC2

1. No console AWS, busque por **EC2**
2. Clique em **EC2**
3. Ou acesse: https://console.aws.amazon.com/ec2

### Passo 6.2: Lançar Instância

Clique em **Launch Instance**

#### 6.2.1 Nome e Tags

- **Name**: `my-portfolio-server`

#### 6.2.2 Aplicação e Sistema Operacional (Amazon Machine Image)

- **AMI**: `Amazon Linux 2023 AMI`
- **Architecture**: `64-bit (x86)`
- ✅ Verifique o selo **Free tier eligible**

#### 6.2.3 Tipo de Instância

- **Instance type**: `t2.micro`
- ✅ Verifique **Free tier eligible**

#### 6.2.4 Par de Chaves (Key Pair)

1. Clique em **Create new key pair**
2. Configure:
   - **Key pair name**: `my-portfolio-key`
   - **Key pair type**: `RSA`
   - **Private key file format**:
     - `.pem` para Linux/Mac
     - `.ppk` para Windows (PuTTY)
3. Clique em **Create key pair**
4. O arquivo será baixado automaticamente

⚠️ **CRÍTICO**: 
- Guarde este arquivo em local seguro!
- No Linux/Mac, mude as permissões:
  ```bash
  chmod 400 ~/Downloads/my-portfolio-key.pem
  mv ~/Downloads/my-portfolio-key.pem ~/.ssh/
  ```

#### 6.2.5 Configurações de Rede

- **VPC**: `Default` (padrão)
- **Subnet**: `No preference`
- **Auto-assign public IP**: `Enable` ✅

#### 6.2.6 Firewall (Security Groups)

1. Selecione: **Create security group**
2. Configure:
   - **Security group name**: `my-portfolio-sg`
   - **Description**: `Security group for portfolio website`

3. **Inbound security group rules**:

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| SSH | TCP | 22 | My IP | SSH access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Public web access |

Clique em **Add security group rule** para adicionar a regra HTTP.

⚠️ **Importante**: 
- "My IP" detecta automaticamente seu IP atual
- `0.0.0.0/0` significa "qualquer IP" (necessário para acesso público ao site)

#### 6.2.7 Configurar Armazenamento

- **Volume**: `8 GiB` (padrão)
- **Volume type**: `gp3` (padrão)
- ✅ Free Tier: até 30 GB

#### 6.2.8 Detalhes Avançados - IAM Instance Profile

📌 **IMPORTANTE**: Precisamos criar um IAM Role primeiro!

**Criar IAM Role:**

1. Em outra aba, acesse **IAM Console**
2. No menu lateral: **Roles** → **Create role**
3. Configure:
   - **Trusted entity type**: `AWS service`
   - **Use case**: `EC2`
   - Clique em **Next**
4. **Add permissions**:
   - Busque e selecione: `AmazonEC2ContainerRegistryReadOnly`
   - Clique em **Next**
5. **Role details**:
   - **Role name**: `EC2-ECR-ReadOnly-Role`
   - **Description**: `Allows EC2 to pull images from ECR`
   - Clique em **Create role**

Volte para a aba de criação da EC2:
- Em **IAM instance profile**, clique no ícone de refresh 🔄
- Selecione: `EC2-ECR-ReadOnly-Role`

### Passo 6.3: Revisar e Lançar

1. No canto direito, veja o **Summary**
2. Revise todas as configurações
3. Clique em **Launch instance**

### Passo 6.4: Aguardar Inicialização

1. Clique em **View all instances**
2. Aguarde o status:
   - **Instance state**: `Running` ✅
   - **Status check**: `2/2 checks passed` ✅

Isso pode levar 2-3 minutos.

### Passo 6.5: Anotar Informações

Clique na instância e anote:

- **Public IPv4 address**: `54.123.45.67`
- **Instance ID**: `i-0abc123def456789`
- **Public IPv4 DNS**: `ec2-54-123-45-67.compute-1.amazonaws.com`

**Dica**: Salve essas informações:
```bash
echo "EC2_IP=54.123.45.67" >> ~/.ec2-info
echo "EC2_DNS=ec2-54-123-45-67.compute-1.amazonaws.com" >> ~/.ec2-info
```

---

## 🚀 Fase 7: Deploy na EC2

### Passo 7.1: Conectar à Instância via SSH

**Linux/Mac:**
```bash
ssh -i ~/.ssh/my-portfolio-key.pem ec2-user@54.123.45.67
```

**Windows (PowerShell com OpenSSH):**
```powershell
ssh -i C:\Users\SeuUsuario\.ssh\my-portfolio-key.pem ec2-user@54.123.45.67
```

**Windows (PuTTY):**
1. Abra PuTTYgen
2. Load → Selecione `my-portfolio-key.pem`
3. Save private key → `my-portfolio-key.ppk`
4. Abra PuTTY:
   - Host: `ec2-user@54.123.45.67`
   - Connection → SSH → Auth → Browse → Selecione `.ppk`
   - Open

**Primeira conexão:**
```
The authenticity of host '54.123.45.67' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
```

**Saída esperada:**
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'

[ec2-user@ip-172-31-xx-xx ~]$
```

✅ Você está conectado na EC2!

### Passo 7.2: Atualizar o Sistema

```bash
sudo yum update -y
```

### Passo 7.3: Instalar Docker

```bash
# Instalar Docker
sudo yum install docker -y

# Iniciar o serviço Docker
sudo systemctl start docker

# Habilitar Docker no boot
sudo systemctl enable docker

# Adicionar ec2-user ao grupo docker
sudo usermod -a -G docker ec2-user

# Verificar instalação
docker --version
```

**Saída esperada:**
```
Docker version 20.10.25, build b82b9f3
```

### Passo 7.4: Aplicar Permissões do Docker

```bash
# Sair da sessão SSH
exit

# Conectar novamente
ssh -i ~/.ssh/my-portfolio-key.pem ec2-user@54.123.45.67
```

**Testar permissões:**
```bash
docker ps
```

Se funcionar sem `sudo`, está pronto! ✅

### Passo 7.5: Autenticar Docker com ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

⚠️ **Substitua** o Account ID e região!

**Saída esperada:**
```
Login Succeeded
```

### Passo 7.6: Pull da Imagem do ECR

```bash
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio:v1.0
```

**Saída esperada:**
```
v1.0: Pulling from my-portfolio
c2c789d2d3e4: Pull complete
5f70bf18a086: Pull complete
e8c96d51d6c7: Pull complete
Digest: sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08
Status: Downloaded newer image for 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio:v1.0
```

### Passo 7.7: Executar o Container em Produção

```bash
docker run -d \
  -p 80:80 \
  --name my-portfolio-prod \
  --restart always \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/my-portfolio:v1.0
```

**Explicação dos parâmetros:**
- `-d`: Background (detached)
- `-p 80:80`: Mapeia porta 80 (HTTP padrão)
- `--name`: Nome do container
- `--restart always`: Reinicia automaticamente se a EC2 reiniciar
- URI completa da imagem no ECR

### Passo 7.8: Verificar se Está Rodando

```bash
docker ps
```

**Saída esperada:**
```
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED         STATUS         PORTS                NAMES
xyz789abc123   123456789012.dkr.ecr...my-portfolio:v1.0                 "nginx -g 'daemon of…"   10 seconds ago  Up 9 seconds   0.0.0.0:80->80/tcp   my-portfolio-prod
```

### Passo 7.9: Testar Internamente

```bash
curl localhost
```

Você deve ver o HTML do seu portfólio! ✅

---

## ✅ Verificação e Testes

### Teste 1: Acessar pelo Navegador

Abra seu navegador e acesse:

```
http://54.123.45.67
```

ou

```
http://ec2-54-123-45-67.compute-1.amazonaws.com
```

🎉 **Seu portfólio está online na AWS!**

### Teste 2: Verificar Logs

```bash
# Logs em tempo real
docker logs -f my-portfolio-prod

# Últimas 100 linhas
docker logs --tail 100 my-portfolio-prod
```

### Teste 3: Verificar Recursos

```bash
# Estatísticas de uso
docker stats my-portfolio-prod

# Informações do container
docker inspect my-portfolio-prod
```

### Teste 4: Teste de Persistência

```bash
# Parar o container
docker stop my-portfolio-prod

# Aguardar 5 segundos
sleep 5

# Container deve reiniciar automaticamente
docker ps | grep my-portfolio-prod
```

Se aparecer, o `--restart always` está funcionando! ✅

### Teste 5: Simular Reinicialização da EC2

⚠️ **Cuidado**: Isso desconectará você da SSH!

```bash
sudo reboot
```

Aguarde 2-3 minutos e conecte novamente:

```bash
ssh -i ~/.ssh/my-portfolio-key.pem ec2-user@54.123.45.67
docker ps
```

O container deve estar rodando automaticamente! ✅

---

## 🔧 Troubleshooting

### Problema 1: "Cannot connect to the Docker daemon"

**Sintoma:**
```
Cannot connect to the Docker daemon. Is the docker daemon running?
```

**Solução:**
```bash
# Iniciar Docker
sudo systemctl start docker

# Verificar status
sudo systemctl status docker

# Se não estiver no grupo docker
sudo usermod -a -G docker ec2-user
exit
# Conectar novamente via SSH
```

### Problema 2: Site Não Abre no Navegador

**Checklist:**

1. **Container está rodando?**
   ```bash
   docker ps
   ```

2. **Security Group tem porta 80 aberta?**
   - AWS Console → EC2 → Security Groups
   - Verifique se tem regra: `HTTP TCP 80 0.0.0.0/0`

3. **IP público está correto?**
   - Confirme no console EC2

4. **Testar internamente:**
   ```bash
   curl localhost
   curl 127.0.0.1
   ```

5. **Verificar logs:**
   ```bash
   docker logs my-portfolio-prod
   ```

### Problema 3: "No basic auth credentials" no ECR

**Sintoma:**
```
Error: no basic auth credentials
```

**Solução:**
```bash
# Re-autenticar no ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

### Problema 4: Permissão Negada no Docker

**Sintoma:**
```
permission denied while trying to connect to the Docker daemon socket
```

**Solução:**
```bash
# Adicionar ao grupo docker
sudo usermod -a -G docker ec2-user

# Logout e login
exit
ssh -i ~/.ssh/my-portfolio-key.pem ec2-user@54.123.45.67

# Testar
docker ps
```

### Problema 5: EC2 Sem Permissão para Acessar ECR

**Sintoma:**
```
Error response from daemon: pull access denied
```

**Solução:**

1. Vá para **IAM Console** → **Roles**
2. Encontre `EC2-ECR-ReadOnly-Role`
3. Verifique se tem a policy: `AmazonEC2ContainerRegistryReadOnly`
4. Vá para **EC2 Console** → Sua instância
5. **Actions** → **Security** → **Modify IAM role**
6. Selecione `EC2-ECR-ReadOnly-Role`
7. **Update IAM role**
8. Na EC2, tente o pull novamente

### Problema 6: Build Falha - "no such file or directory"

**Sintoma:**
```
COPY failed: file not found
```

**Solução:**

1. Verifique se está na raiz do projeto:
   ```bash
   pwd
   ls -la
   ```

2. Confirme que as pastas existem:
   ```bash
   ls -la css/ js/ icon/
   ```

3. Verifique o Dockerfile:
   ```bash
   cat Dockerfile
   ```

4. Rebuild:
   ```bash
   docker build -t my-portfolio:v1.0 .
   ```

---

## 🧹 Limpeza de Recursos

⚠️ **IMPORTANTE**: Para evitar custos na AWS, sempre limpe recursos que não está usando!

### Passo 1: Parar e Remover Container

```bash
# Conectar na EC2
ssh -i ~/.ssh/my-portfolio-key.pem ec2-user@54.123.45.