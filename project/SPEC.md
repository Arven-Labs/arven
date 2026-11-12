# Plataforma da organização — site mãe, conta e cobrança

Documento de design para o repositório central da organização. Cobre como os projetos se organizam (serviços, repos, sites) e como conta, planos e pagamentos ficam centralizados.

## Resumo da arquitetura

- **Um site mãe** no domínio da organização, com backend próprio, responsável por **conta, planos, cobrança e entitlements**.
- **Um subdomínio por projeto**, cada um com um site estático de vitrine, instalação e documentação. Sem pagamentos nem contas nesses sites.
- **Backends dos produtos independentes**: cada projeto com seu próprio serviço e banco, falando com o site mãe apenas por API.
- O site mãe redireciona para os sites dos produtos e vice-versa quando necessário (ex.: botão "assinar" na landing do produto leva ao portal de conta).

```
dominio.org                 → site mãe (landing da organização)
conta.dominio.org           → portal: login, planos, checkout, faturas (backend central)
anchor.dominio.org          → vitrine + docs do Anchor (estático)
truthid.dominio.org         → vitrine + docs do TruthID (estático)
```

## Decisões tomadas

- Backend **por projeto**, não um monólito único para tudo.
- Site mãe centraliza conta, escolha de plano e pagamento; sites de projeto são só vitrine, instalação e documentação.
- Um domínio raiz com subdomínios por projeto (em vez de um domínio por projeto).
- Projetos open source, com self-host como cidadão de primeira classe.
- Site mãe também open source (escopo da licença ainda em aberto, ver seção "Open source").
- Dashboard de vendas e gestão inicial de assinaturas/clientes como área admin dentro do site mãe (ver seção "Dashboard de vendas").

## Por que não um backend único

- Quem quiser self-host de um projeto não pode ser obrigado a subir todos os outros.
- Modelos de confiança diferentes: TruthID (auto-soberano/descentralizado), produtos que rodam local, Anchor (desktop + sidecar).
- Ciclos de release diferentes (contrato de API estável de um produto de API vs. contratos on-chain do TruthID).
- Um bug de auth em um backend único expõe todos os projetos.
- Regra prática: se dois projetos precisarem escrever no mesmo banco, o limite entre eles está errado.

## Responsabilidades do site mãe

**Dentro do escopo**
- Conta e identidade
- Catálogo de planos e assinaturas por produto
- Checkout e faturas (via gateway)
- Entitlements: "esse usuário pode usar a feature X do produto Y?"
- Medição de uso, se necessário

**Fora do escopo**
- Dados de produto (cada backend guarda os seus)
- Lógica de negócio dos produtos
- Emissão fiscal (NFS-e, boleto): pertence ao projeto de integrações burocráticas
- Contabilidade e financeiro interno completo: fica no módulo de finanças gerais do Anchor

## Open source

- **Segredos fora do código:** chave privada que assina entitlements, credenciais do gateway e segredo de webhook só em variáveis de ambiente. A segurança precisa valer com o código público (verificação de assinatura de webhook e controle de acesso sólidos).
- **Preços e planos são dados, não código:** catálogo em config ou banco, senão cada ajuste de preço vira commit público.
- **Licença (opções):** libs (cliente de entitlements, verificação de token) com licença permissiva (MIT/Apache); site mãe com AGPL, ou "source available" sem promessa de suporte.
- **Risco de abrir tudo:** atrair PRs e issues pedindo generalidade, o oposto do escopo decidido (construir para os apps da própria organização).

## Dashboard de vendas e gestão de assinaturas

Área admin fina dentro do site mãe, lendo dados que já existem.

