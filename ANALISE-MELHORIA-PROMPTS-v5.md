# Análise de melhoria dos prompts — Nodimus 3D Studio v5.0.0 (Claude Code)

Base: `CLAUDE.md` (3,7 mil palavras), 127 arquivos em `docs/prompts/` (1,47 MB), 85 logs (457 KB), 3 skills do fluxo + 5 do projeto, `scripts/ais-check-docs.mjs`, `CHANGELOG-PADRONIZACAO.md`, `RELATORIO-BLOQUEIOS.md` e `AJUSTES.txt`. Tudo abaixo foi conferido nos arquivos; números vêm de `grep`/`wc` sobre o zip.

## Resumo

O sistema está **bem alinhado no essencial**: hierarquia da verdade clara, decisões numeradas, PARE E PERGUNTE funcionando (os logs mostram decisões A/B/C reais e coerentes), gate único (`npm run check`) e `ais:docs` impedindo drift de numeração. O que degrada daqui para frente não é falta de regra, e sim **três tipos de drift**: (1) texto que descreve a casca/versão e envelhece, (2) a mesma informação copiada em muitos lugares, (3) lacunas de processo que se repetem nos logs sem dono. Abaixo, em ordem de retorno.

---

## 1. Inconsistências factuais (corrigir já — baratas)

| # | Achado | Evidência | Correção |
|---|---|---|---|
| 1.1 | **`CLAUDE.md` aponta para a regra errada.** O cabeçalho diz "ajuste a regra 9 (a lista de `NodeType`…)", mas a regra 9 hoje é *Nomenclatura*; a lista vive na decisão 19. `ais/SKILL.md` e `docs/prompts` citam "regra 9" para "o código manda" (essa sim é a 9). | `CLAUDE.md` l.5 × l.124 | Trocar por "decisão 19". |
| 1.2 | **Um prompt com total errado:** `AIS-61` diz "nº … de **123**"; os outros 124 dizem 124. | `grep 'de 123'` | `ais-check-docs` deveria ter pego; ver 4.2. |
| 1.3 | **Total de posições citado com 9 valores diferentes** (105, 115, 116, 117, 119, 120, 121, 123, 124) em README de prompts, MATRIZ, RELATORIO, ORDEM. | `grep` de "posições" | Só a ORDEM guarda o número; os demais docs dizem "ver ORDEM" ou são gerados pelo script. |
| 1.4 | **"Estado real da casca" desatualizado em massa:** `AssetImportModal.tsx` aparece com 2364/2375 linhas; hoje tem **276** (virou `assetImport/`). `Viewport3D.tsx` 1690 → **2118**. `BottomPanel` 187 → 195. A regra 10 do CLAUDE.md ("500+ linhas") cita só arquivos que já mudaram de tamanho/lugar. | `wc -l` | Ver 3.1 (remover contagem de linhas da tabela; ela envelhece a cada prompt). |
| 1.5 | **Rótulos legados nos cabeçalhos:** 14 prompts ainda trazem "Decisão Canônica Relacionada: Item N (Opção X)", que o próprio CLAUDE.md declara obsoleto. Outros têm texto livre ("Validação das 3 deliberações oficiais"). | `grep` | Padronizar: "Decisões: 2, 8" (números do CLAUDE.md) ou remover. |
| 1.6 | **Resíduos do AI Studio:** 26 menções a "Preview" e 8 a "AI Studio" nos prompts; logs antigos ainda dizem "Preview do AI Studio". | `grep` | Substituir por "navegador (`npm run dev`)" nos prompts; logs são histórico, deixar. |
| 1.7 | **Nomes de arquivo carregam versão antiga:** `ORDEM-CANONICA-V4.md`, `MAPA-ARQUITETURA-V4`, `ALINHAMENTO-V4` no produto v5.x; a MATRIZ e o RELATORIO são "históricos" mas continuam na raiz de `docs/`. | listagem | Renomear sem sufixo (script já centraliza o caminho em um lugar) e mover históricos para `docs/legado/`. |
| 1.8 | **`AJUSTES.txt` na raiz** (comportamento de câmera) é uma decisão pendente fora do fluxo: não está em CLAUDE.md, ORDEM, nem log. Começa com "CCOMPORTAMENTO" (typo). | arquivo | Virar decisão canônica (ver 2.4) ou emenda do AIS-47; depois apagar. |

