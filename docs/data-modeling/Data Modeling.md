

A arquitetura e a modelagem propostas (com a unificação do Core e do Insights) atacam diretamente os objetivos estratégicos levantados pela TorcePlus na OAT 1:

1. **Redução da Inadimplência (15%):** A separação entre o Banco Operacional (RDS) e o Data Warehouse (Redshift) permite criar dashboards dinâmicos dentro da plataforma sem deixar o app do torcedor lento. O setor Financeiro consegue visualizar imediatamente pela tabela `Fato_Faturamento` quais torcedores estão com o cartão prestes a vencer ou com pagamentos atrasados, permitindo ações de cobrança proativas.

2. **Conformidade total com a LGPD:** Ao modelarmos o Data Warehouse, aplicamos a *anonimização* (a tabela `Dim_Torcedor_Anonimizado` guarda apenas faixa etária e cidade, deixando de fora Nome e CPF). Isso permite que o marketing do clube faça análises de comportamento e campanhas sem expor dados sensíveis. Além disso, a gestão estruturada dos termos físicos protege legalmente a relação com menores de idade.

3. **Diminuir o tempo de Onboarding (de 30 para 10 dias):** A modelagem transacional centralizada usando chaves estrangeiras (onde as tabelas de conta bancária, planos e torcedores apontam para o `clube_id`) cria uma arquitetura *Multi-tenant* (vários inquilinos no mesmo banco). O clube entra na base principal instantaneamente, bastando configurar seus dados bancários, eliminando a necessidade de criar um banco de dados isolado para cada novo cliente.

4. **Fechar contrato com 20 novos clubes:** Ao removermos integrações complexas (como a camada de catracas físicas) nesse momento, o sistema fica mais enxuto, estável e focado na dor principal dos clubes menores: gestão e monetização da base. Entregar o "Core" junto com "Insights" em uma única ferramenta web facilita as vendas, pois o clube não precisa contratar um sistema de BI à parte.


# Modelagem Fora do Digital

Os clubes geralmente possuem planos familiares que incluem menores de idade. Para cumprir a LGPD rigorosamente (um dos focos do sistema), os clubes precisam armazenar o **Termo de Consentimento dos Pais**, que muitas vezes é um documento físico.

* **Tipo de documento:** Físico Legal (Formulário em papel impresso, assinado à mão).

* **Modelagem do Documento (Campos e Estrutura):** 
  - Cabeçalho com logo, Razão Social e CNPJ do Clube.
  - Dados do Torcedor Dependente (Nome completo e Data de Nascimento do menor).
  - Dados do Responsável Legal (Nome do Torcedor Titular, CPF e Grau de Parentesco).
  - Texto legal de consentimento explícito para tratamento de dados pelo clube e pela TorcePlus.
  - Local, Data e Assinatura do responsável.

* **Forma de organização:** Ordem alfabética pelo nome do responsável, em pastas sanfonadas organizadas por "Ano de Associação".

* **Local de armazenamento:** Armários deslizantes de aço, localizados na sala administrativa/secretaria do clube.

* **Identificação e classificação:** Etiquetas adesivas na lombada de cada pasta (Ex: "Termos Menores - 2026 - Letras A até C").

* **Controle de acesso:** A sala fica trancada e as chaves ficam exclusivamente com a gerência administrativa. Apenas funcionários autorizados da secretaria do clube podem manusear o arquivo.

* **Tempo de retenção:** Arquivados até o dependente menor completar 18 anos, acrescidos de 5 anos de resguardo legal para eventuais auditorias.

* **Processo de arquivamento:** 
  1. O responsável entrega o termo assinado presencialmente no clube.
  2. O atendente confere a assinatura com um documento oficial com foto.
  3. O termo recebe um carimbo de "Recebido".
  4. O documento é perfurado e arquivado na pasta correspondente.
  5. O atendente atualiza na **TorcePlus Platform** que o termo físico foi "Entregue e Validado".

* **Forma de recuperação:** Quando necessário (ex: solicitação do titular ou auditoria), o funcionário consulta o sistema web para ver o ano da associação, vai até a sala de arquivo e busca manualmente na pasta alfabética.

* **Riscos envolvidos:** 
  - *Dano:* Umidade ou incêndio na secretaria (mitigado pelo uso de armários de aço).
  - *Perda:* Arquivamento em pasta alfabética errada (erro humano), dificultando a recuperação do termo.
  - *Acesso indevido:* Vazamento da assinatura e CPF caso a sala seja invadida por pessoas não autorizadas.
