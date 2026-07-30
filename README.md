# Sobre o `DESIGN.md`

> Guia de uso do [`DESIGN.md`](./DESIGN.md) — a Única Fonte da Verdade de UI/UX do GovBR-DS para geração de
> telas por assistentes de IA.

## O que é

O `DESIGN.md` faz um prompt curto — *"crie a tela de cadastro de solicitante"* — produzir código já no padrão
do Design System, sem repetir as regras a cada pedido. Ele reúne fundamentos visuais (cores, tipografia,
espaçamento, grid, elevação, estados, iconografia), a **API real dos 67 componentes** `Br*`, padrões de tela
prontos, acessibilidade (eMAG / WCAG 2.1 AA), microcopy em português e as armadilhas de Stencil + Vue.

## Estrutura do repositório

```
DESIGN.md                        ← o principal: use SEMPRE
PADRAO_IMPLEMENTACAO_GOVBR.md    ← estado, eventos e testes (equipe GovBR-DS)
README-DESIGN.md                 ← este arquivo
setup/                           ← SÓ para projeto novo
  ├── QUICKSTART_DSGOV_VUE.md    ← cria o ambiente (Vite + TS + wrapper)
  └── AGENTS.md                  ← ponto de entrada dos agentes na app gerada
```

A pasta `setup/` é **descartável do ponto de vista de quem já tem um projeto**. Se você já tem um app
GovBR-DS Vue rodando, ignore-a por completo.

---

# Como usar

Escolha o seu cenário. São dois, e não se misturam.

## Cenário 1 — Você já tem um app GovBR-DS Vue

**Copie apenas dois arquivos** para a raiz do seu projeto:

```sh
cp DESIGN.md PADRAO_IMPLEMENTACAO_GOVBR.md /caminho/do/seu/projeto/
```

Não copie a `setup/`: o QUICKSTART cria projeto do zero e o `AGENTS.md` colidiria com o que você já tem.

**Faça o agente reconhecer o documento.** Se o seu projeto já tem `AGENTS.md`, `CLAUDE.md` ou `.cursorrules`,
**não substitua** — acrescente uma linha ao que existe:

```text
Consulte o DESIGN.md na raiz antes de gerar ou alterar qualquer interface.
Ele define tokens, API dos componentes, acessibilidade e microcopy, e tem precedência sobre
os padrões de UI já presentes no código legado.
```

Se não existir nenhum desses arquivos, basta mencionar o `DESIGN.md` no prompt.

**Primeiro prompt — diagnóstico (não pule):**

```text
Leia o DESIGN.md deste projeto e execute a Seção 1.0: detecte as versões e a configuração
instaladas (bundler, wrapper Vue, TypeScript ou JavaScript) e me diga o que encontrou e como
isso afeta a geração de código. Não altere nenhuma dependência.
```

Isso importa porque **a versão instalada é a que manda**. Sem o wrapper, por exemplo, o agente gera `<br-*>`
em kebab-case com `event.detail` em vez de `v-model` — e os fundamentos visuais continuam valendo iguais.

**Depois, peça telas normalmente:**

```text
Seguindo o DESIGN.md, crie a view CadastroSolicitante.vue com nome completo, CPF, e-mail e
celular (opcional). Validação lazy no blur, dentro do Template Base.
```

> ⚠️ Em projeto legado o agente tende a **imitar as telas existentes** em vez de seguir o documento. Se o
> resultado sair fora do padrão, reforce no prompt: *"siga o DESIGN.md, não o padrão do código legado"*.

## Cenário 2 — Projeto novo, do zero

Clone este repositório, abra a pasta no seu agente e mande **um único prompt**:

```text
Leia o setup/AGENTS.md e o setup/QUICKSTART_DSGOV_VUE.md deste repositório e execute o guia de
ponta a ponta para criar o projeto Vue + GovBR-DS.

Contexto: este repositório contém só os documentos da iniciativa. Não existe aplicação aqui ainda.

Onde criar: gere o app na pasta ./app. Ao final, copie DESIGN.md e PADRAO_IMPLEMENTACAO_GOVBR.md
para ./app/ e setup/AGENTS.md para ./app/AGENTS.md, e crie ./app/CLAUDE.md apontando para ele —
assim o DESIGN.md fica na raiz da aplicação e as ferramentas o encontram nas próximas sessões.

Siga o guia à risca: últimas versões estáveis, Vite + TypeScript + <script setup>, wrapper
@govbr-ds/webcomponents-vue, e a etapa de diagnóstico (9.2) antes de adaptar qualquer página
baixada do repositório oficial.

Ao terminar, rode npm run dev e me diga: as versões instaladas, o resultado do diagnóstico
(as páginas oficiais já estavam modernas ou precisaram de adaptação?) e as rotas disponíveis.
```