- **Fonte da verdade do dinheiro:** o gateway. O banco do site mãe guarda cliente, status da assinatura e entitlements, sincronizados por webhooks idempotentes, guardando o evento bruto.
- **Somente leitura financeira:** não se editam valores no dashboard; ações passam pelo gateway.
- **Métricas iniciais (só estas):** assinaturas ativas por produto, MRR, novos clientes e cancelamentos, pagamentos falhos, receita por produto.
- **Cuidado com MRR e churn:** trial, plano anual e proration fazem o cálculo ingênuo mentir. Começar com definição simples e documentada, refinar com dado real.
- **Acesso:** admin com login forte (TruthID + segundo fator); dados de cliente mínimos por causa da LGPD.
- **Limite:** visão e ações sobre assinaturas. Sem contabilidade, fiscal ou contas a pagar (isso fica no Anchor e no projeto de integrações).

## Cobrança e assinaturas

- Não construir motor de cobrança do zero (retry, dunning, proration, conciliação são casos de borda demais).
- Usar um gateway atrás de uma **interface de provider**, no mesmo padrão dos outros projetos (`StorageProvider`, `AuthProvider`, `DataProvider`). Candidatos: Asaas, Stripe, Pagar.me; motor open source self-hostable: Lago.
- O site mãe guarda apenas o ID do cliente no gateway e o status do plano, nunca dados de cartão.

## Autenticação entre subdomínios

- **Evitar cookie compartilhado em `.dominio.org`**: um XSS em qualquer subdomínio comprometeria a sessão da conta.
- Usar fluxo **OIDC (authorization code + PKCE)**: o produto redireciona para `conta.dominio.org`, o pai autentica e devolve um token. Cookies ficam host-only no portal de conta.

## Entitlements offline-first

- Produtos locais (produtos locais, Anchor desktop) **não podem depender do servidor central para rodar**.
- Entitlement = token assinado (JWT ou similar) verificado offline com a chave pública do site mãe.
- Tier grátis nunca contacta o servidor; tier pago renova o token enquanto houver assinatura ativa.
- Quem faz self-host simplesmente não usa o site mãe.

## Identidade

- Ideal: a conta do site mãe ser a identidade do **TruthID**, sem uma segunda base de usuários com senha.
- Trade-off: exigir TruthID/wallet como porta de entrada aumenta a fricção para usuário comum.

## Sites de projeto

- Estáticos (Docusaurus, VitePress ou mdBook), versionados junto com o repo do projeto e gerados a partir dele.
- Não misturar landing/marketing e documentação técnica no mesmo build.
- Nada de backend nem de pagamento nesses sites: pouca manutenção e quase nenhuma superfície de ataque.

## Repositórios

- Repos separados por projeto.
- Um repo pequeno (ou workspace) de **libs comuns**: cliente de auth do TruthID, cliente de entitlements, schemas/tipos comuns, padrão de providers.
- Repo da **plataforma** (este): site mãe, portal de conta, serviço de entitlements e cobrança.
- Monorepo só se a manutenção das libs compartilhadas virar dor real.

## Riscos

- O site mãe é ponto único de falha e de ataque (guarda identidade e cobrança). Mantê-lo pequeno, com o mínimo de dados.
- "Site mãe com backend pra tudo" pode virar monólito de fato. O limite é: conta, planos, cobrança e entitlements. O resto fica nos produtos.
- Tentar generalizar para outras empresas usarem (estilo ERP) é um buraco de escopo. Construir para os apps da própria organização, com fronteiras limpas e open source, sem prometer generalidade.

## Ordem sugerida

1. Repo da organização com este documento.
2. Site mãe estático + subdomínios de docs dos projetos (GitHub Pages resolve).
3. Portal de conta e cobrança só quando o primeiro produto for cobrar de fato.
4. Dashboard de vendas (métricas mínimas) junto com a cobrança, alimentado por webhooks do gateway.
5. Serviço de entitlements com tokens assinados quando o segundo produto precisar de plano.

## Pontos em aberto

- **Login:** TruthID obrigatório ou e-mail como porta de entrada (vinculando TruthID depois)?
- **Licença:** plataforma inteira aberta ou só as libs? Qual licença (AGPL, permissiva, source available)?
- Qual produto será o primeiro a cobrar? Define o formato mínimo do módulo de entitlements.
- Nome da organização e do domínio raiz.
- Escolha do gateway e do motor de cobrança.
