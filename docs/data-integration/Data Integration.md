# Integração de Sistemas - Torcedor Plus

## 1. Ecossistema de Integrações Externas

Para operar a gestão de sócios, cobranças recorrentes e comunicação em tempo real com os torcedores, o **Torcedor Plus** integra-se a três serviços externos principais:

| Serviço | Tipo de Comunicação | Protocolo / Formato | Finalidade no Sistema |
| :--- | :--- | :--- | :--- |
| **ViaCEP** | Síncrona (Request/Response) | HTTPS / REST (JSON) | Autocompletar logradouro, bairro, cidade e UF a partir do CEP no cadastro do Sócio Torcedor. |
| **Gateway de Pagamento** *(ex: Pagar.me / Stripe)* | Assíncrona (Fila + Webhook) | HTTPS / REST (JSON) | Processamento de mensalidades recorrentes, cartões de crédito, PIX e compra de ingressos. |
| **Firebase Cloud Messaging (FCM)** | Assíncrona (Disparo de Evento) | HTTPS / API SDK (JSON) | Envio de notificações *Push* para o aplicativo mobile (confirmação de pagamento, alertas de jogos e cupons). |

---

## 2. Arquitetura Assíncrona e Mensageria (RabbitMQ)

Em dias de jogos decisivos ou abertura de vendas de ingressos, milhares de torcedores realizam transações simultaneamente. Para evitar gargalos e indisponibilidade no aplicativo, o fluxo financeiro é desacoplado utilizando **RabbitMQ**.

### Fluxo de Processamento de Pagamentos:
1. **Requisição Inicial:** O Sócio Torcedor confirma a assinatura ou compra no App Mobile.
2. **Publicação na Fila:** O Backend valida os dados básicos, publica a intenção de pagamento na fila `payments.process` do **RabbitMQ** e responde imediatamente ao App com status `202 Accepted` (*"Pagamento em processamento"*).
3. **Consumo em Background:** Um serviço *Worker* consome a mensagem da fila de forma controlada e realiza a chamada segura ao **Gateway de Pagamento**.
4. **Confirmação (Webhook):** O Gateway processa a cobrança e envia uma requisição `POST` para o endpoint de Webhook do Torcedor Plus.
5. **Efetivação e Notificação:** O Backend atualiza o status do sócio no **AWS RDS (PostgreSQL)**, invalida/atualiza o cache de acesso no **Redis** e dispara uma notificação *Push* via **Firebase** confirmando a liberação do acesso ao estádio.

---

## 3. Contratos de Dados (Payloads JSON)

Todas as integrações trafegam dados padronizados em **JSON (UTF-8)** sobre conexões criptografadas (**TLS 1.3**).

### Exemplo: Recebimento de Webhook de Pagamento Aprovado
```json
{
  "event_id": "evt_774839201",
  "event_type": "payment.approved",
  "timestamp": "2026-11-13T21:30:00Z",
  "data": {
    "torcedor_id": "usr_998877",
    "plano_id": "pln_ouro_mensal",
    "valor_centavos": 8990,
    "metodo_pagamento": "credit_card",
    "status_acesso_catraca": "LIBERADO"
  }
}
```

## 4. Resiliência, Segurança e Tratamento de Erros

- **Autenticação de Webhooks:** Todas as notificações recebidas do Gateway de Pagamento passam pela validação de assinatura criptográfica no cabeçalho HTTP (`X-Signature` / HMAC) para impedir requisições forjadas.
- **Idempotência:** Cada transação possui uma chave única (`event_id`). Caso o Gateway envie o mesmo Webhook duas vezes por instabilidade de rede, o sistema ignora a duplicidade sem gerar cobrança ou benefício duplicado.
- **Retry e Dead Letter Queue (DLQ):** Se o Gateway de Pagamento estiver temporariamente fora do ar durante o consumo da fila, o *Worker* devolve a mensagem ao RabbitMQ aplicando **Exponential Backoff** (tentativas após 5s, 30s, 2min). Após 3 falhas consecutivas, a mensagem é movida para a fila `payments.dlq` (*Dead Letter Queue*) para auditoria técnica e reprocessamento sem perda de vendas.
- **Fallback do ViaCEP:** Caso a API do ViaCEP apresente lentidão ou indisponibilidade (*timeout* configurado em 2 segundos), o formulário do aplicativo libera os campos de endereço para preenchimento manual, garantindo que o cadastro do torcedor não seja bloqueado.