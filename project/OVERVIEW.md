# arven: Visão geral

## O que é
Site mãe da Arven Labs e seu backend: conta, planos, cobrança e entitlements. Os produtos mantêm backends independentes e falam com este só por API; os sites dos produtos são estáticos (vitrine, instalação, docs). Entitlements são tokens assinados, verificáveis offline. Detalhes no `SPEC.md`.

## Stack
Não decidida. Fica para a Fase 0 (gateway de cobrança, login e domínio em aberto, ver `PENDING.md`).

## Licença
AGPL-3.0 (ADR 0002).

## Status
Pré-projeto: só governança (ruleset, `guard`, template de PR) e o documento de design. Ordem sugerida no `SPEC.md`: site mãe estático e docs dos projetos primeiro; portal de conta e cobrança só quando o primeiro produto for cobrar.
