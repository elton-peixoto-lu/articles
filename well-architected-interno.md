# Adoção de um Well-Architected Interno para Governança em Ambientes Multi-Cloud  
**Autor:** Elton Peixoto

---

## Resumo

Este artigo apresenta uma abordagem para implementar um **Well-Architected Interno** em ambientes **multi-conta AWS**, estendendo o [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) com políticas e automações que atendem a requisitos de negócio, regulatórios (LGPD, GDPR) e fiscais (CNPJ, entidades pagadoras). A estratégia centraliza o [Cloud Custodian](https://cloudcustodian.io/docs/) para enforcement e remediação automática de configurações obrigatórias, com suporte de SCPs, Terraform, tagging estratégico e provisionamento via AWS Organizations.

---

## 1. Introdução

Empresas que operam em ambientes multi-cloud enfrentam desafios de **governança**, **compliance**, **auditoria** e **controle de custos**. O [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) oferece diretrizes genéricas para resiliência e segurança, mas muitas vezes não captura requisitos específicos de negócio, fiscais e regulatórios de cada organização.  
Um **Well-Architected Interno** estende essas diretrizes, incorporando **mandatory configurations** e automações tailor-made, alinhando a infraestrutura à estratégia corporativa e às normas locais (LGPD, GDPR).

---

## 2. Mandatory Configurations e Tagging Estratégico

A base de um Well-Architected Interno é a **imposição de configurações obrigatórias**, especialmente o **tagging estratégico** de recursos com metadados críticos:

- `cost_center` (CNPJ ou centro financeiro)  
- `squad` (time responsável)  
- `service` (funcionalidade)  
- `region` (localização geográfica)  
- `cnpj` e `entidade_pagadora` (para auditorias fiscais)

Essa granularidade permite gerar relatórios confiáveis, rastrear despesas por entidade legal e cumprir auditorias internas e externas. Veja as [boas práticas de tagging da AWS](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html).

---

## 3. Uso de Cloud Custodian

O [Cloud Custodian](https://cloudcustodian.io/docs/) é a ferramenta central para **enforcement** de políticas como código, permitindo:

1. **Detecção** de recursos sem tags obrigatórias.  
2. **Remediação automática**, como tagging ou deleção de recursos não utilizados.  
3. **Execução via AWS Lambda**, disparadas por EventBridge.

**Exemplo de policy (YAML):**

```yaml
policies:
  - name: enforce-cost-center-tag
    resource: ec2
    filters:
      - "tag:cost_center": absent
    actions:
      - type: tag
        key: cost_center
        value: default-${account_id}
      - type: notify
        to:
          - slack
        transport:
          type: sqs
          queue: https://sqs.us-east-1.amazonaws.com/123456789012/alerts
```

Essa policy aplica automaticamente um `cost_center` padrão e notifica as equipes via SQS + Slack. Mais detalhes estão no artigo [Compliance as Code and Auto-Remediation with Cloud Custodian](https://aws.amazon.com/blogs/opensource/compliance-as-code-and-auto-remediation-with-cloud-custodian/).

---

## 4. Organizações e Baselines

O [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) permite a **implantação centralizada** de baselines de governança em todas as contas:

- **IAM Roles** definidas por código (Terraform/Pulumi) para o Cloud Custodian, garantindo permissões mínimas necessárias e rastreabilidade.  
- **Service Control Policies** (SCPs) que impedem a criação de recursos sem tags ou uso de regiões não autorizadas.  
- **Módulos reutilizáveis** para diferentes **tipos de contas** (produção, sandbox, regulamentada), alinhados a workloads (por exemplo, streaming de dados, workloads GDPR-sensitive).

---

## 5. Controle de Acesso e SCPs

As [SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) definem permissões máximas em cada OU (Organizational Unit):

- **Bloqueio de ações sensíveis** (ex.: `s3:DeleteBucket`, `ec2:TerminateInstances`)  
- **Permissões específicas** para Lambdas do Custodian via **trust relationships**, replicadas em toda a Organization  
- **Restrições finas** por tag e condição `StringEquals` para `aws:RequestTag`

---

## 6. Casos Reais

### 6.1 Capital One  
A [Capital One](https://www.capitalone.com/software/blog/cloud-governance/) desenvolveu seu próprio framework interno de governança, integrando Cloud Custodian ao ciclo de compliance e empregando SCPs e Terraform para auditar automaticamente recursos, resultando em maior agilidade e menor risco regulatório.

### 6.2 Lyft  
A [Lyft](https://aws.amazon.com/solutions/case-studies/lyft-cost-management/) utilizou AWS Cost Management e tagging consistente para reduzir em **40% o custo por viagem em 6 meses**, automatizando relatórios de custo e implementando guardrails com Cloud Custodian e AWS Lambda.

### 6.3 Zalando  
A [Zalando](https://engineering.zalando.com/posts/2022/02/transactional-outbox-with-aws-lambda-and-dynamodb.html) construiu seu data lake em Amazon S3, adotando práticas de tagging e governança para controlar custos e acelerar análises de dados, resultando em uma infraestrutura de multi-petabytes com otimizações de storage e conformidade GDPR.

---

## 7. Benefícios para o Negócio

- **Eficiência de Custos:** eliminação de recursos ociosos e otimização de storage ([CloudZero](https://www.cloudzero.com/blog/how-top-tech-brands-manage-costs/))  
- **Conformidade e Auditoria:** relatórios confiáveis para órgãos fiscais e regulatórios  
- **Escalabilidade de Governança:** replicação instantânea para novas contas/ambientes  
- **Visibilidade Centralizada:** dashboards de compliance e custo para liderança

---

## 8. Conclusão

Um **Well-Architected Interno** capacita organizações a traduzir políticas corporativas em **infraestrutura concreta** e auditável, indo além das recomendações genéricas da AWS e incorporando **automatização**, **compliance** e **estratégia de negócio** em seu DNA operacional.

---

## Referências

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)  
- [AWS Cost Allocation and Tagging Best Practices](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html)  
- [Cloud Custodian Documentation](https://cloudcustodian.io/docs/)  
- [AWS Organizations and SCPs Overview](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)  
- [Compliance as Code and Auto-Remediation with Cloud Custodian](https://aws.amazon.com/blogs/opensource/compliance-as-code-and-auto-remediation-with-cloud-custodian/)  
- [Capital One – Cloud Governance](https://www.capitalone.com/software/blog/cloud-governance/)  
- [Lyft Cost Optimization Case Study](https://aws.amazon.com/solutions/case-studies/lyft-cost-management/)  
- [Zalando Infra Blog](https://engineering.zalando.com/posts/2022/02/transactional-outbox-with-aws-lambda-and-dynamodb.html)  
- [How Top Tech Brands Manage Cloud Costs – CloudZero](https://www.cloudzero.com/blog/how-top-tech-brands-manage-costs/)