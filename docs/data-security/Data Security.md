# Segurança de Dados - Torcedor Plus

## A. Classificação dos Dados
* **Públicos:** Planos de Sócio Torcedor disponíveis, valores e benefícios.
* **Internos:** Métricas de vendas de ingressos, acessos ao estádio.
* **Confidenciais:** Contratos do clube com gateways de pagamento.
* **Sensíveis (LGPD):** CPF do torcedor, dados de pagamento, e-mail e senha.

## B. Controle de Acesso (RBAC)

O sistema utiliza o padrão **RBAC** (Role-Based Access Control - Controle de Acesso Baseado em Perfis) para garantir que cada usuário veja apenas o que deve:

* **Administrador (Clube):** Acesso a relatórios, campanhas e gestão de sócios.
* **Operador (Catraca/Guichê):** Apenas permissão para validar o QR Code do torcedor no estádio.
* **Cliente (Sócio Torcedor):** Tem acesso **exclusivo** aos próprios dados e histórico de pagamentos (Garantido pela política de RLS - Row-Level Security direto no banco de dados).
* **Sistema Externo:** O Gateway de pagamento tem acesso apenas aos endpoints de Webhook via tokens específicos.

## C. Autenticação e Criptografia
* **Login:** Via JWT (JSON Web Tokens). 
* **Proteção de Senhas:** As senhas dos torcedores não são salvas em texto. Utilizamos o algoritmo de hash **Argon2id** (ou Bcrypt). Se o banco vazar, as senhas continuam indecifráveis, protegendo torcedores que repetem a mesma senha em vários sites.

## D. Proteção da Infraestrutura
* **Em Trânsito:** Toda comunicação do App Torcedor Plus com o backend usa HTTPS / TLS 1.3.
* **ORM:** Para evitar injeções de SQL (SQLi) diretas no banco AWS RDS, utilizamos o **Prisma ORM**, que mapeia os objetos de código para SQL de forma blindada e parametrizada.

## E. Riscos e Mitigações
1. **Injeção SQL no App:** (Alto) Mitigado pelo uso de Prisma ORM.
2. **Vazamento de Senha:** (Alto) Mitigado pelo uso de Hash Argon2id.
3. **Roubo de Sessão (MitM):** (Alto) Mitigado por HTTPS obrigatório.
4. **Perda de Dados do Clube:** (Alto) Mitigado por Snapshots diários no AWS RDS.
5. **Acesso Indevido:** (Médio) Mitigado por RBAC e RLS.