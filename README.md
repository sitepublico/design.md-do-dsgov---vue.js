# Sobre o `DESIGN.md`

> Guia rápido de distribuição e uso do arquivo [`DESIGN.md`](./DESIGN.md).

## O que é

O `DESIGN.md` é uma **Única Fonte da Verdade** de UI/UX do GovBR-DS, escrita para ser lida por **assistentes de IA** (Claude Code, Cursor, Copilot). O objetivo é que um prompt curto — *"crie a tela de cadastro de solicitante"* — produza código já no padrão do Design System, sem precisar repetir as regras a cada pedido.

Ele reúne em um único arquivo:

- Fundamentos visuais completos: cores, tipografia, espaçamento, grid, superfície, elevação, estados, iconografia e densidade — sempre com o **token** ou a **classe utilitária** correspondente, nunca com valores soltos.
- A **API real dos 67 componentes** `Br*`: props, tipos, enums, slots e eventos, extraídos dos artefatos publicados de `@govbr-ds/webcomponents` `2.0.0-next.70`.
- Padrões de tela prontos: formulário, listagem com CRUD, login gov.br, dashboard, empty state, tela de erro e fluxo em etapas.
- Regras de acessibilidade (eMAG / WCAG 2.1 AA) e de writing/microcopy em português.
- Armadilhas específicas de Stencil + Vue e receitas de prompt.

## ⚠️ Fase de desenvolvimento e testes

Este documento está **em desenvolvimento e sob testes ativos**. Trate-o como uma base sólida, não como norma fechada:

- Verifique o resultado gerado antes de levar para produção.
- A API da Seção 12 está fixada em `2.0.0-next.70` — uma versão *pré-release*. Ao atualizar a biblioteca, reconfira a seção (o próprio documento traz os comandos para reextrair props, slots e eventos).
- Divergências entre o documento e a [documentação oficial](https://www.gov.br/ds) devem ser resolvidas **em favor da documentação oficial**. Avise a equipe para corrigirmos aqui.

## Escopo: Vue 3

O `DESIGN.md` é **específico para Vue 3** com `@govbr-ds/webcomponents-vue`. Os exemplos usam SFC com `<script setup>`, `v-model`, `v-for` e slots do Vue.

Dito isso, a maior parte do conteúdo é **agnóstica de framework** e serve de base para outras stacks:

| Seção | Portabilidade |
| :-- | :-- |
| 2–11 (fundamentos: cores, tipografia, grid, estados…) | ✅ Totalmente reaproveitável |
| 12 (props, slots, eventos dos componentes) | ✅ Reaproveitável — são Web Components; muda só a sintaxe de binding |
| 13 (padrões de tela) | 🟡 Estrutura e regras servem; o código precisa ser reescrito |
| 14–16 (acessibilidade, writing, do's/don'ts) | ✅ Totalmente reaproveitável |
| 1 e 17 (bootstrap e armadilhas Stencil + Vue) | ❌ Específicos de Vue |

Para adaptar a React, Angular ou Web Components puros, troque as Seções 1, 13 e 17 e mantenha o restante. Consulte o [wrapper correspondente na documentação oficial](https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/webcomponents/).

## Como usar

### Recomendado: partir do Quickstart Vue

O `DESIGN.md` descreve o padrão, mas não instala nada. Ele rende muito mais sobre um projeto que já tem a biblioteca configurada — daí a recomendação de partir do quickstart oficial:

```sh
# 1. Clone o quickstart oficial
git clone https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue.git meu-projeto
cd meu-projeto

# 2. Copie o DESIGN.md para a raiz do projeto
cp /caminho/para/DESIGN.md .

# 3. Instale e rode
npm install
npm run dev
```

O projeto sobe em `http://localhost:5173/`.

Vale como ponto de partida porque o quickstart já traz o `@govbr-ds/core` importado no `main.ts`, a fonte Rawline e o Font Awesome no `index.html`, o Template Base montado em `App.vue` e exemplos funcionais de formulário e listagem — exatamente o que a Seção 1 do `DESIGN.md` descreve.

### Em um projeto Vue já existente

Copie o `DESIGN.md` para a raiz e siga a **Seção 1 (Instalação e Bootstrap)** para conferir dependências, importação do CSS e carregamento de fontes e ícones.

### Com assistentes de IA

Basta o arquivo estar na raiz do repositório — as ferramentas costumam localizá-lo sozinhas. Para tornar isso explícito e confiável:

- **Claude Code:** referencie no `CLAUDE.md` do projeto, por exemplo: *"Consulte o `DESIGN.md` antes de gerar ou alterar qualquer interface."*
- **Cursor:** adicione a mesma instrução em `.cursorrules`.
- **Em qualquer ferramenta:** mencione o arquivo no prompt — *"seguindo o DESIGN.md, crie…"*.

A **Seção 18 (Receitas de Prompt)** traz prompts prontos para os casos mais comuns (barra de ações, formulário com validação, listagem com paginação, login gov.br, dashboard). Use-os como estão ou como modelo.

A **Seção 0 (Contrato do Agente)** concentra as 9 regras inegociáveis. Se precisar de uma versão ultracompacta para colar em um prompt de sistema, é essa seção.

## Uso combinado

| Documento | Papel |
| :-- | :-- |
| [`DESIGN.md`](./DESIGN.md) | **O quê** desenhar: tokens, componentes, padrões de tela, acessibilidade |
| [`PADRAO_IMPLEMENTACAO_GOVBR.md`](https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue/-/blob/main/PADRAO_IMPLEMENTACAO_GOVBR.md) | **Como** implementar: estado, eventos, testes E2E com Playwright em Shadow DOM |

Os dois se complementam: o `DESIGN.md` cobre a camada visual e de padrões; o Padrão de Implementação cobre a integração e os testes.

## Referências oficiais

| Recurso | URL |
| :-- | :-- |
| Design System GovBR-DS | <https://www.gov.br/ds> |
| Web Components | <https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/webcomponents/> |
| Wrapper Vue | <https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/frameworks/vue/> |
| Wiki de desenvolvimento | <https://gov.br/ds/wiki/> |
| Quickstart Vue | <https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue> |
| Repositório dos Web Components | <https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc/> |

A lista completa está na **Seção 19** do `DESIGN.md`.

## Dúvidas e retorno

O `DESIGN.md` melhora com uso. Ao encontrar uma regra ausente, ambígua ou que gerou código incorreto, registre o caso — de preferência com o prompt usado e o resultado obtido.

Para dúvidas sobre o Design System em si, use os canais oficiais: [site](https://www.gov.br/ds), [wiki](https://gov.br/ds/wiki/) ou [Discord](https://discord.gg/U5GwPfqhUP).
