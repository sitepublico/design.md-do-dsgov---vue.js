# QUICKSTART — Vue 3 + Vite + GovBR-DS (Web Components) — versão atualizada (híbrido)

> **Objetivo (para a IA que executa este arquivo):** recriar um ambiente equivalente ao
> [`govbr-ds-wbc-quickstart-vue`](https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue),
> porém com stack moderna (**Vite** no lugar do Vue CLI) e o DSGov na **versão estável mais recente**,
> usando o **wrapper Vue oficial** (`@govbr-ds/webcomponents-vue`).
>
> **Papel deste arquivo:** é um **complemento do [`DESIGN.md`](./DESIGN.md)**, que é o documento principal da
> iniciativa. Este guia só monta o ambiente; **o que desenhar e como escrever as telas está no `DESIGN.md`**.
> Use-o apenas para **projetos novos** — em app GovBR-DS Vue já existente, vá direto ao `DESIGN.md` §1.0.
>
> **Estratégia híbrida:** as páginas de exemplo (`src/views`, `src/components`) são **baixadas do repositório
> oficial na hora da execução** — assim acompanham atualizações do repo sem precisar editar este guia — e depois
> **adaptadas** ao wrapper (ganham `v-model` e os padrões do `DESIGN.md`).
>
> **Por que modernizar:** o `npm install` no quickstart oficial hoje instala a cadeia **Vue CLI / webpack**
> (`@vue/cli-service`, `webpack-chain`, `consolidate`) e reporta **dezenas de vulnerabilidades, incluindo
> críticas**. Migrar para Vite + wrapper resolve isso.
>
> Ao final, `npm run dev` deve subir em `http://localhost:5173` com layout gov.br e as páginas do quickstart
> funcionando, sem erros no console.

---

## 0. Instruções para o agente de IA

- Execute os passos **na ordem**. Onde houver bloco de arquivo, **crie/substitua** com exatamente aquele conteúdo.
- Use **JavaScript** por padrão (compatível com o quickstart). Para TypeScript, veja o Apêndice A.
- Os passos 1–8 montam a base modernizada. O **Passo 9** baixa e **adapta** as páginas oficiais.
- Faixas de versão usam `^` (caret): atualizam patch/minor sozinhas via `npm update`, travando só no próximo major.
- Ao terminar, rode `npm run dev`, abra a Home e confirme: header gov.br renderiza, navegação funciona,
  e não há erros `Failed to resolve component: br-…` no console.

### Stack-alvo

Projeto novo = **últimas versões estáveis**. Não fixe números neste guia; resolva na execução (Passo 2).

| Pacote | Papel |
|---|---|
| `vue` (>=3.3 exigido pelo wrapper) + `vite` + TypeScript | framework e build |
| `vue-router` | roteamento SPA |
| `@govbr-ds/core` | tokens CSS, utilitários, grid |
| `@govbr-ds/webcomponents` | Web Components (peer do wrapper) |
| `@govbr-ds/webcomponents-vue` | **wrapper Vue** (v-model, tipagem, router) |

> **Por que o wrapper?** O approach antigo do quickstart usa Web Components crus + `isCustomElement`, o que
> quebra `v-model` e tipagem. O wrapper gera componentes Vue de verdade (Stencil `vue-output-target`) que
> registram o custom element sozinhos e suportam `v-model` nos campos de formulário.

---

## 1. Criar o projeto (Vite + Vue Router, não-interativo)

```bash
npm create vue@latest meu-projeto-dsgov -- \
  --router --typescript \
  --no-jsx --no-pinia \
  --no-vitest --no-e2e --no-eslint --no-prettier

cd meu-projeto-dsgov
npm install
```

> Se as flags não forem aceitas, rode `npm create vue@latest meu-projeto-dsgov` e responda:
> TypeScript **Yes**, JSX **No**, Vue Router **Yes**, Pinia **No**, testes/lint **No**.
>
> **TypeScript é o padrão** aqui: é o que o quickstart oficial usa e o que o `DESIGN.md` assume nos exemplos
> (`<script setup lang="ts">`). Para JavaScript, troque por `--no-typescript` e remova as anotações de tipo.

---

## 2. Instalar as dependências do DSGov

