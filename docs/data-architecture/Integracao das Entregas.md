# Integração das Entregas

A arquitetura final interliga os conceitos de forma estruturada:

1. **Data Integration:** Os clientes acessam via HTTPS/TLS. Webhooks e sistemas externos comunicam-se por meio de APIs REST e RabbitMQ.
2. **Data Security:** O API Gateway realiza o Rate Limiting e valida os tokens JWT (RBAC), repassando as requisições seguras para os microsserviços.
3. **Data Architecture & Modeling:** Os serviços processam as regras de negócio e interagem com o PostgreSQL (OLTP), modelado em 3FN, com o Row-Level Security ativado para isolar inquilinos/clientes.
4. **Data Storage:** O PostgreSQL cuida da persistência relacional; o Redis gerencia o cache e as sessões em milissegundos; e o S3 armazena documentos e backups. Os dados transacionais são exportados de forma assíncrona para a Read Replica (OLAP) e o Data Lake para fins analíticos.