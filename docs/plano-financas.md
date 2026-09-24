# MartinTech Finanças — plano de produto e implantação

Atualizado em 24/09/2026. Aplicação publicada em `https://financas.martintech.org/` com certificado e redirecionamento HTTPS ativos.

## 1. Objetivo e limites da primeira versão

Organizar finanças pessoais e corporativas em uma única aplicação, com separação rigorosa entre espaços, contas, permissões e relatórios. Na primeira etapa, o Visor será a fonte de verdade dos dados financeiros. A aplicação não deve copiar credenciais nem publicar dados financeiros no GitHub Pages. A prova local de OAuth do Visor foi concluída para leitura; o fluxo completo da aplicação publicada ainda exige validação com o primeiro usuário.

**Critérios de sucesso da primeira entrega:** usuário autenticado; seleção de espaço pessoal ou empresarial; saldos e lançamentos conciliados com o Visor; filtros por período/conta/categoria; registro e edição com trilha de auditoria; importação assistida de CSV/OFX; orçamento e alertas; backup e exportação; uso em celular e computador. Não anunciar sincronização bancária automática antes de validar provedor, consentimento e segurança.

## 2. Arquitetura proposta

| Camada | Primeira etapa | Evolução |
| --- | --- | --- |
| Site institucional | Repositório público `MartinView/martintech-site`, GitHub Pages, `martintech.org` | Manter independente do produto financeiro |
| Aplicação web | Repositório privado separado na organização, frontend responsivo em `financas.martintech.org` | PWA e componentes compartilhados com o site, se útil |
| API | AWS Lambda + API Gateway HTTP API em `sa-east-1`, implantados | Filas para importação e rotinas longas |
| Identidade | Amazon Cognito com convite e TOTP obrigatório; autorização por espaço na API | Papéis empresariais e SSO se houver clientes |
| Dados financeiros | Visor como fonte de verdade; API própria atua como camada de autorização, validação e adaptação | Banco transacional próprio apenas se os requisitos excederem o Visor |
| Dados da aplicação | DynamoDB guarda estado OAuth e vínculo de usuários; o dado financeiro permanece no Visor | Idempotência, auditoria e configurações por espaço |
| Segredos | Token OAuth cifrado com KMS e contexto por usuário; permissões restritas à função Lambda | Rotação, segregação por ambiente e revisão periódica |
| Observabilidade | Logs sem dados financeiros sensíveis, métricas, alarmes e limite de custo | Tracing e painéis operacionais |
| CI/CD | GitHub Actions valida código e publica o frontend no Pages | OIDC para deploy AWS, revisão por pull request e ambientes protegidos |

**Ponto decisivo:** a API já implementa OAuth individual com PKCE, estado de uso único, renovação de token e ferramentas de leitura permitidas. A prova local validou OAuth de leitura, e a API implantada respondeu ao health check e rejeitou chamadas privadas sem JWT. Ainda faltam testes reais do fluxo de ponta a ponta, isolamento entre usuários, falhas de renovação e limites do Visor. Escritas permanecem desabilitadas até haver auditoria e idempotência.

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
- Substituir o login AWS root usado no primeiro deploy por identidade administrativa e de deploy com privilégio mínimo antes da operação recorrente.

## 5. Entrega por fases

| Fase | Entregáveis verificáveis | Condição para avançar |
| --- | --- | --- |
| 0. Fundação | Site institucional e aplicativo em HTTPS; repositório privado do app, DNS do subdomínio, orçamento e pilha AWS implantados | Primeiro usuário e teste de login completo |
| 1. Prova Visor | OAuth local de leitura concluído; API de leitura implantada com autorização individual | Consentimento, renovação e isolamento reproduzidos na aplicação publicada |
| 2. MVP | Login, espaços, contas, transações, categorias, filtros, importação assistida, dashboard, auditoria | Testes de isolamento, conciliação e recuperação aprovados |
| 3. Planejamento | Orçamentos, recorrências, metas, alertas, caixa previsto | Cálculos reconciliados com transações |
| 4. Empresa | Convites, papéis, centro de custo, contas a pagar/receber, relatórios gerenciais e aprovações | Permissões e regras contábeis homologadas |
| 5. Operação | Monitoramento, backups testados, documentação, suporte, política de privacidade, publicação | Segurança, custos e continuidade aprovados |

## 6. Custos e escolhas de plataforma

- Em 23/09/2026 a conta AWS consultada estava em plano **Free**, ativa, com **US$ 100 de créditos restantes** e fim do plano em **24/03/2027 01:34 UTC**. O painel também exibia 0/5 atividades para conquistar até mais US$ 100; esses créditos adicionais ainda não foram ganhos. Não havia buckets S3, zonas Route 53, instâncias EC2, funções Lambda ou bancos RDS nas regiões consultadas. Cost Explorer respondeu que a conta não está habilitada para consulta.
- Foi criado o orçamento AWS `MartinTech-Financas-Dev-Monthly` de **US$ 10/mês**, com créditos excluídos do cálculo e alertas de gasto real em 50%, 80% e 100%, além de previsão em 100%, enviados a `contato@martintech.org`. Alertas são informativos e não interrompem recursos automaticamente.
- O plano Free termina após seis meses ou consumo dos créditos; ao expirar, a conta é fechada se não for migrada para Paid. Portanto, ele serve para prova de conceito e MVP controlado, não para prometer continuidade comercial sem decisão de cobrança. Fonte: [AWS Choosing a plan](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html).
- Começar com funções sob demanda evita a despesa fixa de EC2 e RDS. Estimar custo real com tráfego, logs, chamadas API, segredos e região antes do deploy.
- GitHub Enterprise Cloud oferece Pages, Actions, regras de repositório e recursos de governança; verificar a franquia e licenças efetivas da organização antes de assumir minutos ou recursos avançados. Fontes: [GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud), [GitHub Pages](https://docs.github.com/en/enterprise-cloud@latest/pages/getting-started-with-github-pages/what-is-github-pages).
- Databricks Free Edition fica fora da operação comercial: seus termos a destinam a uso não comercial, sem SLA, com cotas e apps que param após até 24 horas. Pode servir para aprendizado com dados sintéticos. Fonte: [Databricks Free Edition limitations](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations).

## 7. Próximas decisões

1. Convidar o primeiro administrador, cadastrar TOTP e validar consentimento e renovação do Visor.
2. Criar identidade AWS de privilégio mínimo e pipeline controlado de deploy do backend.
3. Homologar acesso entre espaços e usuários, importação, escritas com auditoria e os módulos empresariais.
4. Definir política de cobrança e continuidade antes de 24/03/2027.
