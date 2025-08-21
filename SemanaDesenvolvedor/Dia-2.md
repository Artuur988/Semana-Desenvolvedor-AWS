# 📂 Arquitetura Orientada a Eventos - Ingestão de Arquivos (AWS)

Projeto desenvolvido na **Semana do Desenvolvedor AWS (Escola da Nuvem)** – **Dia 2: Ingestão de Arquivos via S3, Rastreamento e Integração com o Fluxo Principal de Pedidos**.  

---

## 🎯 Objetivo
Adicionar uma **segunda fonte de entrada de pedidos** através de **arquivos JSON no Amazon S3**.  
O fluxo implementado:  

1. Upload de arquivo JSON no **S3**.  
2. **Evento do S3 → SQS Standard** (notificação).  
3. **Lambda** processa, valida e envia pedidos válidos para a **SQS FIFO (Aula 1)**.  
4. **DynamoDB** registra histórico de validação.  
5. **SNS** notifica erros de validação por e-mail.  

---

## 🗺️ Arquitetura (visão geral do fluxo)

![Arquitetura AWS Dia 2](SemanaDesenvolvedor/assets/ArquiteturaDia2.png)


---

## 🛠️ Recursos Criados

- **IAM Role**: `lambda-s3-validation-role-seu-nome`  
- **Amazon S3 Bucket**: `datalake-arquivos-seu-nome`  
- **Amazon SQS (Standard + DLQ)**:  
  - `s3-arquivos-json-queue-seu-nome`  
  - `s3-arquivos-json-dlq-seu-nome`  
- **AWS Lambda**: `validacao-s3-arquivos-lambda-seu-nome`  
- **Amazon DynamoDB Table**: `controle-arquivos-historico-seu-nome`  
- **Amazon SNS Topic**: `notificacao-erro-arquivos-seu-nome`  

---

## 🗂️ Passo a Passo

### 1) IAM Role
- Criada a **role `lambda-s3-validation-role`** com permissões para:  
  - Ler do **S3**.  
  - Ler da fila **SQS de arquivos**.  
  - Escrever no **DynamoDB**.  
  - Publicar no **SNS**.  
  - Enviar mensagens para a **SQS FIFO de pedidos** (Aula 1).  

---

### 2) SQS Standard (Arquivos JSON)
- Criada a **DLQ**: `s3-arquivos-json-dlq`.  
- Criada a **fila principal**: `s3-arquivos-json-queue`.  
  - **Visibility timeout** = 70s (maior que o timeout da Lambda de 60s).  
  - **DLQ configurada** com máximo de 3 tentativas.  

---

### 3) DynamoDB
- Criada tabela: `controle-arquivos-historico`.  
- **Partition key**: `nomeArquivo (String)`.  
- Armazena **status de validação** de cada arquivo processado.  

---

### 4) SNS
- Criado tópico: `notificacao-erro-arquivos`.  
- Subscrição por **e-mail** para receber alertas em caso de erro de validação.  

---

### 5) Lambda de Validação de Arquivos (S3)
- Função: `validacao-s3-arquivos-lambda`.  
- Runtime: **Python 3.12**.  
- **Variáveis de ambiente**:  
  - `DYNAMODB_TABLE_NAME = controle-arquivos-historico`  
  - `SNS_TOPIC_ARN = notificacao-erro-arquivos`  
  - `SQS_FIFO_PEDIDOS_URL = pedidos-fifo-queue.fifo` (Aula 1)  
- Timeout configurado para **1 minuto**.  
- **Trigger**: `s3-arquivos-json-queue` (Batch size = 1).  

---

### 6) S3 (Data Lake)
- Criado bucket: `datalake-arquivos-seu-nome`.  
- Configurada **notificação de eventos** para `.json` → envia mensagens para `s3-arquivos-json-queue`.  
- Caso ocorra erro de permissão (`Unable to validate destination`), foi necessário ajustar a **Access Policy da fila SQS** para permitir envio pelo S3.  

---

## 🔬 Testes Realizados

### Teste 1 — Arquivo válido (`arquivo_com_pedidos.json`)
- Upload no bucket S3.  
- Lambda processa e:  
  - Envia pedidos válidos (`S3P001`, `S3P002`) para a **SQS FIFO de pedidos**.  
  - Registra status `ARQUIVO_VALIDADO` no **DynamoDB**.  
  - Nenhuma notificação de erro via SNS.  
- Logs confirmam integração com a **Lambda de validação da Aula 1**, publicando eventos no **EventBridge**.  

### Teste 2 — Arquivo inválido (`arquivo_schema_invalido.json`)
- Upload no bucket S3.  
- Lambda processa e:  
  - Registra status `ERRO_VALIDACAO_ARQUIVO` no **DynamoDB**.  
  - Nenhum pedido enviado para a fila FIFO.  
  - **E-mail enviado via SNS** notificando o erro.  

---

## ✅ Resultado

- **S3** integrado ao fluxo de pedidos.  
- **Lambda** validando arquivos JSON e extraindo pedidos.  
- **Pedidos válidos** enviados para a mesma **SQS FIFO** da Aula 1.  
- **DynamoDB** rastreando histórico de processamento.  
- **SNS** notificando erros por e-mail.  
- Integração com **EventBridge** garantida pela Lambda de validação de pedidos.  

---

## 🔜 Próximos Passos (Dia 3)
Processar eventos do **EventBridge** (vindos da API e do S3) e persistir pedidos no sistema de forma centralizada.  
