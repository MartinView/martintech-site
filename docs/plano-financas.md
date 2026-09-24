# MartinTech Finanças — plano de produto e implantação

Atualizado em 23/09/2026. Produto previsto em `financas.martintech.org`.

## 1. Objetivo e limites da primeira versão

Organizar finanças pessoais e corporativas em uma única aplicação, com separação rigorosa entre espaços, contas, permissões e relatórios. Na primeira etapa, o Visor será a fonte de verdade dos dados financeiros. A aplicação não deve copiar credenciais nem publicar dados financeiros no GitHub Pages. A integração automática com o Visor depende de uma prova técnica de autenticação e operações de leitura/escrita fora de uma sessão interativa MCP.

**Critérios de sucesso da primeira entrega:** usuário autenticado; seleção de espaço pessoal ou empresarial; saldos e lançamentos conciliados com o Visor; filtros por período/conta/categoria; registro e edição com trilha de auditoria; importação assistida de CSV/OFX; orçamento e alertas; backup e exportação; uso em celular e computador. Não anunciar sincronização bancária automática antes de validar provedor, consentimento e segurança.

## 2. Arquitetura proposta

| Camada | Primeira etapa | Evolução |
| --- | --- | --- |
| Site institucional | Repositório público `MartinView/martintech-site`, GitHub Pages, `martintech.org` | Manter independente do produto financeiro |
| Aplicação web | Repositório privado separado na organização, frontend responsivo em `financas.martintech.org` | PWA e componentes compartilhados com o site, se útil |
| API | AWS Lambda + API Gateway HTTP API em `sa-east-1`, após prova da integração Visor | Filas para importação e rotinas longas |
| Identidade | Serviço de autenticação apropriado à conta AWS, MFA para administradores e autorização por espaço | SSO empresarial se houver clientes e exigência comercial |
| Dados financeiros | Visor como fonte de verdade; API própria atua como camada de autorização, validação e adaptação | Banco transacional próprio apenas se os requisitos excederem o Visor |
| Dados da aplicação | Configurações, vínculos de usuários, idempotência e auditoria em armazenamento AWS de baixo custo, após modelagem | Separar armazenamento operacional de análises |
| Segredos | AWS Secrets Manager e permissões mínimas | Rotação e segregação por ambiente |
| Observabilidade | Logs sem dados financeiros sensíveis, métricas, alarmes e limite de custo | Tracing e painéis operacionais |
| CI/CD | GitHub Actions com OIDC para AWS, revisão por pull request e ambientes protegidos | Deploy automático de homologação e promoção controlada |

**Ponto decisivo:** o MCP do Visor já funciona para o agente em sessão interativa, mas isso não comprova que um backend Lambda possa manter uma autorização OAuth adequada para vários usuários. Antes de criar a API pública, provar fluxos de token, escopos, renovação, limites, operações de escrita, paginação e isolamento por usuário. Se não houver integração de servidor adequada, começar com importação/exportação assistida e reconsiderar a fonte de verdade.

## 3. Modelo funcional

1. **Identidade e espaços:** cadastro, login, MFA, convite, papéis de proprietário/admin/editor/leitor, espaço pessoal e empresas isolados, aprovação de operações sensíveis.
2. **Contas e lançamentos:** saldos, receitas, despesas, transferências, cartões, parcelas, recorrências, anexos, tags, categorias, rateios, centro de custo e histórico de alterações.
3. **Planejamento:** orçamentos, metas, fluxo de caixa previsto, contas a pagar/receber, alertas de vencimento e desvios.
4. **Conciliação:** importação CSV/OFX com prévia, detecção de duplicatas, correspondência com lançamentos e resolução manual de diferenças.
5. **Relatórios:** visão pessoal de patrimônio e orçamento; visão empresarial de caixa, contas a receber/pagar, despesas por centro de custo e resultado gerencial. DRE e impostos exigem regras contábeis definidas e validação profissional.
6. **Governança:** exportação dos dados, exclusão conforme política de retenção, consentimentos, trilha de auditoria, backup e recuperação testada.

Entidades lógicas: usuário, espaço, membro, conta, transação, transferência, categoria, orçamento, meta, recorrência, contraparte, documento, importação, conciliação e evento de auditoria. Toda leitura e escrita deve ser autorizada pelo `space_id`, inclusive relatórios e anexos. Valores monetários em unidades inteiras menores ou decimal exato; moeda e fuso explícitos.

