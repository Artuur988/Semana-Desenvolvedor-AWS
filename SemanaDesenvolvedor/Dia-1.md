# 📦 Arquitetura Orientada a Eventos — Ingestão de Pedidos (AWS)

Projeto desenvolvido na **Semana do Desenvolvedor AWS (Escola da Nuvem)** – **Dia 1: Ingestão de Pedidos via API e EventBridge**. Este módulo implementa um fluxo de entrada de pedidos usando **API Gateway → Lambda (pré-validação) → SQS FIFO (+ DLQ) → Lambda (validação) → EventBridge (Custom Event Bus)**.



---

## 🎯 Objetivo

Criar um endpoint REST que receba pedidos, faça uma pré-validação, enfileire para processamento ordenado e, após validação, publique um **evento** em um **Event Bus** do **Amazon EventBridge**. Ao final, você terá:  
API funcional, 2 Lambdas, fila **SQS FIFO** (+ **DLQ**) e **Event Bus** customizado.

---

## 🗺️ Arquitetura (visão geral do fluxo)

**Cliente → API Gateway → Lambda (pré-validação) → SQS FIFO → Lambda (validação) → EventBridge (Custom Event Bus)**  
O uso de **FIFO** garante **ordem**; **DLQ** aumenta a resiliência; **EventBridge** desacopla produtores/consumidores.

---

## 🛠️ Passo a Passo Implementado

### 1) IAM Roles (permissões)

- **`lambda-prevalidacao-role-seu-nome`**
  - Permissões: **CloudWatch Logs** + **enviar mensagens** para SQS principal.
- **`lambda-validacao-pedidos-role-seu-nome`**
  - Permissões: **receber/deletar** mensagens da SQS + **`events:PutEvents`** no EventBridge.

> Dica: use nomes com sufixo **`-seu-nome`** e **trabalhe sempre na mesma região** para evitar conflitos.

---

### 2) Filas (Amazon SQS FIFO + DLQ)

1. **Crie a DLQ FIFO**: `pedidos-fifo-dlq-seu-nome.fifo`.
2. **Crie a fila principal FIFO**: `pedidos-fifo-queue-seu-nome.fifo`.
   - **DLQ habilitada** com **Maximum receives = 3**.
   - **Anote ARN e URL** da fila (usados nas Lambdas/Policies).

---

### 3) Lambda de **Pré-validação**

- **Função**: `pre-validacao-lambda-seu-nome` (Python).
- **Variável de ambiente**: `SQS_QUEUE_URL` (URL da fila FIFO).
- **Papel**: validar payload básico e **enfileirar** na SQS.

---

### 4) API (Amazon API Gateway REST)

- **API**: `pedidos-api-seu-nome` → **`/pedidos`** (método **POST**), integração **Lambda Proxy** com a pré-validação.
- **Deploy** no **stage `dev`** e **anote a Invoke URL** (ex.: `.../dev`).

**Exemplo de requisição:**

```bash
curl -X POST https://<invoke-url>/dev/pedidos \
  -H "Content-Type: application/json" \
  -d '{
        "pedidoId": "lab001-artur",
        "clienteId": "cliente123-artur",
        "itens": [
          {"produto": "Caneta Azul", "quantidade": 10},
          {"produto": "Caderno Universitário", "quantidade": 2}
        ]
      }'
```
### 5) EventBridge (Custom Event Bus)

- Crie o **Event Bus**: `pedidos-event-bus-seu-nome`.  
- Anote o **ARN** (usado na policy da Lambda de validação).  

---

### 6) Lambda de Validação de Pedidos

- **Função**: `validacao-pedidos-lambda-seu-nome` (Python 3.12).  
- **Variável de ambiente**: `EVENT_BUS_NAME = pedidos-event-bus-seu-nome`.  
- **Trigger**: SQS (fila principal) com **Batch size = 1** (mantém ordem FIFO).  
- **Papel**: consumir SQS, validar e publicar evento no **EventBridge**.  

---

### 7) Teste do fluxo ponta-a-ponta

1. **Envie um pedido** pelo endpoint `/pedidos` (curl acima).  
2. Verifique **CloudWatch Logs** da `pre-validacao-lambda`  
   - mensagens: *“Evento recebido do API Gateway”* / *“Mensagem enviada para SQS”*.  
3. Veja a **mensagem na SQS** (*Send and receive messages → Poll*).  
4. A `validacao-pedidos-lambda` deve **consumir, validar e publicar** no **EventBridge**.  
5. Em **EventBridge → Event buses → Monitoring**, acompanhe:  
   - **PutEvents_Invocations**  
   - **MatchedEvents**  
   *(os gráficos podem levar alguns minutos).*  

---

## ✅ Resultado

- **API REST** pública para ingestão de pedidos.  
- **Processamento assíncrono e ordenado** com **SQS FIFO + DLQ (3 tentativas)**.  
- **Lambdas desacopladas** (pré-validação e validação).  
- **Eventos publicados** em um **Event Bus customizado** no **EventBridge**.  

---

## 🔜 Próximos passos (Aula 2)

- Adicionar **ingestão de arquivos via Amazon S3**.  
- Criar uma **regra do EventBridge** que encaminhe eventos para novos alvos  
  (ex.: outra fila SQS) para observar eventos **em tempo real**.  