```bash
# Projeto NOVO: instale as últimas estáveis (o @latest resolve na hora da execução)
npm install @govbr-ds/core@latest \
            @govbr-ds/webcomponents@latest \
            @govbr-ds/webcomponents-vue@latest

# Confira o que foi resolvido e registre no README do projeto
npm ls @govbr-ds/core @govbr-ds/webcomponents @govbr-ds/webcomponents-vue
```

> O `npm install …@latest` grava faixas com caret (`^`) no `package.json`, então atualizações de patch/minor
> chegam depois via `npm update`. Mantenha os três pacotes no **mesmo major**.

> `@govbr-ds/webcomponents-vue` declara `@govbr-ds/webcomponents` e `vue` como **peerDependencies**;
> por isso o `webcomponents` é instalado explicitamente. Mantenha os três no **mesmo major**.

---

## 3. `vite.config.ts`

Substitua o arquivo inteiro:

```javascript
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
})
```

> **Importante:** ao usar o **wrapper**, **NÃO** configure `isCustomElement` para casar `br-*`.
> Se marcar `br-*` como custom element, o Vue deixa de resolver os componentes do wrapper e o `v-model` para de funcionar.
> (Só use `isCustomElement: (tag) => tag.includes('br-')` no modo cru — Apêndice B.)

---

## 4. Fontes e ícones no `index.html`

Web Components isolam estilos no Shadow DOM, mas **fontes e ícones precisam estar no documento**.
Adicione dentro do `<head>` de `index.html`:

```html
<link rel="stylesheet" href="https://cdngovbr-ds.estaleiro.serpro.gov.br/design-system/fonts/rawline/css/rawline.min.css" />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css" />
```

> Sem acesso a CDN? Instale `@fortawesome/fontawesome-free` via npm e importe o CSS no `main.ts`.
> Confirme `<html lang="pt-BR">` — é requisito de acessibilidade (`DESIGN.md` §1.3).

---

## 5. Como importar os componentes (padrão do `DESIGN.md`)

**Não é preciso registrar nada globalmente.** O padrão da iniciativa é **importar por SFC**, do jeito que o
`DESIGN.md` §1.4 define — assim o TypeScript valida props e o tree-shaking funciona:

```vue
<script setup lang="ts">
import { BrButton, BrInput, BrMessage } from '@govbr-ds/webcomponents-vue'
</script>

<template>
  <BrInput label="Nome completo" v-model="nome" />
  <BrButton emphasis="primary" type="submit">Salvar Alterações</BrButton>
</template>
```

Prefira **PascalCase** (`<BrButton>`): o Vue resolve como componente registrado e valida as props.
A forma `<br-button>` também funciona, mas perde a validação.

> **Registro global (opcional).** Se você tem muitas telas legadas em kebab-case e não quer adicionar imports
> em cada arquivo, pode registrar tudo de uma vez criando `src/plugins/govbr-ds.ts`:
>
> ```ts
> import * as GovBRDS from '@govbr-ds/webcomponents-vue'
> import type { App } from 'vue'
>
> export default {
>   install(app: App) {
>     for (const [name, component] of Object.entries(GovBRDS)) {
>       if (name.startsWith('Br')) app.component(name, component as never)
>     }
>   },
> }
> ```
>
> Custo: perde-se o tree-shaking (todos os componentes vão para o bundle). Use só se precisar.

---

## 6. `src/main.ts`

O CSS do `core` vem **primeiro** — é o que define os tokens usados por tudo o mais (`DESIGN.md` §1.2):

```ts
import '@govbr-ds/core/dist/core.min.css' // OBRIGATÓRIO e primeiro
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

> Se você optou pelo registro global do Passo 5, acrescente `.use(GovBRDS)` antes do `.mount()`.

> Se o `src/App.vue` vier do repositório (Passo 9), garanta que ele contenha `<router-view />`.
> Se preferir um layout base próprio (header + footer), use o modelo do Apêndice C.

---

## 7. Rotas — `src/router/index.ts`

Este arquivo será **reescrito no Passo 9.3** com base nas views realmente baixadas.
Por ora, deixe um esqueleto mínimo:

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [{ path: '/', name: 'home', component: HomeView }],
})

export default router
```

---

## 8. Sanity check da base

```bash
npm run dev
```

Confirme que sobe sem erro (mesmo que a Home ainda seja a padrão do create-vue). Depois pare o servidor (Ctrl+C).

---

## 9. Baixar e ADAPTAR as páginas oficiais (o coração do modo híbrido)

### 9.1 Baixar o `src/` oficial

