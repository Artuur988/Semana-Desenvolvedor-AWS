# 📦 Semana do Desenvolvedor AWS - Arquitetura Orientada a Eventos  

Repositório com os projetos desenvolvidos na **Semana do Desenvolvedor AWS (Escola da Nuvem)**.  
Durante 4 dias, construímos uma **arquitetura serverless, desacoplada e orientada a eventos** para o **processamento de pedidos**.  

---

## 📑 Índice

- [🚀 Dia 1 — Ingestão de Pedidos via API e EventBridge](#-Dia-1--ingestão-de-pedidos-via-api-e-eventbridge)  
- [📂 Dia 2 — Ingestão de Arquivos via S3 e Rastreamento](#-Dia-2--ingestão-de-arquivos-via-s3-e-rastreamento)  
- [⚙️ Dia 3 — Processamento Central e Persistência](#️-Dia-3--processamento-central-e-persistência)  
- [🔄 Dia 4 — Fluxos Adicionais e DLQs](#-Dia-4--fluxos-adicionais-e-dlqs)  
- [📊 Arquitetura Final](#-arquitetura-final)  

---

## 🚀 Dia 1 — Ingestão de Pedidos via API e EventBridge

- Criado **API Gateway REST** para receber pedidos (`/pedidos`).  
- Lambda de **pré-validação** envia pedidos para uma **fila SQS FIFO** (com DLQ).  
- Lambda de **validação** lê da SQS, valida e publica evento `NovoPedidoValidado` no **EventBridge**.  

✅ **Resultado**:  
- API REST funcional.  
- Pipeline assíncrono com **SQS FIFO + DLQ**.  
- Publicação de eventos no **EventBridge**.  

---

## 📂 Dia 2 — Ingestão de Arquivos via S3 e Rastreamento

- Criado bucket **S3** para upload de arquivos JSON de pedidos.  
- Eventos do S3 → **SQS Standard** → Lambda de **validação de arquivos**.  
- Lambda valida, envia pedidos válidos para a **SQS FIFO de pedidos (Aula 1)** e registra histórico em **DynamoDB**.  
- Em caso de erro → **notificação por e-mail via SNS**.  

✅ **Resultado**:  
- Arquivos JSON processados no mesmo pipeline da API.  
- Histórico no **DynamoDB**.  
- Notificações automáticas em caso de erro.  

---

## ⚙️ Dia 3 — Processamento Central e Persistência

- Criada fila **SQS de pedidos pendentes** (Standard + DLQ).  
- Criada Lambda de **processamento central** que consome essa fila.  
- Nova regra no **EventBridge** envia `NovoPedidoValidado` para essa fila.  
- Lambda processa pedidos e persiste em **DynamoDB** (`pedidos-db`).  

✅ **Resultado**:  
- Processamento unificado de pedidos da **API** e do **S3**.  
- Persistência centralizada em DynamoDB.  

---

## 🔄 Dia 4 — Fluxos Adicionais e DLQs

- Criados fluxos de **cancelamento e alteração de pedidos**.  
- Novas regras do **EventBridge** para `CancelarPedido` e `AlterarPedido`.  
- Filas SQS + Lambdas específicas para cada operação.  
- Reforço do uso de **DLQs** em todos os pontos.  

✅ **Resultado**:  
- Arquitetura capaz de lidar com **criação, validação, alteração e cancelamento de pedidos**.  
- **DLQs testadas** com falhas simuladas.  
- Sistema mais resiliente e preparado para falhas.  

---

## 📊 Arquitetura Final

Abaixo, a arquitetura final construída ao longo das 4 aulas:  

![Arquitetura AWS Dia 2](./assets/ArquiteturaCompleta.png)

## 🎉 Conclusão

Durante 4 dias foi construída uma **arquitetura moderna, escalável e resiliente** baseada em serviços **serverless** da AWS:

- **API Gateway**
- **Lambda Functions**
- **Amazon SQS (FIFO e Standard)**
- **Amazon EventBridge**
- **Amazon S3**
- **Amazon DynamoDB**
- **Amazon SNS**

📌 O resultado final é um **sistema de processamento de pedidos totalmente orientado a eventos**, capaz de lidar com **criação, validação, persistência, alteração e cancelamento**, com **tolerância a falhas** garantida pelas **DLQs**.



