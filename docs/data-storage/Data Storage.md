# Estratégia de Data Storage - Torcedor Plus

## A. Armazenamento e Tecnologias

### 1. Tecnologias Utilizadas
* **Banco Operacional (OLTP):** AWS RDS (PostgreSQL).
* **Cache e Sessões:** Redis (In-Memory).
* **Armazenamento de Arquivos:** Amazon S3.
* **Ambiente Analítico (OLAP):** AWS Redshift.

### 2. Justificativas das Escolhas
* **AWS RDS (PostgreSQL):** Conformidade ACID estrita, integridade relacional e suporte a RLS (Row-Level Security) para separar dados sensíveis.
* **Redis:** Usado especificamente para guardar tokens de sessão e gerar o QR Code de acesso ao estádio em milissegundos, evitando sobrecarga no banco principal em dias de jogo.
* **AWS Redshift:** Um Data Warehouse colunar focado em processar milhões de registros rapidamente para relatórios do clube.
* **Amazon S3:** Para guardar imagens de perfil dos torcedores e banners das campanhas do clube.

### 3. Volume e Escalabilidade
* **Volume Inicial:** Cadastros dos sócios torcedores e histórico de compras. O RDS suporta escala vertical (aumento de máquina).
* Em caso de picos (ex: final de campeonato), o sistema utiliza o Redis para absorver a carga de leitura de QR Codes e réplicas de leitura no RDS.

## B. Persistência
* **Persistentes:** Perfil do Sócio Torcedor, mensalidades pagas e histórico de acesso (salvos no AWS RDS).
* **Temporários:** Sessão do app e QR Code dinâmico do ingresso (salvos no Redis com expiração/TTL de 15 minutos).
* A exclusão de usuários segue a LGPD via *Soft Delete* (apenas marcando `deleted_at`), apagando dados sensíveis mas mantendo o ID para estatísticas.

## C. Backup e Recuperação
* **Estratégia:** Snapshot diário automatizado pelo AWS RDS + retenção de logs WAL para Point-in-Time Recovery (PITR).
* **Retenção:** Snapshots mantidos por 30 dias em bucket S3 geograficamente separado.

## D. Transacional vs Analítico
* **Operacional (AWS RDS):** Onde as compras e check-ins no estádio acontecem em tempo real.
* **Analítico (AWS Redshift):** Recebe os dados de forma desnormalizada durante a madrugada (ETL). 
* **Regra de Privacidade (Modelagem):** Dados sensíveis como CPF, nome exato e cartão de crédito **NÃO** são enviados para o Redshift. Apenas os IDs dos torcedores, idades, planos e frequência nos jogos são migrados para gerar inteligência de negócio sem ferir a LGPD.