---

## 2. Gaps de conteúdo

**2.1 Skills embutidas: 187 blocos em 97 prompts, mas só 5 skills existem de verdade.**
Os prompts citam ~40 nomes (`nodimus-scenegraph-modelo` ×23, `nodimus-projetos-workspace` ×11…) cujo texto está **copiado dentro de cada prompt**. Na amostra, a mesma skill aparece em 3 versões diferentes (18 idênticas, 2 e 1 divergentes) — ou seja, já há drift. Isso infla contexto (o AIS-AP tem 53 KB; AIS-25b 32 KB) e deixa a skill "viva" e a "embutida" dessincronizadas.
→ Mover cada skill para `.claude/skills/<nome>/SKILL.md` (já é o mecanismo do Claude Code: carregamento sob demanda, `description` decide quando), e o prompt só lista "Skills: `a`, `b`". Um arquivo por skill, uma verdade. Ordem sugerida: as citadas ≥ 6 vezes primeiro (≈ 15 skills cobrem 80% dos blocos).

**2.2 Skill × spec em conflito sem regra de desempate.**
O AIS-47 parou em PARE E PERGUNTE porque a skill (`nodimus-controle-camera-colisao`) mandava Behaviors em `Camera` e a spec do prompt, no personagem. Foi resolvido caso a caso (opção A) e o seu `AJUSTES.txt` agora propõe a separação oposta (câmera independente). Falta no CLAUDE.md uma linha na *Hierarquia da verdade*: **skill é referência de conhecimento; prompt e decisões canônicas vencem; divergência = PARE E PERGUNTE e o resultado vira decisão numerada**.

**2.3 Verificação visual sem plano B padronizado.**
23 linhas de log citam o mesmo bloqueio: painel embutido sem PointerLock, mouse sintético não dispara DnD HTML5, `requestAnimationFrame` suspenso, janela estreita, sandbox sem `npm run dev`. Resultado: 30 logs "concluído com pendências" e dezenas de `NÃO VERIFICADO` (AIS-44, 37, 35, 26b, 05r com 2 cada) que dependem do usuário conferir a mão.
→ Regra 5 ganha uma tabela "limites conhecidos do navegador do agente → como cobrir": (a) interação nativa (PointerLock, DnD, gizmo) → teste puro/jsdom + **checklist de 3 linhas para o usuário** no resumo final, em formato fixo; (b) rAF suspenso → stepar a simulação por chamada direta (já feito no AIS-47, vale como padrão); (c) Playwright headless disponível no ambiente → usar para o que o painel não faz. E um campo novo no log: `Conferência manual pelo usuário:` (separado de "Pendências").

**2.4 Camadas de decisão sem lugar de nascimento.**
Decisões nascem de logs (24, 26, 27 vieram de realinhamentos), mas não há um fluxo escrito de "decisão → CLAUDE.md → prompts afetados". Resultado: a v5.6 teve de corrigir 96 prompts com "decisões 1–22" quando já eram 27, e o `$1|` quebrado na tabela. O texto desses preâmbulos agora aponta para o CLAUDE.md (bom), mas o cabeçalho "Decisões canônicas v4.6.1 (15–20) / v5.1 (23) / v4.7.0 (21–22)…" está **fora de ordem e agrupado por versão**, não por assunto.
→ Uma tabela única 1–N ordenada; coluna "dono (AIS)" e "prompts que consomem"; e um item de checklist do log: *"Este prompt criou/alterou decisão? → registrar no CLAUDE.md + rodar `ais:docs`"*.

