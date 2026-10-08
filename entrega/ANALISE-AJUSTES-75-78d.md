# Análise — impacto dos ajustes AIS-75 a AIS-78d nos prompts vindouros (revisão v6.9)

Base: logs de AIS-75, 76/76b, 77, 78c, 78d; `CLAUDE.md` (decisão 32); `src/runtime/engineApi.ts` (1.4); `src/core/capabilities.ts`; prompts 79–85c; specs e docs de processo. Conferido nos arquivos do zip.

## Veredito
A ordem canônica **continua** (135 posições). Foram necessárias alterações em AIS-79 (bloqueante), AIS-81, AIS-85, 85b, 85c, e ajustes de texto em 80, 82, 83, 84. **AIS-77 foi reaberto**: concluiu só C01–C05; C06–C08 (emenda v5.8) nunca foram executados.

## Alterações aplicadas
| Arquivo | Mudança |
|---|---|
| `AIS-79` | skill `nodimus-engine-api` reescrita sobre `ENGINE_OPERATIONS` (a API antiga citada não existe); regras de dono/`no_owner`, `negotiateEngineApi`, assinaturas por `subscriptionId`; critérios de fronteira; CSP deduplicada (4× → 2 linhas); relaciona AIS-78d |
| `AIS-81` | `getNodesByTag` (inexistente) → cria `nodes.byTag` na EngineApi 1.5 |
| `AIS-85` | §1.1 debug overlay só de Play; §5 aponta ao `CapabilityRegistry` em vez de `PUBLISH_CAPABILITY_MATRIX` |
| `AIS-85b` | §4 consome `capabilities.ts`/`capabilityCatalog.ts`/`requires` (29 pastas já declaradas) |
| `AIS-85c` | usa `resolveSupport`/`unsupportedFor`/`TARGET_PROFILES`; preâmbulo "1–20" corrigido |
| `AIS-80/82/83/84` | preâmbulo sem número fixo de decisões; `SpriteAsset` → `SpriteProject` (83, 84) |
| `CLAUDE.md` | cabeçalho aponta decisão 19 (não "regra 9"); regra 2: conferir existência antes de criar (incidente do 78c); decisão 32: sem `getNodesByTag`, AIS-79 marshala só o subconjunto `marshalable` |
| `docs/specs/dominios/node.spec.md` | `AiService`/`ctx.ai` → `ActorRegistry` + `state.memory.brain` + `ctx.engine.*` |
| `docs/logs/AIS-77.md` | Execução 2 PENDENTE (C06–C08) → `ais:status` passa a indicar AIS-77 como próximo |
| `ORDEM-CANONICA-V4.md`, `CHANGELOG-PADRONIZACAO.md` | revisão v6.9 |

## Documentos relacionados examinados (sem alteração necessária)
- `.claude/skills/ais`, `ais-proximo`, `ais-verificar`, `nodimus-logic-pastas`, `nodimus-projeto`: sem referência a `ctx.ai`/`AiService`; "regra 9" nelas é a regra correta.
- `scripts/ais-check-docs.mjs`: continua verde (135/135/207).
- `docs/specs/dominios/sprite.spec.md`, `logic-registry.spec.md`, `tauri-bridge.spec.md`: sem conflito detectado com 75–78d (não reescritos; vale conferir `logic-registry.spec.md` quanto a `requires`).

## Pontos deixados sem alteração (decisão sua)
1. **Prompts já executados** (AIS-06, 14, 15, 16, 24, 25b, 27, 31 citam `getNodesByTag`; AIS-45/49 citam `getNodeProperty('transform.*')`): são histórico; não reescritos.
2. **AIS-77**: executar C06–C08 antes do AIS-79 (recomendado) ou mover para um AIS-77b (renumeraria a ordem).
3. Pendentes declaradas nos logs: AIS-75-C03 e conferência manual do "Laboratório 2D" (teclado/mouse no Tauri); AIS-76 Collision Mask/Separate Texture Page ("em breve") e etiqueta `SIGNALS_WITHOUT_EMITTER` desatualizada; AIS-76b conferência manual no Play; AIS-78c: conferir se o `engineApi.ts` original do AIS-48 tinha algo além de `onPhysicsEvent` (não há cópia).
4. AIS-79 aplica `nodes.byTag` só depois do AIS-81 estar feito — hoje o AIS-81 vem depois do 79 na ordem; o texto do 79 trata os dois casos.