Baixe o repositório oficial num diretório temporário e copie apenas as pastas de conteúdo
(`views`, `components` e, se existirem, `assets`/`layouts`). **Não** copie `main.ts`/`main.js`, `router`
nem configs — esses você controla nos passos acima.

**Método principal (git clone raso):**

```bash
git clone --depth 1 \
  https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue.git \
  tmp-oficial

# Copia só o conteúdo de exemplo (ignora o que não existir)
cp -r tmp-oficial/src/views      ./src/ 2>/dev/null || true
cp -r tmp-oficial/src/components ./src/ 2>/dev/null || true
cp -r tmp-oficial/src/layouts    ./src/ 2>/dev/null || true
cp -r tmp-oficial/src/assets/*   ./src/assets/ 2>/dev/null || true

# Guarde referências úteis (para 9.3) antes de apagar
cat tmp-oficial/src/router/index.* 2>/dev/null || true
cat tmp-oficial/src/App.vue         2>/dev/null || true

rm -rf tmp-oficial
```

**Alternativa (tarball, sem git):**

```bash
curl -L -o oficial.tar.gz \
  "https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue/-/archive/main/govbr-ds-wbc-quickstart-vue-main.tar.gz"
tar xzf oficial.tar.gz
cp -r govbr-ds-wbc-quickstart-vue-main/src/views ./src/ 2>/dev/null || true
cp -r govbr-ds-wbc-quickstart-vue-main/src/components ./src/ 2>/dev/null || true
rm -rf oficial.tar.gz govbr-ds-wbc-quickstart-vue-main
```

**Alternativa 3 (degit):** `npx degit gitlab:govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue tmp-oficial`
— use só se as anteriores falharem; o degit às vezes tropeça em subgrupos aninhados do GitLab.

> Se **nenhum** método baixar (rede/proxy bloqueando gitlab.com), use as páginas de fallback do Apêndice D
> e registre no README gerado que as páginas são fallback, não as oficiais.

### 9.2 DIAGNÓSTICO — o projeto oficial já está adequado?

**Antes de adaptar qualquer coisa, analise o que foi baixado.** O repositório oficial pode ter se modernizado
por conta própria; nesse caso, adaptar seria desnecessário (e arriscado). Inspecione os arquivos do `tmp-oficial`
(capturados em 9.1) e classifique o projeto marcando cada sinal:

**Sinais de que JÁ está moderno (não precisa adaptar):**
- `package.json` depende de `@govbr-ds/webcomponents-vue` (o wrapper).
- Build por **Vite** (`vite` nas devDeps, `vite.config.*` presente) — e **não** Vue CLI (`@vue/cli-service`).
- Campos de formulário usam `v-model` diretamente nos `<br-*>`.
- `vite.config` **não** força `isCustomElement` para `br-*` (o wrapper dispensa isso).

**Sinais de que está ANTIGO (precisa adaptar):**
- Usa Vue CLI (`@vue/cli-service`, `vue.config.js`) e/ou Webpack. *Verificado em 07/2026: o `npm install`
  do quickstart oficial ainda instala essa cadeia (`webpack-chain`, `consolidate`, `glob@7`) e reporta
  dezenas de vulnerabilidades, inclusive críticas. Confirme com `npm ls @vue/cli-service` e `npm audit`.*
- Importa Web Components crus (`@govbr-ds/webcomponents/loader`, `defineCustomElements`) sem o wrapper.
- `isCustomElement: (tag) => ...br-...` configurado.
- Formulários com `:value` + `@input="... = $event.target.value"` (ou `$event.detail`).
- Estilos antigos (`@govbr-ds/[email protected]` ou anteriores; imports de fontawesome/CSS dentro das páginas).

**Decida o caminho:**

| Diagnóstico | Ação |
|---|---|
| **Totalmente moderno** (todos os sinais "moderno", wrapper ^2 / core ^3) | **Não adapte.** Reúse as páginas como estão. Só confira versões (Passo 2) e o registro global (Passo 5). Pode até dispensar o plugin do Passo 5 se o oficial já registrar o wrapper — nesse caso, siga o método do repo. |
| **Antigo** | Aplique **todas** as transformações de 9.2.1. |
| **Misto / parcial** (ex.: já é Vite mas ainda usa web components crus) | Aplique **apenas** as transformações cujo *sinal antigo* você encontrou. Não mexa no que já está moderno. |