O ambiente sobe em `http://localhost:5173/`. **Daí em diante o cenário passa a ser o 1**: os prompts são sobre
telas, e quem comanda é o `DESIGN.md`.

O quickstart oficial cita o Vite, mas o `npm install` do repositório instala hoje a cadeia **Vue CLI / webpack** (`@vue/cli-service`, `webpack-chain`, `consolidate`) e
reporta algumas vulnerabilidades. O guia da `setup/` monta uma base moderna, **baixa
as páginas de exemplo do repositório oficial na hora da execução** (para acompanharem atualizações) e
**diagnostica antes de adaptar**: se o repo oficial já tiver se modernizado, ele não mexe em nada.

---

## Receitas de prompt

A **Seção 18 do `DESIGN.md`** traz prompts prontos para os casos mais comuns: barra de ações de formulário,
formulário com validação, listagem com paginação e modal de exclusão, login gov.br e dashboard de indicadores.
Use-os como estão ou como modelo.

A **Seção 0 (Contrato do Agente)** concentra as regras inegociáveis — é a versão compacta para colar num prompt
de sistema.

## Política de versão

O `DESIGN.md` **não fixa versões**. A regra é:

| Cenário | Versões |
| :-- | :-- |
| Projeto novo | Últimas **estáveis** (resolvidas na execução do QUICKSTART) |
| App existente | **As que já estão instaladas** — o documento se adapta ao projeto, não o contrário |

A regra 10 do Contrato do Agente formaliza: **propor atualização é bem-vindo; executá-la sem pedir, não.**

## ⚠️ Fase de desenvolvimento e testes

Este documento está **em desenvolvimento e sob testes ativos**. Trate-o como base sólida, não como norma
fechada:

- Verifique o resultado gerado antes de levar para produção.
- A API da Seção 12 reflete a linha `2.0.0`. Ao trabalhar em outra versão, reconfira — o documento traz a
  receita para reextrair props, slots e eventos direto do `node_modules`.
- Divergências entre o documento e a [documentação oficial](https://www.gov.br/ds) resolvem-se **em favor da
  documentação oficial**. Avise a equipe para corrigirmos aqui.

## Escopo: Vue 3

O `DESIGN.md` é **específico para Vue 3** com `@govbr-ds/webcomponents-vue`. Ainda assim, a maior parte do
conteúdo é agnóstica de framework:

| Seção | Portabilidade |
| :-- | :-- |
| 2–11 (fundamentos: cores, tipografia, grid, estados…) | ✅ Totalmente reaproveitável |
| 12 (props, slots, eventos) | ✅ São Web Components; muda só a sintaxe de binding |
| 13 (padrões de tela) | 🟡 Estrutura e regras servem; o código precisa ser reescrito |
| 14–16 (acessibilidade, writing, do's/don'ts) | ✅ Totalmente reaproveitável |
| 1 e 17 (bootstrap e armadilhas Stencil + Vue) | ❌ Específicos de Vue |

> O `PADRAO_IMPLEMENTACAO_GOVBR.md` é da equipe do GovBR-DS e complementa a Seção 17 (estado, eventos
> `valueChange`, validação no `blur`, store reativo, CRUD, E2E com Playwright em Shadow DOM). Algumas partes
> citam Angular — aqui o foco é **Vue**: aproveite as seções agnósticas e traduza os testes unitários para
> Vitest.

## Referências oficiais

| Recurso | URL |
| :-- | :-- |
| Design System GovBR-DS | <https://www.gov.br/ds> |
| Web Components | <https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/webcomponents/> |
| Wrapper Vue | <https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/frameworks/vue/> |
| Wiki de desenvolvimento | <https://gov.br/ds/wiki/> |
| Quickstart Vue (oficial) | <https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue> |
| Repositório dos Web Components | <https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc/> |

A lista completa está na **Seção 19** do `DESIGN.md`.

## Dúvidas e retorno

O `DESIGN.md` melhora com uso. Ao encontrar uma regra ausente, ambígua ou que gerou código incorreto, registre
o caso — de preferência com o prompt usado e o resultado obtido.

Para dúvidas sobre o Design System em si: [site](https://www.gov.br/ds), [wiki](https://gov.br/ds/wiki/) ou
[Discord](https://discord.gg/U5GwPfqhUP).