**2.5 Pendências "herdadas" sem rastreio.**
Só 3 logs usam "Herdadas e já redistribuídas"; o resto lista pendências em prosa. Não há índice. Exemplos recorrentes: terreno real (AIS-50), Worker de scripts (AIS-79), `CollidesWithWorld`/`CanFly` (AIS-47), `run_script` desabilitado (6 prompts).
→ `docs/PENDENCIAS.md` (ou campo estruturado no log: `- [ ] <texto> → dono: AIS-NN`) e `ais-proximo` listar as pendências cujo dono é o prompt a executar. O `ais-check-docs` verifica que o dono existe.

---

## 3. Consistência e nivelamento entre prompts

**3.1 Tabela "Estado real da casca" é a parte mais cara de manter e a que mais envelhece** (1.4). O `ais/SKILL.md` já diz "o código manda". Proposta: a tabela guarda só **caminho + EXISTE/CRIAR** (sem contagem de linhas) e o `ais-status` valida que todo "EXISTE" existe e todo "CRIAR" não existe (falha vira aviso ao abrir o prompt, não ao escrever). Isso substitui a verificação manual que os logs fazem hoje.

**3.2 Tamanho muito desigual.** 2,6 KB (AIS-51c) a 53 KB (AIS-AP) e 32 KB (AIS-25b); metade do peso dos grandes são skills embutidas (2.1). Dividir o AIS-AP em passos `AP-NN` como arquivos próprios (o apêndice já adota IDs por passo e logs por passo, mas segue num único arquivo — o `/ais AP-63` precisa ler 53 KB para executar 1 passo). Prompts de 3 KB como 51b/c/d são o formato-alvo: spec curta + critérios + referência à skill.

**3.3 Critérios de aceite com granularidade variável.** `AIS-AP` usa `AP-62-C01 "Execução limpa sem regressão visual…"` (genérico, copiado), enquanto prompts novos têm critérios verificáveis (`AIS-47-C02`: "W anda 2,74 m"). Os genéricos `C01/C02` repetidos ("Execução limpa", "Tipagem estrita validada") são cobertos pelo gate e pela regra 5; devem sair dos prompts e ficar numa linha fixa do log (`Gate: OK`), deixando os critérios só para o que é específico. Isso também reduz log (hoje 457 KB, ≈ 5 KB por prompt).

**3.2b Formato do log não é aplicado a todos:** 9 logs iniciais (AIS-01…05) não têm *Validação*; `AIS-02` e `AIS-76-pre` não têm *Resultado*; `AIS-26b-v1-rascunho` e `AIS-FIX-persistencia` não têm critérios `Cnn`. Parte é pré-regra (OK), mas `AIS-FIX-*`/`-pre`/`-rascunho` mostram que há trabalho legítimo **fora do ID de prompt**. Criar categoria explícita no formato (`AIS-FIX-<tema>`, `AIS-<ID>-pre`) com um cabeçalho curto próprio, em vez de exceções silenciosas.

**3.3b Reexecuções são o sinal mais valioso e hoje são tratadas como ruído.** 30 logs têm ≥ 2 execuções; os motivos se repetem:
- **Bug só visível no uso real** (AIS-19: filho não herdava transform do pai; AIS-24: arrasto gera 1 gesto no histórico; AIS-38: rig falso transitório) → o prompt deveria ter exigido, em critério, a **interação de uso real** (mover pai/filho, arrastar → Undo único). Padrão a adicionar em prompts de editor: *critério de usuário-primeiro* ("faça X com o mouse, observe Y").
- **Pendência resolvida a pedido** (AIS-35/36/37: "opção B do `ais-proximo`") → ver 2.5.
- **Reconciliação documental sem código** (v5.1.1, FIX-persistencia) → ver 4.

---

## 4. Processo e automação

