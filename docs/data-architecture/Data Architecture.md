

A arquitetura de dados do TorcePlus foi redesenhada para unificar as funcionalidades transacionais (Core) e analíticas (Insights) em uma única plataforma integrada para os clubes.

### A. Sistemas Envolvidos

1. **TorcePlus App (Mobile/Web):** Interface do torcedor para cadastro, pagamento de assinaturas e resgate de descontos e benefícios.
2. **TorcePlus Platform (Web Admin):** A união do antigo Core com o Insights. É o portal onde a diretoria do clube gerencia os planos de sócio e também visualiza os dashboards analíticos de engajamento e inadimplência no mesmo ambiente.
3. **TorcePlus API (Backend):** O motor central das regras de negócio.
4. **Banco de Dados Operacional (Transacional):** AWS RDS (ex.: PostgreSQL) para armazenar os dados do dia a dia.
5. **Storage de Arquivos:** AWS S3 para armazenar documentos de identificação, termos e assets do clube (escudos e banners).
6. **Data Warehouse (Analítico):** AWS Redshift para processar grandes volumes de dados históricos e alimentar o módulo de Insights da plataforma.
7. **Integração Externa:** *Gateway de Pagamento* (ex.: Pagar.me ou Stripe) para o processamento das cobranças recorrentes.

### B. Fluxo de Dados

1. O clube configura seus dados institucionais, contas bancárias e planos de sócio diretamente na **TorcePlus Platform**.
2. O torcedor acessa o **TorcePlus App**, escolhe um plano e insere seus dados. A **API** salva as informações no **Banco Operacional**.
3. No momento da assinatura, os dados do cartão vão para o **Gateway de Pagamento**, que devolve apenas um *token* de segurança.
4. Quando o torcedor utiliza um desconto em uma loja parceira (uso de benefício), essa ação é registrada na **API**.
5. Diariamente, um processo de ETL (Extração, Transformação e Carga) leva os dados do **Banco Operacional** para o **Data Warehouse**, realizando a limpeza dos dados sensíveis (anonimização).
6. O módulo de Insights, dentro da **TorcePlus Platform**, consome os dados do **Data Warehouse** para exibir os gráficos de receita e churn para os gestores do clube, sem deixar o sistema principal lento.

### C. Transacional vs. Analítico

- **Banco Operacional (Transacional / OLTP):** Focado no tempo de resposta rápido para o torcedor (ex.: login, pagamento e atualização de cadastro). Trabalha com dados normalizados para garantir a integridade das cobranças mês a mês.

- **Data Warehouse (Analítico / OLAP):** Focado na leitura rápida de grandes volumes de dados. Ele responde às perguntas do módulo Insights: "Qual a taxa de cancelamento de planos familiares neste semestre?" ou "Qual a previsão de receita para o mês que vem?".

## D. Integração das Entregas

A arquitetura final interliga os conceitos de forma estruturada:

1. **Data Integration:** Os clientes acessam via HTTPS/TLS. Webhooks e sistemas externos comunicam-se por meio de APIs REST e RabbitMQ.
2. **Data Security:** O API Gateway realiza o Rate Limiting e valida os tokens JWT (RBAC), repassando as requisições seguras para os microsserviços.
3. **Data Architecture & Modeling:** Os serviços processam as regras de negócio e interagem com o PostgreSQL (OLTP), modelado em 3FN, com o Row-Level Security ativado para isolar inquilinos/clientes.
4. **Data Storage:** O PostgreSQL cuida da persistência relacional; o Redis gerencia o cache e as sessões em milissegundos; e o S3 armazena documentos e backups. Os dados transacionais são exportados de forma assíncrona para a Read Replica (OLAP) e o Data Lake para fins analíticos.