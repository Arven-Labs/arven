# 0002: Repositório público na Arven-Labs, licença AGPL-3.0

**Data:** 2026-11-12

## Contexto
O documento de design previa o site mãe open source, com o escopo da licença em aberto: libs permissivas, site mãe com AGPL ou "source available". Este repo guarda conta e cobrança, não bibliotecas.

## Decisão
- Repositório `Arven-Labs/arven`, **público**.
- Licença **AGPL-3.0**.
- As libs comuns (cliente de entitlements, verificação de token) ficam em repo próprio, quando existirem, com licença permissiva.

## Alternativas descartadas
- MIT: permitiria um fork comercial como serviço sem devolver o código.
- Source available: não é open source; contraria a decisão de abrir.
- Repo privado: o design prevê o código público, com segurança que vale com ele aberto.

## Consequências
- Quem rodar versão modificada como serviço de rede precisa publicar o código.
- Segredos (chave de assinatura de entitlements, credenciais do gateway, segredo de webhook) só em variáveis de ambiente; preços e planos são dados, não código.
- Público é praticamente irreversível para o que já foi publicado.
- Código de autenticação, cobrança e assinatura exige confirmação e revisão de segurança (AGENTS.md §4).
