# 🚀 My DevOps Project

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://nginx.org/)

> Projeto DevOps completo: containerização de portfólio pessoal com Docker e deploy na AWS EC2 usando Amazon ECR.

## 📋 Sobre o Projeto

Este projeto demonstra a implementação de práticas DevOps fundamentais, containerizando um portfólio web pessoal (HTML, CSS, JavaScript) usando Docker e realizando deploy manual em infraestrutura AWS. O projeto utiliza Amazon ECR como registry privado de imagens e Amazon EC2 para hospedar a aplicação.

### 🎯 Objetivos de Aprendizado

- ✅ Containerização de aplicações web estáticas
- ✅ Gerenciamento de imagens Docker com Amazon ECR
- ✅ Provisionamento e configuração de instâncias EC2
- ✅ Implementação de boas práticas de segurança (IAM, Security Groups)
- ✅ Deploy manual de containers em ambiente de produção

## 🏗️ Arquitetura

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Código Local   │────▶│  Docker Image   │────▶│   Amazon ECR    │
│  (Portfolio)    │     │  (Nginx Alpine) │     │   (Registry)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │    Browser      │◀────│   Amazon EC2    │
                        │  (User Access)  │     │  (t2.micro)     │
                        └─────────────────┘     └─────────────────┘
```

## 🛠️ Tecnologias Utilizadas

- **Docker**: Containerização da aplicação
- **Nginx**: Servidor web (Alpine Linux - versão leve)
- **Amazon ECR**: Registry privado de imagens Docker
- **Amazon EC2**: Hospedagem da aplicação containerizada
- **AWS CLI**: Gerenciamento de recursos AWS via linha de comando
- **IAM**: Gerenciamento de permissões e roles
- **Security Groups**: Firewall para controle de tráfego

## 📂 Estrutura do Projeto

```
my-devops-project/
├── index.html              # Página principal do portfólio
├── css/                    # Estilos CSS
│   └── style.css
├── js/                     # Scripts JavaScript
│   └── script.js
├── icon/                   # Ícones e imagens
│   └── ...
├── Dockerfile              # Configuração do container
├── README.md               # Este arquivo
└── SETUP.md               # Guia passo a passo detalhado
```

## 🐳 Dockerfile Customizado

O Dockerfile foi otimizado para copiar apenas os arquivos necessários:

```dockerfile
FROM nginx:stable-alpine

RUN rm -rf /usr/share/nginx/html/*

COPY index.html /usr/share/nginx/html/
COPY css/ /usr/share/nginx/html/css/
COPY js/ /usr/share/nginx/html/js/
COPY icon/ /usr/share/nginx/html/icon/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### ⚡ Otimizações Implementadas

- **Imagem base Alpine**: ~5MB (comparado a ~133MB da versão padrão)
- **Remoção de arquivos padrão**: Limpeza do diretório nginx antes da cópia
- **Cópia seletiva**: Apenas diretórios necessários são copiados
- **Multi-stage build ready**: Estrutura preparada para futuras otimizações

## 🚀 Quick Start

### Pré-requisitos

- Docker Desktop instalado
- AWS CLI configurado
- Conta AWS ativa
- Git (para clonar o repositório)

### Execução Local

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/my-devops-project.git
cd my-devops-project

# Build da imagem
docker build -t my-portfolio:latest .

# Executar localmente
docker run -d -p 8080:80 --name portfolio my-portfolio:latest

# Acessar no navegador
open http://localhost:8080
```

### Deploy na AWS

Para instruções completas de deploy na AWS, consulte o [**SETUP.md**](SETUP.md).

## 📊 Recursos AWS Utilizados

| Recurso | Tipo | Custo Estimado |
|---------|------|----------------|
| EC2 | t3.micro | Free Tier (750h/mês) |
| ECR | Repositório privado | $0.10/GB/mês |
| Data Transfer | Saída de dados | $0.09/GB (após Free Tier) |

> ⚠️ **Aviso**: Sempre monitore seus custos no AWS Cost Explorer e desligue recursos quando não estiverem em uso.

## 🔒 Segurança

### Implementações de Segurança

- ✅ Security Groups configurados com regras mínimas necessárias
- ✅ IAM Role com princípio de least privilege (apenas leitura ECR)
- ✅ Repositório ECR privado
- ✅ Scan de vulnerabilidades habilitado no ECR
- ✅ Chaves SSH privadas para acesso EC2

### Melhorias Futuras de Segurança

- [ ] HTTPS com certificado SSL/TLS (Let's Encrypt)
- [ ] WAF (Web Application Firewall)
- [ ] CloudWatch Logs para auditoria
- [ ] AWS Secrets Manager para credenciais

## 🧪 Testes e Validação

### Testes Locais

```bash
# Verificar logs do container
docker logs portfolio

# Verificar recursos utilizados
docker stats portfolio

# Testar conectividade
curl http://localhost:8080
```

### Testes em Produção

```bash
# Conectar via SSH
ssh -i meu-website-key.pem ec2-user@<EC2_PUBLIC_IP>

# Verificar status do container
docker ps
docker logs meu-website-prod

# Testar internamente
curl localhost
```

## 📈 Métricas e Monitoramento

Atualmente o projeto usa monitoramento básico via:
- `docker stats` - Uso de CPU, memória, rede
- `docker logs` - Logs de acesso e erros do Nginx
- AWS Console - Métricas da instância EC2

**Próximas implementações**:
- CloudWatch para métricas avançadas
- Application Load Balancer com health checks
- SNS para alertas automáticos

## 🗑️ Limpeza de Recursos

Para evitar custos desnecessários, sempre limpe os recursos após testes:

```bash
# Parar e remover container
docker stop meu-website-prod
docker rm meu-website-prod

# Na AWS Console:
# 1. Terminar instância EC2
# 2. Deletar imagens do ECR
# 3. Deletar repositório ECR
# 4. Deletar Security Group
# 5. Deletar IAM Role
```

## 🎓 Aprendizados e Conceitos

Este projeto aborda conceitos fundamentais de DevOps:

1. **Containerização**: Empacotamento de aplicações com suas dependências
2. **Infrastructure as Code** (básico): Configuração reproduzível via Dockerfile
3. **Cloud Computing**: Utilização de serviços AWS
4. **Security Best Practices**: IAM, Security Groups, princípio de least privilege
5. **CI/CD Foundation**: Base para futuras automações

## 🚧 Roadmap

### Fase 1 - Básico ✅
- [x] Containerização com Docker
- [x] Deploy manual na AWS EC2
- [x] Uso de Amazon ECR

### Fase 2 - Automação 🔄
- [ ] CI/CD com GitHub Actions
- [ ] Terraform para IaC
- [ ] Docker Compose para ambientes locais

### Fase 3 - Avançado 📅
- [ ] Kubernetes/ECS para orquestração
- [ ] Load Balancer e Auto Scaling
- [ ] Monitoramento com CloudWatch/Prometheus
- [ ] HTTPS e domínio customizado

## 🤝 Contribuindo

Contribuições são bem-vindas! Sinta-se livre para:

1. Fazer um fork do projeto
2. Criar uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abrir um Pull Request


## 📧 Contato

**Willian Diniz Menezes** - Williandiniz2412@hotmail.com

Linkedin - https://www.linkedin.com/in/willian-diniz-2360b74b/
Github - https://github.com/WillianDinizMenezes

---

⭐ Se este projeto te ajudou, considere dar uma estrela no repositório!