> **Idempotência:** cada transformação em 9.2.1 só deve ser aplicada **se o padrão antigo correspondente existir
> no arquivo**. Se o trecho já está no formato moderno, **não altere**. Rodar este guia sobre um projeto já
> adequado não deve produzir nenhuma mudança.

> **Registre a decisão** no README gerado (ex.: "páginas oficiais já modernas — reutilizadas sem adaptação",
> ou "adaptadas do padrão cru para o wrapper").

### 9.2.1 Transformações (aplicar CONDICIONALMENTE, só onde o sinal antigo existir)

Percorra cada `.vue` copiado em `src/views`, `src/components` e `src/layouts`. Para cada item abaixo,
**verifique primeiro se o padrão antigo está presente**; se já estiver moderno, pule.
**Preserve** markup, textos e a intenção de cada página — mude só o mecanismo.

1. **Binding de formulário → `v-model`.** *(Só se houver `:value` + `@input`/`@change`.)* Onde houver o par `:value` + `@input`/`@change` capturando
   `$event.target.value` (ou `$event.detail`), troque por `v-model`:
   - De:
     ```vue
     <br-input :value="nome" @input="nome = $event.target.value" />
     ```
   - Para:
     ```vue
     <br-input v-model="nome" />
     ```
   - Vale também para `br-select`, `br-textarea`, `br-checkbox`, `br-radio`, `br-switch` etc.
     Para checkbox/switch (boolean), o `v-model` liga ao estado marcado.

2. **Handlers de eventos nativos do Stencil.** *(Só se houver eventos custom no arquivo.)* Mantenha o **nome
   do evento** (ex.: `@brChange`, `onBrClick`), mas leia o valor de `$event.detail` quando aplicável.
   Não invente nomes: preserve o que estava no original.

3. **Remoção de registro manual.** *(Só se existir.)* Se algum `.vue` (ou o `App.vue` oficial) registrar web
   components manualmente (`defineCustomElements`, `import '@govbr-ds/webcomponents/...'`, registro local de
   `<br-*>`), **remova** essas linhas — o registro global do Passo 5 já cobre. Se o projeto oficial já usa o
   wrapper com um método próprio de registro, **mantenha o método dele** e não duplique.

4. **Imports de estilo antigos.** *(Só se existir dentro das páginas.)* Remova imports de CSS do padrão embutidos
   nas páginas (ex.: `@govbr-ds/core/...css`, fontawesome) — o estilo global já entra pelo Passo 5 e `index.html`.

5. **Assets/caminhos.** *(Só se quebrado.)* Ajuste imports de imagens/ícones para o novo projeto (alias `@/` →
   `src/`). Corrija apenas caminhos que dependiam da estrutura antiga.

6. **Nada de `isCustomElement`.** Confirme que nenhuma config reintroduz `isCustomElement` para `br-*`
   (a menos que o projeto oficial já opere, por escolha própria, em modo cru consistente — então respeite).

> **Regra de ouro:** se um `<br-*>` não renderizar, o problema quase sempre é (a) `isCustomElement` reativado,
> ou (b) o componente não existe nesse nome no wrapper 2.x. Cheque em <https://www.gov.br/ds/webcomponents>.

### 9.3 Reconstruir o `router` a partir das views baixadas

Liste os arquivos em `src/views` e gere `src/router/index.ts` com uma rota por view (a `HomeView.vue`
como `/`). Use o `router/index.ts` oficial capturado em 9.1 como **referência de nomes/paths**, mas com a
sintaxe do Vue Router 4 e imports relativos ao novo projeto. Exemplo de forma final:

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/', name: 'home', component: HomeView },
    // + uma entrada por view adicional encontrada em src/views,
    //   usando lazy import: () => import('../views/NomeView.vue')
  ],
})