## 4. Segurança e privacidade

- Nenhum segredo, token OAuth, extrato, anexo ou dado pessoal real em repositórios, Actions logs ou frontend estático.
- MFA para administradores; sessões curtas; permissões mínimas; criptografia em trânsito e em repouso.
- Consentimento claro para cada integração; segregação de dados pessoais e empresariais; política de retenção e exportação.
- Trilha de auditoria imutável para alterações financeiras; idempotência em criação/importação; testes de autorização entre espaços.
- Revisão da LGPD, contratos, papéis de controlador/operador e necessidade de assessoria contábil/jurídica antes da oferta comercial.
- Substituir o login AWS root usado apenas na descoberta inicial por identidade administrativa e de deploy com privilégio mínimo antes de criar recursos.

## 5. Entrega por fases

| Fase | Entregáveis verificáveis | Condição para avançar |
| --- | --- | --- |
| 0. Fundação | Repositório institucional na organização, domínio e HTTPS funcionando; repositório privado do app; custos e acesso AWS revisados | Site estável, proprietários e ambientes definidos |
| 1. Prova Visor | Contrato de integração, teste de OAuth de servidor, leitura/escrita, escopos, limites, erros e reconciliação | Operações reproduzíveis, sem sessão manual do agente |
| 2. MVP | Login, espaços, contas, transações, categorias, filtros, importação assistida, dashboard, auditoria | Testes de isolamento, conciliação e recuperação aprovados |
| 3. Planejamento | Orçamentos, recorrências, metas, alertas, caixa previsto | Cálculos reconciliados com transações |
| 4. Empresa | Convites, papéis, centro de custo, contas a pagar/receber, relatórios gerenciais e aprovações | Permissões e regras contábeis homologadas |
| 5. Operação | Monitoramento, backups testados, documentação, suporte, política de privacidade, publicação | Segurança, custos e continuidade aprovados |

## 6. Custos e escolhas de plataforma

- Em 23/09/2026 a conta AWS consultada estava em plano **Free**, ativa, com **US$ 100 de créditos restantes** e fim do plano em **24/03/2027 01:34 UTC**. O painel também exibia 0/5 atividades para conquistar até mais US$ 100; esses créditos adicionais ainda não foram ganhos. Não havia buckets S3, zonas Route 53, instâncias EC2, funções Lambda ou bancos RDS nas regiões consultadas; não havia orçamento criado. Cost Explorer respondeu que a conta não está habilitada para consulta.
- O plano Free termina após seis meses ou consumo dos créditos; ao expirar, a conta é fechada se não for migrada para Paid. Portanto, ele serve para prova de conceito e MVP controlado, não para prometer continuidade comercial sem decisão de cobrança. Fonte: [AWS Choosing a plan](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html).
- Começar com funções sob demanda evita a despesa fixa de EC2 e RDS. Estimar custo real com tráfego, logs, chamadas API, segredos e região antes do deploy. Configurar orçamento e alertas quando existir um destinatário confirmado.
- GitHub Enterprise Cloud oferece Pages, Actions, regras de repositório e recursos de governança; verificar a franquia e licenças efetivas da organização antes de assumir minutos ou recursos avançados. Fontes: [GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud), [GitHub Pages](https://docs.github.com/en/enterprise-cloud@latest/pages/getting-started-with-github-pages/what-is-github-pages).
- Databricks Free Edition fica fora da operação comercial: seus termos a destinam a uso não comercial, sem SLA, com cotas e apps que param após até 24 horas. Pode servir para aprendizado com dados sintéticos. Fonte: [Databricks Free Edition limitations](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations).

## 7. Próximas decisões

1. Confirmar o uso de `financas.martintech.org` e a identidade visual herdada do site MartinTech.
2. Validar o método de integração de servidor do Visor e a titularidade dos dados em cada espaço.
3. Criar identidade AWS de privilégio mínimo, orçamento/alertas e repositório privado do app.
4. Construir protótipo navegável e contrato de dados antes de provisionar infraestrutura permanente.
5. Definir política de cobrança e continuidade antes de 24/03/2027.
