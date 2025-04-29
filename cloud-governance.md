<center>
**ADOÇÃO DE UM WELL-ARCHITECTED INTERNO PARA GOVERNANÇA EM AMBIENTES MULTI-CLOUD**
</center>

<p style="text-align: right;">Elton Peixoto¹</p>

**RESUMO**

Este artigo explora como a adoção de um Well-Architected Interno — uma extensão personalizada do AWS Well-Architected Framework — pode trazer ganhos substanciais de governança, auditoria, alinhamento regulatório e eficiência de custos em ambientes multi-cloud. A proposta fundamenta-se em práticas como o uso de Cloud Custodian, Terraform, SCPs (Service Control Policies) e provisionamento automatizado de contas via baselines específicas, discutindo casos de uso reais e benefícios aferidos por empresas do setor.

*(Nota: Palavras-chave geralmente são incluídas após o resumo, separadas por ponto.)*

---

**1 INTRODUÇÃO**

Empresas que operam em ambientes multi-cloud enfrentam desafios crescentes relacionados à governança, compliance, auditoria e custo. O AWS Well-Architected Framework provê diretrizes úteis, mas genéricas. Um Well-Architected Interno permite incorporar regras, controles e automações específicas, alinhando a operação de infraestrutura à estratégia de negócio, a requisitos fiscais, jurídicos e a regulações locais, como LGPD e GDPR.

**2 DESENVOLVIMENTO** *(Nota: A ABNT não exige explicitamente um título "Desenvolvimento", mas agrupa as seções subsequentes sob ele. Mantive para clareza estrutural, mas poderia ser omitido, iniciando diretamente em 2.1)*

**2.1 Mandatory Configurations e Tagging Estratégico**

O uso de mandatory configurations garante que nenhum recurso seja criado sem tags obrigatórias como `cost_center`, `squad`, `service`, `region`, `cnpj` ou `entidade_pagadora`. Isso permite auditorias que cruzam informações técnicas com estruturas financeiras e regulatórias. Por exemplo, é possível verificar se um bucket pertence ao centro de custo relacionado a uma filial no Brasil, vinculando o uso ao CNPJ correto para fins fiscais.

**2.2 Uso de Cloud Custodian**

O Cloud Custodian permite automatizar e reforçar políticas como:

* Deleção automática de EBS volumes não utilizados.
* Notificações para recursos com tags ausentes.
* Restrições geográficas para uso de serviços sensíveis.
* Adoção de políticas de desligamento de recursos fora do horário comercial.

Essas funções podem ser executadas por Lambdas com permissões definidas via Terraform, utilizando roles seguras distribuídas por uma Organization.

**2.3 Organizações e Baselines**

Por meio do AWS Organizations, é possível instalar uma baseline de governança automaticamente:

* Com roles mínimas para execução de automações.
* Com SCPs específicas para diferentes tipos de conta (ex: produção, sandbox, regulamentada).
* Com Terraform/Pulumi para declarar a infraestrutura como código.

As contas são preparadas com perfis para workloads específicos (por exemplo: dados sensíveis, serviços de streaming, geografias com restrições legais), e recebem apenas os recursos que fazem sentido para o seu contexto de negócio.

**2.4 Controle de Acesso e SCPs**

As SCPs podem garantir:

* Que serviços sensíveis (como Kinesis ou S3 público) só sejam usados em contas específicas.
* Que a Cloud Custodian Lambda tenha acesso permitido via trust relationship central, replicada em toda a Organization.
* Que não haja permissões genéricas em roles sensíveis.

O uso de provisionadores como o Terraform garante rastreabilidade e versionamento desses controles.

**2.5 Casos Reais**

* **Capital One:** Criadora do Cloud Custodian, integrou-o a seu ciclo de compliance, automatizando o enforcement de políticas e reduzindo riscos legais.
* **Lyft:** Reduziu gastos com recursos ociosos, economizando milhões por meio de Lambdas automatizadas com Custodian.
* **Zalando:** Criou uma plataforma de governança automatizada multi-cloud para atender ao GDPR e à LGPD, alinhando sua infraestrutura ao negócio.

**2.6 Benefícios para o Negócio**

* **Eficiência de custos:** eliminação de desperdício com recursos não utilizados.
* **Governança e Compliance:** aderência contínua a políticas internas e externas.
* **Auditabilidade:** geração de relatórios confiáveis para órgãos reguladores.
* **Padronização:** deploys consistentes para qualquer tipo de workload.
* **Escalabilidade de Governança:** fácil replicação para novas contas/ambientes.

**3 CONCLUSÃO**

Criar um Well-Architected Interno é sair da dependência de recomendações genéricas e construir um framework vivo, adaptável e fortemente conectado à realidade da organização. Isso empodera engenheiros e líderes técnicos a operarem como parte estratégica do negócio, traduzindo políticas corporativas em infraestrutura concreta, eficiente e auditável.

---

**REFERÊNCIAS**

AMAZON WEB SERVICES. **AWS Organizations**. Disponível em: https://aws.amazon.com/organizations/. Acesso em: 29 abr. 2025.

CAPITAL ONE. **Cloud Governance at Capital One**: Leveraging AWS Native Services and Open Source Tools. Capital One Tech Blog. Disponível em: https://www.capitalone.com/tech/cloud/aws-cloud-governance/. Acesso em: 29 abr. 2025.

CLOUD CUSTODIAN. **Cloud Custodian**. GitHub. Disponível em: https://github.com/cloud-custodian/cloud-custodian. Acesso em: 29 abr. 2025.

LYFT ENGINEERING. **How Lyft saves multi-million dollars on cloud cost by using automated resource hibernation**. Lyft Blog. Disponível em: https://www.lyft.com/blog/posts/how-lyft-saves-cloud-cost. Acesso em: 29 abr. 2025.

ZALANDO ENGINEERING BLOG. **Zalando Technology Blog**. Disponível em: https://www.zalando.com/company/technology/blog/. Acesso em: 29 abr. 2025. *(Nota: Idealmente, a referência seria a um post específico sobre governança, se disponível).*

---
¹ *Nota: Informações como afiliação institucional e e-mail do autor geralmente são incluídas aqui ou em nota de rodapé.*