export default router
```

### 9.4 Garantir o layout (`App.vue`)

- Se o `App.vue` **oficial** foi copiado e traz header/footer gov.br + `<router-view />`, mantenha-o
  (após aplicar 9.2). 
- Se preferir/necessário, use o layout base do **Apêndice C** (header + navegação por `router-link` + footer).

---

## 10. Rodar e validar

```bash
npm run dev
```

Critérios de aceite:
- Header e footer gov.br renderizam com a fonte Rawline.
- Todas as rotas geradas em 9.3 abrem sem erro.
- Campos de formulário refletem digitação via `v-model`.
- Console **sem** `Failed to resolve component: br-…`.

Se algo falhar, revise o arquivo `.vue` correspondente conforme a "Regra de ouro" (9.2.1).

**Último passo — conformidade com o `DESIGN.md`.** Ambiente pronto é só metade: rode o checklist de
acessibilidade do `DESIGN.md` (seção 14) nas páginas resultantes e confira se elas seguem o Contrato do
Agente (seção 0). Páginas herdadas do repositório oficial são didáticas e podem não cumprir tudo.

---

## Apêndice A — JavaScript em vez de TypeScript

Se a equipe não usa TS: no Passo 1 troque `--typescript` por `--no-typescript`, renomeie
`main.ts→main.js`, `vite.config.ts→vite.config.js`, `router/index.ts→.js`, e remova `lang="ts"` e as
anotações de tipo dos exemplos. Você perde a validação de props do wrapper — o restante é idêntico.

## Apêndice B — Modo "cru" (fallback, sem wrapper)

1. Em `vite.config.ts`, dentro de `vue({...})`:
   `template: { compilerOptions: { isCustomElement: (tag) => tag.includes('br-') } }`
2. Em `main.ts`:
   ```ts
   import '@govbr-ds/core/dist/core.min.css'
   import { defineCustomElements } from '@govbr-ds/webcomponents/loader'
   defineCustomElements()
   ```
3. **Sem `v-model`:** use `:value` + `@input`/`@change` com `event.target.value`.
   (Nesse modo, as páginas oficiais no padrão cru funcionam sem adaptação — pule 9.2.1.)

## Apêndice C — Layout base (`src/App.vue`)

**Não improvise o layout.** O Template Base canônico (SkipLink, Header, Breadcrumb/Menu, Conteúdo, Footer,
com os espaçamentos obrigatórios entre áreas) está no **`DESIGN.md` seção 6.4** — copie de lá.

Regras que o Passo 10 valida: Header e Footer são **obrigatórios** em toda tela; `BrSkipLink` é obrigatório
(`DESIGN.md` seção 14); o Breadcrumb, quando existir, é o **primeiro** elemento da área de conteúdo.

Para navegar entre as rotas geradas em 9.3, use `router-link` com `custom`/`v-slot` envolvendo um
`BrButton`, ou um `BrMenu` conforme `DESIGN.md` seção 12.5.

## Apêndice D — Páginas de fallback (se o download falhar)

Crie `src/views/HomeView.vue` mínima só para o projeto subir, e registre no README que são fallback:

```vue
<template>
  <section>
    <h1 class="text-weight-bold mb-2">Quickstart Vue + DSGov</h1>
    <p class="mb-4">Base modernizada (Vite + wrapper Vue). Páginas oficiais não puderam ser baixadas.</p>
    <br-card class="p-4">
      <div class="d-flex align-items-center" style="gap:1rem;">
        <br-button type="primary" @click="c++">Cliquei {{ c }}x</br-button>
        <br-input v-model="nome" label="Seu nome" placeholder="Digite..." />
      </div>
      <p class="mt-3">Olá, {{ nome || 'visitante' }}!</p>
    </br-card>
  </section>
</template>
<script setup>
import { ref } from 'vue'
const c = ref(0)
const nome = ref('')
</script>
```

## Apêndice E — Autocomplete de tags no VS Code

`.vscode/settings.json`:

```json
{ "html.customData": ["./node_modules/@govbr-ds/webcomponents/dist/webcomponents.html-custom-data.json"] }
```

## Referências

- **[`DESIGN.md`](./DESIGN.md) — documento principal da iniciativa** (tokens, API dos componentes, padrões
  de tela, acessibilidade, microcopy). Este guia apenas prepara o ambiente onde ele será aplicado.
- **[`AGENTS.md`](./AGENTS.md)** — ponto de entrada para agentes de IA (ordem de leitura).
- **[`PADRAO_IMPLEMENTACAO_GOVBR.md`](./PADRAO_IMPLEMENTACAO_GOVBR.md)** — estado, eventos e testes E2E.
- Design System gov.br: <https://www.gov.br/ds>
- Web Components (docs/playground): <https://www.gov.br/ds/webcomponents>
- Quickstart oficial (fonte das páginas): <https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue>
- Wrapper Vue (npm): `@govbr-ds/webcomponents-vue`