4.1 **`ais-check-docs` cobre ordem/ID/links; falta cobrir o que mais driftou:** (a) total "de N" em todo o texto (pegaria 1.2 e 1.3); (b) skills citadas existem em `.claude/skills/` (hoje ~35 inexistentes — pós 2.1 vira checagem útil); (c) decisão citada (`decisão N`) ≤ máximo em CLAUDE.md; (d) "regra N" citada aponta para título esperado; (e) log tem os campos obrigatórios.

4.2 **Permissões em `.claude/settings.json`:** `deny: git push *` conflita com o fluxo de sessões em nuvem (branch designada + push), mas é coerente com a regra 8 local. Vale um perfil por ambiente (local: deny; nuvem: allow só na branch da sessão) — senão o agente em nuvem precisa pedir permissão sempre.

4.3 **Contexto fixo alto:** `CLAUDE.md` ≈ 5 mil tokens carregados em toda sessão, incluindo a tabela de 28 contratos. A tabela de contratos só é relevante para o prompt que a consome. Sugestão: manter no CLAUDE.md apenas hierarquia, regras 1–14, formato do log e as decisões 1–27 em uma linha cada; mover a tabela de contratos para `docs/CONTRATOS.md` e referenciá-la pelo prompt/`ais` (que já lê o prompt antes de codar).

4.4 **`/ais` já faz o certo (checa ordem, cria tarefas por critério, PARE E PERGUNTE antes de implementar).** Acrescentar um **passo 0 de "pré-condições"**: verificar `git status` limpo, `npm run check` verde *antes* de começar (para distinguir regressão própria de anterior — regra 4 assume isso e os logs mostram erros de console herdados misturados), e declarar qual navegador/ferramenta será usado.

---

## Plano sugerido (do mais barato ao mais estrutural)

| Ordem | Ação | Esforço | Efeito |
|---|---|---|---|
| 1 | Corrigir 1.1, 1.2, 1.3, 1.5, 1.6, 1.8 | ~1 h | consistência imediata |
| 2 | Hierarquia da verdade: regra skill × prompt (2.2); tabela de limites do navegador (2.3) | 1 h | menos NÃO VERIFICADO e menos PARE E PERGUNTE repetido |
| 3 | Estender `ais-check-docs` (4.1) e tirar contagem de linhas da tabela da casca (3.1) | 2–3 h | elimina drift por construção |
| 4 | Campo `Conferência manual` + `docs/PENDENCIAS.md` (2.3, 2.5) | 2 h | fecha o ciclo de pendências |
| 5 | Skills embutidas → `.claude/skills/` (2.1), começando pelas 15 mais usadas | 1 dia | −30–40% do tamanho dos prompts, uma verdade por skill |
| 6 | Dividir AIS-AP em arquivos por passo (3.2); CONTRATOS.md separado (4.3) | 3–4 h | contexto menor por execução |
| 7 | Decisão da câmera (`AJUSTES.txt`) → decisão 28 + emenda AIS-47/58 | decisão sua | fecha o conflito skill × spec |

## Pontos de decisão (seus)

1. **Câmera:** adotar a separação do `AJUSTES.txt` (câmera como nó independente + Behavior de câmera + runtime como autoridade) como decisão 28? Isso reverte a opção A tomada no AIS-47 (Behaviors no personagem, câmera de jogo só com controlador) e afeta AIS-47/58/77. Recomendação: sim, mas como **emenda aditiva** (o controlador atual continua válido como preset "câmera embutida"), para não reabrir o log do AIS-47.
2. **Skills:** migrar todas para `.claude/skills/` ou só as mais usadas? Recomendação: as 15 mais citadas agora, o resto sob demanda quando o prompt for executado.
3. **Renomear `*-V4`** (1.7)? Recomendação: sim, junto com o passo 3, pois o script já centraliza o caminho.

Se quiser, aplico os passos 1–3 diretamente no zip (arquivos corrigidos + `ais-check-docs` estendido) e devolvo o diff.
