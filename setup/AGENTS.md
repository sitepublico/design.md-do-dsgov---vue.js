# AGENTS.md — Instruções para agentes de IA neste repositório

> **Este arquivo é destinado a projetos NOVOS**, criados a partir do `QUICKSTART_DSGOV_VUE.md`. Ele vai para a
> raiz da aplicação gerada e serve de ponto de entrada para qualquer agente (Claude Code, Cursor, Copilot…).
>
> **Já tem um `AGENTS.md` no seu projeto?** Não substitua. Copie apenas o bloco "Regras do GovBR-DS" abaixo e
> cole no arquivo existente, entre marcadores, para conseguir reaplicá-lo depois:
>
> ```text
> <!-- govbr-ds:start -->  …conteúdo…  <!-- govbr-ds:end -->
> ```
>
> Para Claude Code, mantenha uma cópia como `CLAUDE.md` (ou `ln -s AGENTS.md CLAUDE.md`).

## O que é este projeto

Base Vue 3 + Vite + **GovBR-DS** (Web Components via wrapper oficial) usada para **gerar telas no padrão
do Governo Federal a partir de prompts**. A identidade visual, a API dos componentes e as regras de
acessibilidade são **inegociáveis** e vivem no `DESIGN.md`.

## Ordem de leitura obrigatória

1. **`DESIGN.md` — Fonte Única da Verdade de UI/UX.** Leia sempre. Comece pela Seção 0 ("Contrato do
   Agente"); use as Seções 3–12 ao escrever markup; a Seção 13 ao montar telas inteiras; a 14 (checklist
   de acessibilidade) antes de considerar a tela pronta; a 18 tem receitas de prompt prontas.
2. **`PADRAO_IMPLEMENTACAO_GOVBR.md` — padrões de implementação e teste.** Consulte ao lidar com sincronização
   de estado (`valueChange`/`event.detail`), validação reativa no `blur`, store centralizado, formulário
   criar/editar (bi-modal), `br-table` responsiva, CRUD e testes E2E (Playwright sobre Shadow DOM).
3. **`setup/QUICKSTART_DSGOV_VUE.md` — setup do ambiente.** Já foi executado para criar este projeto;
   consulte apenas se precisar reconfigurar a base.

## Antes de gerar código: detecte o contexto

Este repositório pode ser um **projeto novo** ou um **app GovBR-DS Vue já existente**. Não presuma — verifique:

```bash
npm ls @govbr-ds/core @govbr-ds/webcomponents @govbr-ds/webcomponents-vue vue
ls vite.config.* vue.config.js 2>/dev/null   # Vite ou Vue CLI?
```

- **A versão instalada é a que vale.** Gere código compatível com ela, não com a última publicada.
- **Nunca atualize dependências por conta própria** — proponha, não execute.
- Sem o wrapper (`webcomponents-vue`) instalado: use `<br-*>` em kebab-case com `event.detail`, sem `v-model`.
  Os fundamentos visuais, a acessibilidade e o microcopy continuam valendo integralmente.
- Detalhes e tabela de decisão: `DESIGN.md` Seção 1.0.

## Stack deste projeto

- Vue 3 (`>=3.3`) + Vite + **TypeScript** + `<script setup lang="ts">`.
- `@govbr-ds/core`, `@govbr-ds/webcomponents` e `@govbr-ds/webcomponents-vue` nas **últimas estáveis**, sempre
  no **mesmo major** (são peerDependencies entre si).
- Importe os componentes por SFC: `import { BrButton, BrInput } from '@govbr-ds/webcomponents-vue'`;
  prefira PascalCase no template (`<BrButton>`) para ter validação de props.
- `@govbr-ds/core/dist/core.min.css` importado **primeiro** no `main.ts`. Fontes/ícones via CDN no `index.html`,
  com `<html lang="pt-BR">`.

## Regras que o agente NÃO pode violar (resumo — detalhes no `DESIGN.md` §0 e §16)

- Use os componentes do Design System; só escreva HTML/CSS cru quando não existir componente equivalente.
- **Respeite a versão instalada** (regra 10 do Contrato do Agente).
- **Nunca invente props.** A API real está no `DESIGN.md` §12. Prop que não está lá não existe.
- Nunca escreva hexadecimal nem `px` em CSS custom — use tokens/classes utilitárias.
- Nunca remova o foco (`outline: none` sem substituto = violação de acessibilidade).
- Toda tela no Template Base: Header → Conteúdo → Footer (§6.4), com `BrSkipLink`.
- Português do Brasil, voz ativa, microcopy conforme §15.
- Cumpra o checklist de acessibilidade (§14) antes de dar a tela por concluída.

## Fluxo ao receber um pedido de tela

1. Identifique o padrão de tela em `DESIGN.md` §13 (formulário, CRUD, login, dashboard, empty state, erro, etapas).
2. Monte o markup só com componentes/props reais da §12; aplique cores, tipografia e espaçamento por token/classe.
3. Trate eventos conforme §17 e o `PADRAO` (`v-model` nos campos de formulário; `event.detail` no resto).
4. Rode o checklist de acessibilidade §14.
5. Se o pedido envolver testes, siga a estratégia E2E do `PADRAO` (§4 e §5).

## Manutenção

Ao subir a versão da biblioteca, reconfira a Seção 12 do `DESIGN.md` contra os artefatos do novo pacote
(recipe em `DESIGN.md` §19). Ao alterar regras, atualize `DESIGN.md` **e** este arquivo.
