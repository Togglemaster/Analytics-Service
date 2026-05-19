# Analytics Service 📊

Serviço de análise (analytics) do projeto **ToggleMaster**. Este é um worker assíncrono de backend responsável por processar eventos de avaliação de feature flags em tempo real.

## 🎯 Descrição do Serviço

O Analytics Service consome mensagens de uma fila AWS SQS (preenchida pelo Evaluation Service) e persiste os dados de análise em uma tabela DynamoDB. Ele funciona como um processador de eventos assíncrono que:

1. Escuta continuamente a fila AWS SQS
2. Consome mensagens de eventos de avaliação de flags
3. Valida e processa os dados de evento
4. Persiste os dados em DynamoDB para análise histórica
5. Fornece um endpoint `/health` para monitoramento

**Nota:** Este serviço não possui uma API pública (exceto `/health`). É puramente um worker que processa dados em background.

## 📦 Stack Técnico

- **Linguagem:** Python 3.9+
- **Framework:** Flask
- **Serviços AWS:** SQS, DynamoDB
- **Servidor:** Gunicorn
- **Dependências principais:** boto3, python-dotenv

## 🚀 Como Usar

### Pré-requisitos Locais

- Python 3.9 ou superior
- Conta AWS com permissões para SQS e DynamoDB
- AWS CLI configurada (`aws configure`) ou variáveis de ambiente

### Setup Local

#### 1. Clone e Navegue para o Diretório
```bash
cd Analytics-Service
```

#### 2. Configure as Variáveis de Ambiente
Crie um arquivo `.env` na raiz do serviço:

```env
# Porta para health check
PORT=8005

# Configuração AWS
AWS_REGION=us-east-1
AWS_SQS_URL=https://sqs.us-east-1.amazonaws.com/123456789012/togglemaster-events
AWS_DYNAMODB_TABLE=ToggleMasterAnalytics

# (Credenciais AWS serão carregadas via aws configure ou variáveis de ambiente)
```

#### 3. Instale as Dependências
```bash
pip install -r requirements.txt
```

#### 4. Configure a Tabela DynamoDB

Crie a tabela usando AWS CLI:
```bash
aws dynamodb create-table \
    --table-name ToggleMasterAnalytics \
    --attribute-definitions AttributeName=event_id,AttributeType=S \
    --key-schema AttributeName=event_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --region us-east-1
```

#### 5. Inicie o Serviço
```bash
gunicorn --bind 0.0.0.0:8005 app:app
```

O servidor estará disponível em `http://localhost:8005`.

### Testando Localmente

#### Health Check
```bash
curl http://localhost:8005/health
# Resposta esperada: {"status":"ok"}
```

#### Gerar Eventos
Faça requisições no Evaluation Service para gerar eventos:
```bash
curl "http://localhost:8004/evaluate?user_id=test-user-1&flag_name=enable-new-dashboard"
```

#### Verificar Dados no DynamoDB
```bash
aws dynamodb scan \
    --table-name ToggleMasterAnalytics \
    --region us-east-1
```

## 🔧 Variáveis de Ambiente

### Obrigatórias
| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `AWS_REGION` | Região AWS | `us-east-1` |
| `AWS_SQS_URL` | URL completa da fila SQS | `https://sqs.us-east-1.amazonaws.com/123456789012/togglemaster-events` |
| `AWS_DYNAMODB_TABLE` | Nome da tabela DynamoDB | `ToggleMasterAnalytics` |

### Opcionais
| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `PORT` | Porta do servidor | `8005` |
| `LOG_LEVEL` | Nível de log | `INFO` |
| `BATCH_SIZE` | Tamanho do batch de processamento | `10` |

### Credenciais AWS

Configure via um destes métodos:

1. **AWS CLI (Recomendado)**
   ```bash
   aws configure
   ```

2. **Variáveis de Ambiente**
   ```bash
   export AWS_ACCESS_KEY_ID=your_key
   export AWS_SECRET_ACCESS_KEY=your_secret
   export AWS_SESSION_TOKEN=your_token  # se necessário
   ```

3. **IAM Role** (em produção via ECS/Lambda)

## 🔐 GitHub Secrets Necessários

Configure os seguintes secrets no GitHub para CI/CD:

```yaml
AWS_REGION
  Descrição: Região AWS
  Valor: us-east-1

AWS_ACCESS_KEY_ID
  Descrição: AWS Access Key ID
  Valor: <sua-access-key>

AWS_SECRET_ACCESS_KEY
  Descrição: AWS Secret Access Key
  Valor: <sua-secret-key>

AWS_SQS_URL
  Descrição: URL completa da fila SQS
  Valor: https://sqs.us-east-1.amazonaws.com/123456789012/togglemaster-events

AWS_DYNAMODB_TABLE
  Descrição: Nome da tabela DynamoDB
  Valor: ToggleMasterAnalytics

DOCKERHUB_USERNAME
  Descrição: Docker Hub username (para push de imagens)
  Valor: <seu-username>

DOCKERHUB_TOKEN
  Descrição: Docker Hub personal access token
  Valor: <seu-token>
```

## 📊 Monitoramento

### Endpoints de Health Check
```bash
GET /health
```

### Observar Logs
```bash
docker logs analytics-service
# ou localmente
tail -f logs/analytics-service.log
```

### CloudWatch Metrics (AWS)
O serviço envia métricas automaticamente para CloudWatch:
- `EventsProcessed` - Eventos processados
- `EventsFailed` - Eventos falhados
- `ProcessingLatency` - Latência de processamento

## 🐛 Troubleshooting

### Problema: "Unable to locate credentials"
**Solução:** Configure AWS CLI com `aws configure` ou defina variáveis de ambiente

### Problema: "Table not found"
**Solução:** Crie a tabela DynamoDB usando o comando AWS CLI acima

### Problema: "Queue not found"
**Solução:** Verifique se a URL da SQS está correta no `.env`

## 📚 Recursos Adicionais

- [Documentação AWS SQS](https://docs.aws.amazon.com/sqs/)
- [Documentação AWS DynamoDB](https://docs.aws.amazon.com/dynamodb/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [ToggleMaster Architecture](../README.md)

## 👥 Suporte

Para dúvidas ou problemas, abra uma issue no repositório principal ou entre em contato com o time DevOps.
