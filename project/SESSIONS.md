# arven: Log de sessões

Mais recente no topo.

### 2026-11-12 — Sessão 1 (criação do projeto)

- **Pedido**: criar o projeto a partir do documento "Plataforma da organização" (site mãe), com o nome arven.
- **Decisão do dono**: repo público, AGPL-3.0, só governança e docs; spec movida da raiz do workspace para `project/SPEC.md`, com as menções a um produto privado trocadas por termos genéricos.
- **Feito**: repo `Arven-Labs/arven`, commit inicial na `main`, PR #1 com template, `guard` e `project/`.
- **CI**: a conferir no PR #1 (`guard`).
- **Não verificado**: ruleset e segurança ainda não ligados (dependem do `guard` verde na `main`).
- **Fica**: ligar ruleset, secret scanning, push protection e alertas do Dependabot depois do merge.
