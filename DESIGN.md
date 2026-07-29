# Design System — GovBR-DS (Padrão Digital de Governo) para Vue 3 + Web Components

> **Versão:** 3.0 — escrito a partir da [documentação oficial do GovBR-DS](https://www.gov.br/ds) e da API real de `@govbr-ds/webcomponents` / `@govbr-ds/webcomponents-vue` `2.0.0-next.70`. Fontes completas na [Seção 19](#19-referências).
> **Finalidade:** Única Fonte da Verdade para geração de telas de alta fidelidade por agentes de IA (Claude / Cursor / Copilot). Cada valor deste documento é rastreável à documentação oficial ou aos tipos `.d.ts` da biblioteca.
> **Idioma da UI:** Português do Brasil. Toda label, mensagem e microcopy gerada deve seguir a Seção 15.

---

## Sumário

| # | Seção | Use quando |
| :-- | :-- | :-- |
| 0 | [Contrato do Agente](#0-contrato-do-agente) | **Sempre leia primeiro** |
| 1 | [Instalação e Bootstrap](#1-instalação-e-bootstrap) | Configurar projeto |
| 2 | [Princípios e Atmosfera](#2-princípios-e-atmosfera) | Decisões de estilo |
| 3 | [Cores](#3-cores) | Qualquer cor |
| 4 | [Tipografia](#4-tipografia) | Qualquer texto |
| 5 | [Espaçamento](#5-espaçamento) | Margin / padding |
| 6 | [Grid e Layout](#6-grid-e-layout) | Estrutura de página |
| 7 | [Superfície](#7-superfície) | Bordas, cantos, opacidade |
| 8 | [Elevação e Camadas](#8-elevação-e-camadas) | Sombras, z-index |
| 9 | [Estados](#9-estados) | Foco, hover, erro… |
| 10 | [Iconografia](#10-iconografia) | Qualquer ícone |
| 11 | [Densidade](#11-densidade) | Telas densas/amplas |
| 12 | [Referência de Componentes](#12-referência-de-componentes) | Escrever markup |
| 13 | [Padrões de Tela](#13-padrões-de-tela) | Montar uma view inteira |
| 14 | [Acessibilidade](#14-acessibilidade) | Checklist obrigatório |
| 15 | [Writing e Microcopy](#15-writing-e-microcopy) | Escrever textos |
| 16 | [Do's e Don'ts](#16-dos-e-donts) | Revisão final |
| 17 | [Armadilhas Stencil + Vue](#17-armadilhas-stencil--vue) | Debug |
| 18 | [Receitas de Prompt](#18-receitas-de-prompt) | Pedir telas |
| 19 | [Referências](#19-referências) | Consultar a fonte oficial |

---

## 0. Contrato do Agente

Regras não negociáveis ao gerar qualquer código para este projeto:

1. **Use os componentes `Br*` do pacote Vue.** Importe de `@govbr-ds/webcomponents-vue`. Só escreva HTML/CSS cru quando não existir componente equivalente na Seção 12.
2. **Nunca invente props.** A Seção 12 lista a API real extraída dos tipos da biblioteca. Prop que não está lá não existe. Em especial: `<BrButton>` **não** tem `circle` nem `block` — tem `shape="circle" | "block" | "pill"`.
3. **Nunca escreva hexadecimal.** Use classe utilitária (`.text-primary`, `.bg-gray-2`) ou token CSS (`var(--blue-warm-vivid-70)`).
4. **Nunca escreva `px` em CSS customizado.** Use tokens de espaçamento/tipografia ou classes utilitárias.
5. **Nunca remova o foco.** `outline: none` sem substituto equivalente é violação de acessibilidade e do eMAG.
6. **A fonte base é 14px, não 16px.** Toda a escala tipográfica deriva disso (Seção 4).
7. **Não importe outra biblioteca de UI.** Bootstrap, Tailwind, Vuetify, PrimeVue e afins são proibidos — o `@govbr-ds/core` já entrega grid, utilitários e reset.
8. **Toda tela vive no Template Base:** Header (obrigatório) → Conteúdo (obrigatório, com Breadcrumb e Menu opcionais) → Footer (obrigatório). Ver Seção 6.4.
9. **Português, voz ativa, singular.** Ver Seção 15.

---

## 1. Instalação e Bootstrap

### 1.1 Dependências

```jsonc
// package.json
"dependencies": {
  "@govbr-ds/core": "^3.6.1",                  // CSS: tokens, grid, utilitários
  "@govbr-ds/webcomponents": "2.0.0-next.70",  // Web Components (Stencil)
  "@govbr-ds/webcomponents-vue": "2.0.0-next.70", // Wrappers Vue 3
  "vue": "^3.3.4",
  "vue-router": "^4.2.2"
}
```

### 1.2 Entrada da aplicação

```ts
// src/main.ts
import '@govbr-ds/core/dist/core.min.css' // OBRIGATÓRIO e primeiro
import { createApp } from 'vue'
import App from './App.vue'
import router from './routes'

createApp(App).use(router).mount('#app')
```

### 1.3 Fontes e ícones (`index.html`)

A fonte Rawline e o Font Awesome **não** vêm no pacote — devem ser carregados no HTML:

```html
<html lang="pt-BR">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- Rawline: fonte oficial do Governo Federal -->
    <link rel="stylesheet"
          href="https://cdngovbr-ds.estaleiro.serpro.gov.br/design-system/fonts/rawline/css/rawline.css" />
    <!-- Raleway: fallback oficial -->
    <link rel="stylesheet"
          href="https://fonts.googleapis.com/css?family=Raleway:300,400,500,600,700,800,900&display=swap" />
    <!-- Font Awesome 5 (o DS usa a 5.10.2; 5.15.4 é compatível) -->
    <link rel="stylesheet"
          href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css" />
  </head>
</html>
```

> `lang="pt-BR"` é requisito de acessibilidade (leitores de tela escolhem o sintetizador de voz por ele).

### 1.4 Importação de componentes

```vue
<script setup lang="ts">
import { BrButton, BrInput, BrMessage } from '@govbr-ds/webcomponents-vue'
</script>
```

No template, ambas as formas funcionam. **Prefira PascalCase** — o Vue resolve como componente registrado e valida props:

```vue
<BrButton emphasis="primary">Salvar</BrButton>  <!-- preferido -->
<br-button emphasis="primary">Salvar</br-button> <!-- também funciona -->
```

---

## 2. Princípios e Atmosfera

O GovBR-DS estabelece a identidade visual dos serviços públicos digitais federais. A atmosfera é **institucional, transparente, limpa e democrática** — transmite confiança e autoridade sem sacrificar usabilidade.

Os quatro princípios que atravessam todos os fundamentos:

| Princípio | O que significa na prática |
| :-- | :-- |
| **Experiência Única** | O cidadão reconhece um sistema do governo pela cor, tipografia e componentes. Não personalize sem necessidade. |
| **Eficiência e Clareza** | Cada recurso visual tem função. Cor indica estado e hierarquia — nunca é decoração. |
| **Acessibilidade** | Nível **AA da WCAG 2.1 é o mínimo** (contraste 4.5:1 texto normal, 3:1 texto grande e elementos gráficos). |
| **Reutilização e Colaboração** | Use o que já existe. Ampliar o DS exige validação da equipe de design. |

### 2.1 Caráter visual

- **Flat com propósito.** Sombra existe para comunicar hierarquia de camada (Seção 8), nunca para "enfeitar".
- **Conteúdo em primeiro plano.** Tipografia estruturada e grid definido; dados e serviços são o protagonista.
- **Identidade unificada.** Header e Footer oficiais em todas as views.

### 2.2 Fundo claro vs. fundo escuro (o modelo real de temas)

> ⚠️ **Correção importante.** O GovBR-DS **não possui** um modo "alto contraste" ativado por uma classe `.high-contrast`. Se você viu isso em uma versão anterior deste documento, era incorreto. O modelo real é **fundo claro vs. fundo escuro**, e componentes que suportam superfície escura expõem a prop `colorMode="dark"` (ou `theme="dark"` no Footer, `isDarkMode` no Divider).

Regra para classificar um fundo: *se a cor de texto prevista é escura, o fundo é claro; se é clara, o fundo é escuro.*

| Função | Fundo claro | Fundo escuro |
| :-- | :-- | :-- |
| Superfície principal | `--pure-0` `#FFFFFF` | `--blue-warm-vivid-90` `#071D41` |
| Superfície alternativa | `--gray-2` `#F8F8F8` | `--blue-warm-vivid-80` `#0C326F` |
| Texto / ícone | `--gray-80` `#333333` | `--pure-0` `#FFFFFF` |
| Interativo | `--blue-warm-vivid-70` `#1351B4` | `--blue-warm-20` `#C5D4EB` |
| Foco | `--gold-vivid-40` `#C2850C` | `--gold-vivid-20` `#FFBE2E` |

Componentes com suporte a fundo escuro: `BrButton`, `BrSignIn`, `BrCarousel`, `BrPagination`, `BrTab`, `BrTabItem` (`colorMode="dark"`); `BrFooter` (`theme="dark" | "light"`); `BrDivider` (`isDarkMode`).

---

## 3. Cores

O sistema de cores é adaptado do U.S. Web Design System. Uma cor é nomeada por **família + luminância** (`Blue Warm Vivid 70`), e o token CSS é o kebab-case correspondente (`--blue-warm-vivid-70`).

### 3.1 Funções das cores

O DS não pensa em "cor primária/secundária" — pensa em **função**:

| Função | Papel | Fundo claro | Fundo escuro |
| :-- | :-- | :-- | :-- |
| **Container** (superfície) | Planos de fundo de tela e componentes | `--pure-0` (P) / `--gray-2` (A) | `--blue-warm-vivid-90` (P) / `--blue-warm-vivid-80` (A) |
| **Leitura** (tipografia/ícone) | Texto que precisa de legibilidade | `--gray-80` | `--pure-0` |
| **Interação** | Sinaliza que algo é clicável | `--blue-warm-vivid-70` | `--blue-warm-20` |
| **Aviso** | Feedback de sistema | ver 3.2 | ver 3.2 |

*(P) = cor principal, (A) = alternativa.*

### 3.2 Cores de feedback (estados de aviso)

Estas quatro famílias são **reservadas** — não as use para outros fins.

| Estado | Família | Principal | Alternativa (fundo suave) | Ícone Font Awesome |
| :-- | :-- | :-- | :-- | :-- |
| **Informativo** | `Blue Warm Vivid` | `--blue-warm-vivid-60` `#155BCB` | `--blue-warm-vivid-10` `#D4E5FF` | `fa-info-circle` |
| **Sucesso** | `Green Cool Vivid` | `--green-cool-vivid-50` `#168821` | `--green-cool-vivid-5` `#E3F5E1` | `fa-check-circle` |
| **Alerta** | `Yellow Vivid` | `--yellow-vivid-20` `#FFCD07` | `--yellow-vivid-5` `#FFF5C2` | `fa-exclamation-triangle` |
| **Erro** | `Red Vivid` | `--red-vivid-50` `#E52207` | `--red-vivid-10` `#FDE0DB` | `fa-times-circle` |

> Cor **nunca** é o único indicador. Todo feedback carrega ícone **e** texto.

### 3.3 Tokens semânticos de tema

Ao invés de referenciar a paleta diretamente, prefira os tokens semânticos — eles seguem o tema ativo:

| Token | Uso |
| :-- | :-- |
| `--background`, `--background-alternative` | Superfície clara principal / alternativa |
| `--background-dark`, `--background-dark-alternative` | Superfície escura |
| `--border`, `--border-dark` | Bordas e divisores |
| `--color`, `--color-dark` | Texto |
| `--h1` … `--h6`, `--h1-dark` … `--h6-dark` | Cor por nível de título |
| `--interactive`, `--interactive-alternative` | Estado interativo |
| `--interactive-dark`, `--interactive-dark-alternative` | Idem, fundo escuro |
| `--info`, `--success`, `--warning`, `--danger` | Feedback (+ variantes `-alternative`) |
| `--focus`, `--focus-dark` | Anel de foco |
| `--selected` | Estado selecionado |
| `--active` | Estado ativo |

### 3.4 Paleta de referência (subconjunto operacional)

Famílias completas em <https://www.gov.br/ds/fundamentos-visuais/cores>. Os valores usados no dia a dia:

| Token | Hex | Papel típico |
| :-- | :-- | :-- |
| `--pure-0` | `#FFFFFF` | Superfície clara, texto em fundo escuro |
| `--pure-100` | `#000000` | Sombras, overlay/scrim |
| `--gray-2` | `#F8F8F8` | Fundo de página |
| `--gray-5` | `#F0F0F0` | Fundo do header de tabela, bloco de código |
| `--gray-10` | `#E6E6E6` | Divisores sutis |
| `--gray-20` | `#CCCCCC` | Borda padrão (`--border`) |
| `--gray-40` | `#888888` | Texto desabilitado, borda de superfície |
| `--gray-70` | `#555555` | Texto secundário |
| `--gray-80` | `#333333` | **Texto primário** |
| `--blue-warm-20` | `#C5D4EB` | Interativo em fundo escuro |
| `--blue-warm-vivid-70` | `#1351B4` | **Interativo / identidade** |
| `--blue-warm-vivid-80` | `#0C326F` | Estado ativo, link visitado, superfície escura alt. |
| `--blue-warm-vivid-90` | `#071D41` | Superfície escura principal |
| `--gold-vivid-40` | `#C2850C` | **Foco (fundo claro)** |
| `--gold-vivid-20` | `#FFBE2E` | **Foco (fundo escuro)** |

### 3.5 Classes utilitárias de cor

```
bg-<cor>       → cor de fundo      ex.: .bg-blue-warm-vivid-70, .bg-gray-2
text-<cor>     → cor de texto      ex.: .text-gray-80, .text-pure-0
border-<cor>   → cor de borda      ex.: .border-gray-20   (só em elemento que já tem borda)
```

Estados disponíveis como cor: `interactive`, `danger`, `warning`, `success`, `info`.

```html
<div class="bg-danger">
  <p class="text-pure-0">Falha ao enviar o formulário</p>
</div>
```

---

## 4. Tipografia

Fonte oficial: **Rawline**. Fallback: `Raleway`, depois `sans-serif`.

### 4.1 A regra que mais se erra

> **A fonte base do GovBR-DS é `14px` (= `1em`), não 16px.** A escala é **Minor Third (razão 1.2)**. Todos os tamanhos devem ser expressos em `em`/`rem` — nunca `px` — para permitir zoom até 200% sem quebra.

### 4.2 Escala tipográfica

| Token | em | px | Classe utilitária |
| :-- | --: | --: | :-- |
| `--font-size-scale-up-07` | 3.583 | 50.16 | `.text-up-07` |
| `--font-size-scale-up-06` | 2.986 | 41.80 | `.text-up-06` |
| `--font-size-scale-up-05` | 2.488 | 34.84 | `.text-up-05` |
| `--font-size-scale-up-04` | 2.074 | 29.03 | `.text-up-04` |
| `--font-size-scale-up-03` | 1.728 | 24.19 | `.text-up-03` |
| `--font-size-scale-up-02` | 1.440 | 20.16 | `.text-up-02` |
| `--font-size-scale-up-01` | 1.200 | 16.80 | `.text-up-01` |
| `--font-size-scale-base` | 1.000 | **14.00** | `.text-base` |
| `--font-size-scale-down-01` | 0.833 | 11.67 | `.text-down-01` |
| `--font-size-scale-down-02` | 0.694 | 9.72 | `.text-down-02` |
| `--font-size-scale-down-03` | 0.579 | 8.10 | `.text-down-03` |

### 4.3 Pesos

| Token | Peso | Classe |
| :-- | --: | :-- |
| `--font-weight-thin` | 100 | `.text-weight-thin` |
| `--font-weight-extra-light` | 200 | `.text-weight-extra-light` |
| `--font-weight-light` | 300 | `.text-weight-light` |
| `--font-weight-regular` | 400 | `.text-weight-regular` |
| `--font-weight-medium` | 500 | `.text-weight-medium` |
| `--font-weight-semi-bold` | 600 | `.text-weight-semi-bold` |
| `--font-weight-bold` | 700 | `.text-weight-bold` |
| `--font-weight-extra-bold` | 800 | `.text-weight-extra-bold` |
| `--font-weight-black` | 900 | `.text-weight-black` |

### 4.4 Entrelinha

| Token | Valor | Quando |
| :-- | --: | :-- |
| `--font-line-height-low` | 1.15 | Títulos e textos acima da fonte base |
| `--font-line-height-medium` | 1.45 | Parágrafos e textos até a fonte base |
| `--font-line-height-high` | 1.85 | Textos que exigem respiro extra |

### 4.5 Tabela de estilos (grid 12/8 colunas)

Estes são os estilos que o `core.min.css` já aplica às tags nativas. Não os reescreva.

| Elemento | Size | Weight | Line-height | Margin-bottom | Margin-top | Extra |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| `h1` | `up-06` (41.8px) | **light (300)** | low | `4x` | — | — |
| `h2` | `up-05` (34.84px) | regular (400) | low | `2xh` | `3xh` | — |
| `h3` | `up-04` (29.03px) | medium (500) | low | `2xh` | `3xh` | — |
| `h4` | `up-03` (24.19px) | semi-bold (600) | low | `2xh` | `3xh` | — |
| `h5` | `up-02` (20.16px) | bold (700) | low | `2x` | `3xh` | `text-transform: uppercase` |
| `h6` | `up-01` (16.8px) | extra-bold (800) | low | `2x` | `3xh` | `text-transform: uppercase` |
| `p` | `up-01` (16.8px) | regular | medium | `2x` | — | — |
| `label` | `base` (14px) | semi-bold | medium | `half` | — | — |
| `input` | `up-01` | medium | low | `half` | — | — |
| `placeholder` | `base` | regular | medium | `half` | `half` | `font-style: italic` |
| `legend` | `up-01` | semi-bold | low | `2x` | `2x` | — |
| `mark` | — | — | — | — | — | fundo `--red-warm-vivid-10` |
| `code` | `base` | medium | low | — | — | monospace, fundo `--gray-5` |
| `ul`/`ol`/`dl` | `up-01` | regular | medium | `base` (item) / `2x` (lista) | — | — |

> ⚠️ **`h1` é `light` (300), não bold.** O DSGov comunica hierarquia por *tamanho*, não por peso. Um `h1` em bold destoa imediatamente do padrão.

### 4.6 Ajuste para grid de 4 colunas (mobile)

Em `Smartphone Portrait` a escala comprime. Só as propriedades que mudam:

| Elemento | Size | Weight | Margin |
| :-- | :-- | :-- | :-- |
| `h1` | `up-04` | medium | mb `2xh` |
| `h2` | `up-03` | semi-bold | — |
| `h3` | `up-02` | bold | — |
| `h4` | `up-01` | bold | mt `2x` |
| `h5` | `base` | extra-bold | mt `2x` |
| `h6` | `down-01` | — | mt `2x` |
| `p` | `base` | — | — |

### 4.7 Classes utilitárias de texto

```
Estilo de título sem a tag:  .h1 .h2 .h3 .h4 .h5 .h6 .label .legend .placeholder .input .mark
Alinhamento:                 .text-left .text-center .text-right .text-justify
Quebra:                      .text-wrap .text-nowrap .text-truncate .text-break
Transformação:               .text-lowercase .text-uppercase .text-capitalize
```

Todas aceitam breakpoint: `.text-up-01 .text-lg-up-06 .text-center .text-lg-left`.

> Use `.h2` em um `<span>` quando precisar do *visual* de h2 sem quebrar a hierarquia semântica do documento. Nunca pule níveis de heading por motivo estético.

---

## 5. Espaçamento

`box-sizing: border-box` em todo o DS. Espaçamento padrão de qualquer elemento é `0` (`--spacing-scale-default`).

### 5.1 Escala Layout (incremento 8px)

Escala principal — vale para qualquer elemento.

| Token | Valor | Alias na classe |
| :-- | --: | :-- |
| `--spacing-scale-base` | 8px | `base` ou `2` |
| `--spacing-scale-2x` | 16px | `2x` ou `3` |
| `--spacing-scale-3x` | 24px | `3x` ou `4` |
| `--spacing-scale-4x` | 32px | `4x` ou `5` |
| `--spacing-scale-5x` | 40px | `5x` ou `6` |
| `--spacing-scale-6x` | 48px | `6x` |
| `--spacing-scale-7x` | 56px | `7x` |
| `--spacing-scale-8x` | 64px | `8x` |
| `--spacing-scale-9x` | 72px | `9x` |
| `--spacing-scale-10x` | 80px | `10x` |

### 5.2 Escala Ajuste (incremento 4px)

> **Restrição:** apenas **texto e ícone** podem usar a escala Ajuste.

| Token | Valor | Alias |
| :-- | --: | :-- |
| `--spacing-scale-half` | 4px | `half` ou `1` |
| `--spacing-scale-baseh` | 12px | `baseh` |
| `--spacing-scale-2xh` | 20px | `2xh` |
| `--spacing-scale-3xh` | 28px | `3xh` |
| `--spacing-scale-4xh` | 36px | `4xh` |
| `--spacing-scale-5xh` | 44px | `5xh` |

### 5.3 Classes utilitárias

Formato: `{p|m}{direção|sentido}{-breakpoint}-{tamanho}`

```
p = padding          m = margin (aceita também  -auto)
direção:  t (top)  r (right)  b (bottom)  l (left)
sentido:  x (horizontal)  y (vertical)
breakpoint: sm | md | lg | xl   (mobile-first: aplica do breakpoint para cima)
```

```html
<div class="p-3x">Padding 24px em todos os lados</div>
<div class="mb-2x mt-base">Margin-bottom 16px, margin-top 8px</div>
<div class="py-2x px-3x">Vertical 16px, horizontal 24px</div>
<div class="p-2x p-sm-4x p-lg-10x">Padding cresce com o breakpoint</div>
<div class="mx-auto">Centralizado horizontalmente</div>
```

### 5.4 Otimização de margens

Quando dois elementos se sequenciam, **não some as margens**: se forem iguais, prevalece uma; se diferentes, prevalece a maior.

---

## 6. Grid e Layout

### 6.1 Breakpoints

| Dispositivo | Faixa (px) | Breakpoint | Colunas | Gutter | Margem |
| :-- | :-- | --: | --: | --: | --: |
| Smartphone Portrait | 0 – 575 | `0` (xs) | **4** | 16px | 8px |
| Smartphone Landscape / Tablet Portrait | 576 – 991 | `576` (sm) | **8** | 24px | 40px |
| Tablet Landscape | 992 – 1279 | `992` (md) | **8** | 24px | 40px |
| Desktop | 1280 – 1599 | `1280` (lg) | **12** | 24px | 40px |
| TV | 1600+ | `1600` (xl) | **12** | 40px | 40px |

> Ainda que o CSS ofereça 12 classes de coluna em todos os breakpoints, o *design* deve respeitar 4 / 8 / 12 conforme a tabela.

### 6.2 Containers

| Classe | xs | sm | md | lg | xl |
| :-- | --: | --: | --: | --: | --: |
| `.container` / `.container-sm` | 100% | 536px | 912px | 1200px | 1520px |
| `.container-md` | 100% | 100% | 912px | 1200px | 1520px |
| `.container-lg` | 100% | 100% | 100% | 1200px | 1520px |
| `.container-xl` | 100% | 100% | 100% | 100% | 1520px |
| `.container-fluid` | 100% | 100% | 100% | 100% | 100% |

**Fixa vs. fluida:** use container fixo para conteúdo informativo (portais, notícias) e fluido para sistemas que precisam aproveitar a tela.

### 6.3 Linhas e colunas

Regras absolutas — quebrá-las quebra o layout:

| Elemento | Certo ✅ | Errado ❌ |
| :-- | :-- | :-- |
| Container | — | Container dentro de container |
| Linha (`.row`) | Dentro de container; dentro de coluna | Fora de container; dentro de outra linha |
| Coluna (`.col*`) | Dentro de uma linha | Fora de linha; dentro de outra coluna |

```
.col                     → proporcional (divide o espaço)
.col-1 … .col-12         → largura predefinida  (coluna / 12) * 100%
.col-auto                → largura do conteúdo
.col-{sm|md|lg|xl}[-N]   → mesmo comportamento a partir do breakpoint
```

```html
<div class="container-lg">
  <div class="row">
    <div class="col-12 col-md-6 col-lg-4">A</div>
    <div class="col-12 col-md-6 col-lg-4">B</div>
    <div class="col-12 col-lg-4">C</div>
  </div>
</div>
```

> **Evite** criar `.row > .col-12` quando o conteúdo ocupa a largura toda — é ruído estrutural. Só crie linha/coluna quando houver divisão real.

### 6.4 Template Base (esqueleto obrigatório de toda tela)

| # | Área | Componente | Obrigatório |
| :-- | :-- | :-- | :-- |
| 1 | Cabeçalho | `BrHeader` | ✅ |
| 2 | Conteúdo | — | ✅ |
| 3 | Localização | `BrBreadcrumb` | Opcional (primeiro item do conteúdo) |
| 4 | Navegação | `BrMenu` | Opcional (à esquerda do conteúdo) |
| 5 | Rodapé | `BrFooter` | ✅ |

Espaçamentos mínimos entre áreas:

| Área | Propriedade | Token |
| :-- | :-- | :-- |
| Cabeçalho | `margin-bottom` | `--spacing-scale-3x` (24px) |
| Navegação | `margin-right` | `--spacing-scale-3x` (24px) |
| Localização | `margin-bottom` | `--spacing-scale-3x` (24px) |
| Rodapé | `margin-top` | `--spacing-scale-5x` (40px) |

```vue
<!-- src/App.vue -->
<template>
  <BrSkipLink>
    <BrSkiplinkItem target="#main-content">Ir para o conteúdo principal</BrSkiplinkItem>
    <BrSkiplinkItem target="#main-navigation">Ir para o menu</BrSkiplinkItem>
  </BrSkipLink>

  <AppHeader class="mb-3x" @toggle-menu="isMenuVisible = !isMenuVisible" />

  <main class="d-flex flex-fill mb-5x" id="main">
    <div class="container-fluid d-flex">
      <div class="row">
        <AppMenu v-show="isMenuVisible" id="main-navigation" class="mr-3x" />
        <div class="col mb-5x">
          <AppBreadcrumb class="mb-3x" />
          <div class="main-content" id="main-content">
            <router-view />
          </div>
        </div>
      </div>
    </div>
  </main>

  <AppFooter class="mt-5x" />
</template>
```

### 6.5 Sangria (bleed)

Elementos **puramente visuais** (fundo, container decorativo) podem invadir a margem da grid. **Conteúdo informativo sempre respeita a margem.** Header e Footer sangram por padrão.

### 6.6 Utilitários de layout

```
Display:   .d-none .d-block .d-flex .d-inline .d-inline-block .d-inline-flex   (+ breakpoint: .d-lg-flex)
Overflow:  .overflow-auto .overflow-hidden                                      (+ breakpoint)
```

---

## 7. Superfície

Superfície é qualquer forma indivisível que contém elementos — a base de todo componente.

### 7.1 Bordas

| Token de largura | px |
| :-- | --: |
| `--surface-width-none` | 0 |
| `--surface-width-sm` | 1 |
| `--surface-width-md` | 2 |
| `--surface-width-lg` | 4 |

Estilos: `solid` | `dashed`. Cor padrão de borda de superfície: `--gray-40`.

Tokens compostos: `--surface-border-{solid|dashed}-{none|sm|md|lg}`

Classes utilitárias:

```
.border-solid-none  .border-solid-sm  .border-solid-md  .border-solid-lg
.border-dashed-none .border-dashed-sm .border-dashed-md .border-dashed-lg
```

### 7.2 Cantos (arredondamento)

| Token | px | Classe |
| :-- | --: | :-- |
| `--surface-rounder-none` | 0 | `.rounder-none` |
| `--surface-rounder-sm` | 4 | `.rounder-sm` |
| `--surface-rounder-md` | 8 | `.rounder-md` |
| `--surface-rounder-lg` | 16 | `.rounder-lg` |
| `--surface-rounder-pill` | altura ÷ 2 | `.rounder-pill` |

> **Botões usam `--surface-rounder-pill`.** O botão padrão do GovBR-DS é uma pílula, não um retângulo de canto suave.

### 7.3 Opacidade

| Token | Valor | Uso típico |
| :-- | --: | :-- |
| `--surface-opacity-none` | 0 | Invisível |
| `--surface-opacity-xs` | 0.16 | Overlay de hover (fundo claro), sombra |
| `--surface-opacity-sm` | 0.30 | Overlay de hover (fundo escuro), dropzone ativo |
| `--surface-opacity-md` | 0.45 | **Desabilitado**, pressionado (claro), scrim |
| `--surface-opacity-lg` | 0.65 | Pressionado (fundo escuro) |
| `--surface-opacity-xl` | 0.85 | Arrastando |
| `--surface-opacity-default` | 1 | Opaco |

### 7.4 Overlay

| Token | Composição | Uso |
| :-- | :-- | :-- |
| `--surface-overlay-scrim` | `--pure-100` @ `--surface-opacity-md` | Foco em modal / área (componente `BrScrim`) |
| `--surface-overlay-text` | `linear-gradient(180deg, --pure-0 @ 0% → --pure-100 @ 100%)` | Legibilidade de texto sobre imagem |

> Evite overlay colorido sobre superfície de outra cor — cria matizes fora da paleta e corrói a identidade visual.

---

## 8. Elevação e Camadas

A elevação é organizada em **5 camadas**. Só o `offset` varia por camada; `blur` (`--surface-blur-lg` = 6px), cor (`--pure-100`) e opacidade (`--surface-opacity-xs` = 0.16) são fixos.

| Camada | Offset | Token de camada | Classe sombra | Componentes |
| :-- | --: | :-- | :-- | :-- |
| **0** | 0 | `--z-index-layer-0` | `.shadow-none` | Formulários (input, checkbox, radio, textarea, switch, upload), ações (button, link, sign-in, tag), texto/ícone/divider, informativos (table, breadcrumb, list, pagination, message, loading), navegação (tabs, wizard), footer |
| **1** | ±1 | `--z-index-layer-1` | `.shadow-sm` | **Header**, menu fixo, **Card**, magic button, *content overflow* |
| **2** | ±3 | `--z-index-layer-2` | `.shadow-md` | DateTimePicker, notification, select aberto |
| **3** | ±6 | `--z-index-layer-3` | `.shadow-lg` | Menu flutuante, sticky header, sticky footer |
| **4** | ±9 | `--z-index-layer-4` | `.shadow-xl` | **Modal**, tooltip, cookieBar, loading com superfície |

Classes de camada: `.layer-0` … `.layer-4`.

Variações direcionais das sombras (o offset Y padrão é positivo = sombra inferior):

```
.shadow-{sm|md|lg|xl}                → padrão (inferior)
.shadow-{…}-up / -right / -left      → direção alternativa
.shadow-{…}-inset[-up|-right|-left]  → sombra interna
```

Tokens de offset: `--surface-offset-{none|sm|md|lg|xl}` = 0/1/3/6/9px (sufixo `-n` para negativo).
Tokens de blur: `--surface-blur-{none|sm|md|lg|xl}` = 0/1/3/6/9px.

> Componentes da camada 0 **não projetam sombra**. Um card com sombra pesada, um input com sombra ou um botão elevado são erros comuns e imediatamente visíveis.

---

## 9. Estados

Estados são feedback visual. Regra de herança: quando vários estados coexistem, **todos** os estilos se aplicam, com prioridade para o estado gerado por interação direta do usuário.

### 9.1 Foco — a regra suprema

```css
/* Foco padrão (alta ênfase) */
border-width: var(--surface-width-lg);   /* 4px  */
border-style: dashed;
padding:      var(--spacing-scale-half); /* 4px de segurança */
border-color: var(--gold-vivid-40);      /* #C2850C — fundo claro  */
/* fundo escuro: var(--gold-vivid-20)  #FFBE2E */
```

**Foco tênue** (quando o foco chega *indiretamente*, ex.: clique num input): `--surface-width-md` (2px), `solid`, sem padding de segurança.

Cuidados:
- Só um elemento em foco por vez; só um tipo de foco por elemento.
- Elemento desabilitado não recebe foco.
- Por padrão, **clique/toque não gera foco visual** — teclado e voz sim.

### 9.2 Tabela de estados

| Estado | Especificação |
| :-- | :-- |
| **Interativo** | Cor `--blue-warm-vivid-70` (claro) / `--blue-warm-20` (escuro). Hiperlink com `underline` — dispense o sublinhado se prejudicar a legibilidade (dentro de List, textos longos). Link externo recebe `fa-external-link-alt`. |
| **Hover** | Overlay na cor do primeiro plano: opacidade `xs` (0.16) em fundo claro, `sm` (0.30) em fundo escuro. `cursor: pointer`. Um por vez na tela. |
| **Pressionado** | Overlay com opacidade `md` (0.45) claro / `lg` (0.65) escuro. `cursor: pointer`. **Remove a sombra.** |
| **Desabilitado** | `opacity: --surface-opacity-md` (0.45), `cursor: not-allowed`, sem sombra. Não herda outros estados (exceto interativo). |
| **Ativo** | `--blue-warm-vivid-80` + `--pure-0` (máximo contraste entre fundo e primeiro plano), borda `--surface-width-lg` `solid`. Geralmente basta estilizar **uma** borda. Um por conjunto. |
| **Selecionado** | Fundo `--blue-warm-vivid-50` + elemento gráfico (`fa-check`, checkbox ou radio). |
| **Visitado** | `--blue-warm-vivid-80` (claro) / `--gray-20` (escuro). Só em hiperlink; use com cautela (privacidade). |
| **Ligado / Desligado** | `--blue-warm-vivid-40` (ligado) / `--gray-40` (desligado). |
| **Arrastar** | Ícone `fa-grip-vertical`, `cursor: grab`. |
| **Arrastando** | Borda `md` solid na cor interativa, sombra offset-Y `md` + blur `lg` na cor interativa @ opacidade `sm`, elemento a `opacity: xl` (0.85) e `rotate: -5°`, `cursor: grabbing`. |
| **Dropzone** | Borda `--surface-width-sm` `dashed` na cor interativa. |
| **Dropzone ativo** | Herda dropzone + overlay na cor interativa @ opacidade `sm`. `cursor: copy`. |
| **Erro** | `--red-vivid-50` + `fa-times-circle`. Persiste até ser resolvido. |
| **Alerta** | `--yellow-vivid-20` + `fa-exclamation-triangle`. |
| **Sucesso** | `--green-cool-vivid-50` + `fa-check-circle`. Pode se auto-dispensar. |
| **Informativo** | `--blue-warm-vivid-60` + `fa-info-circle`. |

### 9.3 Ativo vs. Selecionado

| | Ativo | Selecionado |
| :-- | :-- | :-- |
| Ação | Imediata após a escolha | Posterior, via outra ação |
| Quantidade | Um por conjunto | Um ou vários |
| Exemplo | Aba visível, alinhamento de texto | Linhas marcadas para exclusão em lote |

`Selecionado` **não** se aplica a: Menu, Tabs, Modal, Tooltip, Button, Message, Divider.

### 9.4 Evite o estado desabilitado

Se o usuário não tem como habilitar o elemento, **remova-o** da interface em vez de desabilitá-lo. Nunca desabilite: componentes de navegação (Menu, Tabs), Modal, Tooltip, Magic Button.

---

## 10. Iconografia

**Font Awesome 5.10.2**, estilos `fas` (Solid — padrão) e `fab` (Brand).

### 10.1 Tamanhos

Base: `--icon-size-base` = 16px (1em).

| Classe FA | Token | Valor |
| :-- | :-- | :-- |
| `fa-xs` | `--icon-size-xs` | 0.5em (8px) |
| `fa-sm` | `--icon-size-sm` | 0.75em (12px) |
| *(sem classe)* | `--icon-size-base` | 1em (16px) |
| `fa-lg` | `--icon-size-lg` | 1.25em (20px) |
| `fa-2x` … `fa-10x` | `--icon-size-2x` … `10x` | 2em … 10em |

### 10.2 Área mínima de interação

- **Clique (mouse):** 24 × 24px
- **Toque (touch):** 48 × 48px

Válido mesmo que o ícone seja menor — a área invisível conta.

### 10.3 Acessibilidade de ícones

```html
<!-- Decorativo: acompanha um texto que já comunica o sentido -->
<BrButton emphasis="primary">
  <i class="fas fa-check mr-base" aria-hidden="true"></i> Salvar Alterações
</BrButton>

<!-- Semântico: o ícone É a informação → o rótulo vai no elemento interativo -->
<BrButton shape="circle" emphasis="tertiary" aria-label="Excluir registro">
  <i class="fas fa-trash-alt" aria-hidden="true"></i>
</BrButton>
```

Regra: o `<i>` é **sempre** `aria-hidden="true"`; o nome acessível vive no elemento interativo (`aria-label`) ou num texto adjacente.

### 10.4 Vocabulário canônico de ícones

Use exatamente estes ícones para estas ações — a consistência entre sistemas do governo depende disso.

| Ação | Classe | Ação | Classe |
| :-- | :-- | :-- | :-- |
| Pesquisar | `fa-search` | Filtrar | `fa-sliders-h` |
| Visualizar | `fa-eye` | Não visualizar | `fa-eye-slash` |
| Editar | `fa-edit` | Excluir | `fa-trash-alt` |
| Adicionar | `fa-plus` | Subtrair | `fa-minus` |
| Confirmar | `fa-check` | Fechar | `fa-times` |
| Limpar | `fa-eraser` | Atualizar | `fa-sync` |
| Tela inicial | `fa-home` | Configurações | `fa-cog` |
| Login | `fa-user` | Menu principal | `fa-bars` |
| Acessar opções | `fa-ellipsis-v` | Ajuda | `fa-question` |
| Voltar | `fa-chevron-left` | Avançar | `fa-chevron-right` |
| Retrair | `fa-chevron-up` | Expandir | `fa-chevron-down` |
| Abrir dropdown | `fa-caret-down` | Fechar dropdown | `fa-caret-up` |
| Download | `fa-download` | Upload | `fa-upload` |
| Anexar | `fa-paperclip` | Imprimir | `fa-print` |
| Copiar | `fa-copy` | Cortar | `fa-cut` |
| Selecionar data | `fa-calendar-alt` | Marcar hora | `fa-clock` |
| Bloquear | `fa-lock` | Desbloquear | `fa-unlock` |
| Notificações | `fa-bell` | Desabilitar notif. | `fa-bell-slash` |
| Mensagem | `fa-envelope` | Mensagem lida | `fa-envelope-open` |
| Enviar | `fa-share` | Exportar/Compartilhar | `fa-share-square` |
| Diretório | `fa-folder` | Diretório aberto | `fa-folder-open` |
| Ordenação padrão | `fa-sort` | Crescente / Decrescente | `fa-sort-up` / `fa-sort-down` |
| Incluir imagem | `fa-image` | Fiscalizar | `fa-clipboard-list` |
| Densidade baixa | `fa-th-large` | Densidade alta | `fa-th` |
| Ativar contraste | `fa-adjust` | Ativar Libras | `fa-deaf` |
| Áudio ativado | `fa-volume-up` | Áudio mudo | `fa-volume-mute` |
| Habilitar | `fa-toggle-on` | Desabilitar | `fa-toggle-off` |
| Msg. sucesso | `fa-check-circle` | Msg. erro | `fa-times-circle` |
| Msg. informativa | `fa-info-circle` | Msg. alerta | `fa-exclamation-triangle` |

---

## 11. Densidade

Densidade ajusta espaçamentos internos e dimensões. A escala varia de **4 em 4px**.

| Tipo | Prop | Efeito |
| :-- | :-- | :-- |
| Alta | `density="small"` | Compacto — tabelas extensas, mobile, componente dentro de componente |
| Padrão | `density="medium"` (default) | Uso geral |
| Baixa | `density="large"` | Amplo — destaque, redução de carga cognitiva |

**Restrições mínimas:**
- Área mínima de leitura: `4px` entre conteúdo e limite da superfície.
- Área mínima de ação: `24px` (mouse) / `40px` (toque) na superfície de elementos interativos.

**Boas práticas:** mantenha a mesma densidade entre componentes vizinhos semelhantes; nunca aumente a densidade de alertas ou de elementos que pedem atenção; nunca use densidade alta em elementos de interação frequente.

Suportam `density`: `BrAvatar`, `BrButton`, `BrHeader`, `BrInput`, `BrItem`, `BrMagicButton`, `BrMenu`, `BrPagination`, `BrSignIn`, `BrSwitch`, `BrTab`, `BrTable`, `BrTag`, `BrTextarea`, `BrTooltip`.

---

## 12. Referência de Componentes

API extraída de `@govbr-ds/webcomponents` `2.0.0-next.70`. Todos os componentes aceitam `customId` (gerado automaticamente se omitido) — omitido das tabelas abaixo por brevidade.

**Convenção de nomes:** props são `camelCase` no Vue (`isLoading`, `showIcon`), que o Stencil expõe como `kebab-case` no DOM (`is-loading`, `show-icon`). Ambos funcionam no template Vue.

### 12.1 Ações

#### `BrButton`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `emphasis` | `'primary' \| 'secondary' \| 'tertiary'` | — |
| `shape` | `'circle' \| 'block' \| 'pill'` | — |
| `density` | `'small' \| 'medium' \| 'large'` | `'medium'` |
| `type` | `'button' \| 'submit' \| 'reset'` | — |
| `disabled` | `boolean` | `false` |
| `isLoading` | `boolean` | `false` |
| `isActive` | `boolean` | `false` |
| `colorMode` | `'dark'` | — |
| `ariaLabel` | `string` | — |
| `ariaPressed` | `string` | — |
| `customTabIndex` | `number` | — |
| `value` | `string` | — |

**Slots:** default (rótulo e ícone).

**Ênfases:** *primária* = ação principal da tela (use com parcimônia, idealmente uma por bloco); *secundária* = ação de importância intermediária ("Cancelar", "Salvar Rascunho"); *terciária* = baixa importância, opcionais e de suporte.

**Especificação visual:** rótulo `--font-size-scale-up-01` / `--font-weight-semi-bold`, sem `text-transform`. Ícone `--icon-size-base`, `margin-right: --spacing-scale-base`. Padding lateral `--spacing-scale-3x`. Border-radius `--surface-rounder-pill`. Altura: 48px (large) / 40px (medium) / 32px (small); circular é 1:1.

| Ênfase | Texto | Fundo | Borda |
| :-- | :-- | :-- | :-- |
| Primária | `--pure-0` | `--blue-warm-vivid-70` | — |
| Primária (fundo escuro) | `--blue-warm-vivid-90` | `--blue-warm-vivid-20` | — |
| Secundária | `--blue-warm-vivid-70` | `--pure-0` | `--blue-warm-vivid-70` |
| Secundária (fundo escuro) | `--blue-warm-vivid-20` | `--blue-warm-vivid-90` | `--blue-warm-vivid-20` |
| Terciária | `--blue-warm-vivid-70` | transparente (0%) | — |
| Terciária (fundo escuro) | `--blue-warm-vivid-20` | transparente | — |

```vue
<div class="d-flex justify-content-end">
  <BrButton emphasis="tertiary" type="button" @click="onHelp">Informações Adicionais</BrButton>
  <BrButton emphasis="secondary" type="button" class="mx-2x" @click="onCancel">Cancelar</BrButton>
  <BrButton emphasis="primary" type="submit" :is-loading="saving" :disabled="!canSubmit">
    <i class="fas fa-check mr-base" aria-hidden="true"></i> Salvar Alterações
  </BrButton>
</div>

<!-- Circular: aria-label é OBRIGATÓRIO -->
<BrButton shape="circle" emphasis="tertiary" density="small"
          aria-label="Editar registro" @click="onEdit(item.id)">
  <i class="fas fa-edit" aria-hidden="true"></i>
</BrButton>

<!-- Bloco: 100% da largura, recomendado em mobile/login -->
<BrButton emphasis="primary" shape="block" type="submit">Entrar</BrButton>
```

> **Ordem hierárquica.** Na horizontal: terciária → secundária → **primária à direita**. Na vertical: primária **embaixo**.
> Evite botões "Redefinir" e "Limpar Formulário".

#### `BrSignIn` — login gov.br

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `emphasis` | `'primary' \| 'secondary'` | — |
| `shape` | `'block' \| 'circle' \| 'pill'` | — |
| `density` | `'small' \| 'medium' \| 'large'` | — |
| `colorMode` | `'dark'` | — |
| `href` / `target` | `string` | — |
| `disabled` | `boolean` | `false` |
| `ariaLabel` | `string` | — |

**Slots:** default, `icon`, `image`.

```vue
<BrSignIn emphasis="primary" shape="block" href="/auth/govbr">Entrar com gov.br</BrSignIn>
```

#### `BrMagicButton`

| Prop | Tipo |
| :-- | :-- |
| `label` / `icon` / `ariaLabel` | `string` |
| `circle` | `boolean` |
| `density` | `'small' \| 'medium' \| 'large'` |

Use para reforçar o início ou encerramento de um fluxo relevante. **Nunca desabilite.**

#### `BrTag`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `iconName` / `name` / `value`* | `string` | — |
| `shape` | `'circle' \| 'rounded' \| 'default'` | — |
| `density` | `'small' \| 'medium' \| 'large'` | — |
| `status` | `boolean` | `false` |
| `interaction` | `boolean` | `false` |
| `interactionSelect` | `boolean` | `false` |
| `selected` / `multiple` / `disabled` | `boolean` | `false` |
| `bgColor` | `string` | — |
| `ariaLabel` / `ariaDescribedby` | `string` | — |

**Evento:** `radioSelected`. **Slot:** default.

Tipos: *interação* (dispensável/persistente), *texto*, *status*, *contagem*, *ícone*.

> **Microcopy:** rótulo de tag **não usa verbo** — use substantivo ou adjetivo. ✅ "Criação de conteúdo" · ❌ "Criar conteúdo".

### 12.2 Formulário

#### `BrInput`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `placeholder` / `name` / `value` | `string` | — |
| `type` | `'text' \| 'email' \| 'password' \| 'number' \| 'tel' \| 'url' \| 'search' \| 'color' \| 'range' \| 'file' \| 'hidden'` | `'text'` |
| `state` | `'info' \| 'warning' \| 'danger' \| 'success'` | — |
| `density` | `'small' \| 'medium' \| 'large'` | `'medium'` |
| `mask` | `string` (`#` = posição numérica) | — |
| `helpText` | `string` | — |
| `required` / `disabled` / `readonly` | `boolean` | `false` |
| `isInline` | `boolean` | `false` |
| `isHighlight` | `boolean` | `false` |
| `borderless` | `boolean` | `false` |
| `controlWidth` | `string` (ex.: `'88px'`) | — |
| `min` / `max` / `minlength` / `maxlength` / `step` | `number` | — |
| `pattern` | `string` | — |
| `autocomplete` / `autocorrect` | `'on' \| 'off'` | — |
| `actionLabel` / `actionTabIndex` | `string` / `number` | — |

**v-model:** liga em `value`. **Evento:** `valueChange`. **Slots:** `action`, `icon`, `help-text`, `feedback`.
**Métodos:** `checkValidity()`, `reportValidity()`, `setCustomValidity(msg)`.

```vue
<BrInput
  id="cpf"
  label="CPF"
  mask="###.###.###-##"
  placeholder="000.000.000-00"
  help-text="Informe apenas números"
  v-model="form.cpf"
  :state="fieldState('cpf')"
  required
  @blur="touch('cpf')"
/>
<BrMessage v-if="showError('cpf')" state="danger" is-feedback
           message="Informe um CPF válido" />
```

#### `BrTextarea`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `placeholder` / `name` / `value` / `ariaLabel` | `string` | — |
| `state` | `'danger' \| 'success' \| 'warning'` | — |
| `density` | `'small' \| 'medium' \| 'large'` | — |
| `rows` / `cols` / `maxlength` | `number` | — |
| `showCounter` | `boolean` | `false` |
| `required` / `disabled` / `isInline` | `boolean` | `false` |

**v-model:** `value`. **Evento:** `valueChange`.

> Atenção: `BrTextarea.state` **não** aceita `'info'` (`BrInput` aceita).

#### `BrSelect` / `BrSelectOption`

| `BrSelect` | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `placeholder` / `name` | `string` | — |
| `value` | `string \| string[]` | — |
| `options` | `string \| objeto` | — |
| `isMultiple` | `boolean` | `false` |
| `keepOpenOnSelect` | `boolean` | `false` |
| `showSearchIcon` | `boolean` | `false` |
| `isOpen` / `isInline` / `borderless` | `boolean` | `false` |
| `required` / `disabled` | `boolean` | `false` |
| `inputWidth` | `string` | — |
| `selectAllLabel` / `unselectAllLabel` | `string` | — |

**Eventos:** `valueChange`, `optionHover`, `opened`, `closed`. **v-model:** `value`.

| `BrSelectOption` | Tipo |
| :-- | :-- |
| `label` / `value` | `string` |
| `selected` | `boolean` |

```vue
<BrSelect id="uf" label="Unidade Federativa" placeholder="Selecione…"
          show-search-icon @value-change="onUfChange">
  <BrSelectOption v-for="uf in ufs" :key="uf.value"
                  :label="uf.label" :value="uf.value"
                  :selected="form.uf === uf.value" />
</BrSelect>
```

```ts
function onUfChange(event: CustomEvent) {
  const d = event.detail
  form.uf = Array.isArray(d) ? d[0] : (d?.value ?? d ?? '')
}
```

#### `BrCheckbox` / `BrRadio`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `name` / `value` | `string` | — |
| `checked` | `boolean` | `false` |
| `state` | `'valid' \| 'invalid'` | — |
| `required` / `disabled` | `boolean` | `false` |
| `hasHiddenLabel` | `boolean` | `false` |
| `indeterminate`¹ | `boolean` | `false` |
| `isFather`¹ | `boolean` | `false` |

¹ Só em `BrCheckbox`.

**Eventos:** `checkedChange` (+ `indeterminateChange` no checkbox). **v-model:** `checked`.

> ⚠️ `state` aqui é `'valid' | 'invalid'` — **não** `'danger'`. Confundir isso é o erro mais comum em formulários.

#### `BrCheckgroup`

| Prop | Tipo |
| :-- | :-- |
| `label` / `labelSelecionado` / `labelDesselecionado` | `string` |
| `indeterminate` | `boolean` |

#### `BrSwitch`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `labelOn` / `labelOff` / `name` / `value` | `string` | — |
| `labelPosition` | `'right' \| 'left' \| 'top'` | — |
| `checked` / `hasIcon` / `required` / `disabled` | `boolean` | `false` |
| `density` | `'small' \| 'medium' \| 'large'` | — |

**Evento:** `checkedChange`. **v-model:** `checked`.

#### `BrUpload`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `label` / `name` / `accept` | `string` | — |
| `uploadFiles` | `string \| IUploadFile[]` | — |
| `state` | `'info' \| 'warning' \| 'danger' \| 'success'` | — |
| `multiple` / `required` / `disabled` | `boolean` | `false` |

**Eventos:** `selectedFilesChange`, `brRemove`. **Slots:** default, `upload-list`.

#### `BrDatetimePicker`

| Prop | Tipo |
| :-- | :-- |
| `mode` | `DatetimePickerMode` |
| `value` | `Date \| null` |
| `name` / `placeholder` | `string` |
| `weekStartsOn` | `WeekDayIndex` |
| `autoSelectToday` / `required` / `disabled` | `boolean` |

**Evento:** `brDateTimeChange`. Vive na camada 2.

### 12.3 Feedback

#### `BrMessage`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `state` | `'info' \| 'warning' \| 'danger' \| 'success'` | — |
| `messageTitle` / `message` / `ariaLabel` | `string` | — |
| `showIcon` | `boolean` | `false` |
| `isFeedback` | `boolean` | `false` |
| `isInline` | `boolean` | `false` |
| `isClosable` | `boolean` | `false` |
| `autoRemove` | `boolean` | `false` |

**Evento:** `brDidClose`. **Slots:** default, `icon`.

Dois tipos: **padrão** (fundo suave, texto escuro) e **contextual** (`isInline` — fundo saturado, texto claro).

| Estado | Fundo padrão | Fundo contextual | Ícone |
| :-- | :-- | :-- | :-- |
| Info | `--blue-warm-vivid-10` | `--blue-warm-vivid-60` | `info-circle` |
| Sucesso | `--green-cool-vivid-5` | `--green-cool-vivid-50` | `check-circle` |
| Alerta | `--yellow-vivid-5` | `--yellow-vivid-20` | `exclamation-triangle` |
| Erro | `--red-vivid-10` | `--red-vivid-50` | `times-circle` |

Tipografia: título `up-01`/semi-bold; mensagem `up-01`/regular; contextual `base`/medium. Padding: `3x` vertical, `2x` esquerda, `base` direita.

```vue
<!-- Mensagem global no topo do formulário -->
<BrMessage v-if="alert" :state="alert.state" :message="alert.text"
           show-icon is-closable class="mb-3x d-block" />

<!-- Feedback de campo: sempre imediatamente abaixo do input -->
<BrMessage state="danger" is-feedback message="Campo obrigatório" />
```

#### `BrLoading`

| Prop | Tipo |
| :-- | :-- |
| `mode` | `'spinner' \| 'progress'` |
| `size` | `'small' \| 'medium' \| 'large'` |
| `speed` | `'slow' \| 'normal' \| 'fast'` |
| `labelPosition` | `'top' \| 'right' \| 'bottom' \| 'left'` |
| `completion` | `'persist' \| 'reset' \| 'hide'` |
| `label` / `cancelLabel` | `string` |
| `value` | `number` |
| `cancelable` | `boolean` |

**Eventos:** `brLoadingChange`, `brLoadingComplete`, `brLoadingReset`, `brLoadingCancel`, `brIndeterminateStateChange`, `brDidShow`, `brDidHide`.

#### `BrModal`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `titleText` | `string` | — |
| `show` | `boolean` | `false` |
| `size` | `'xsmall' \| 'small' \| 'medium' \| 'large' \| 'auto'` | — |
| `alignFooter` | `'start' \| 'end' \| 'center'` | — |
| `autoClose` / `scrollable` | `boolean` | `false` |
| `initialFocusSelector` | `string` | — |

**Eventos:** `brModalOpen`, `brModalClose`, `brModalBeforeClose`, `brModalOpened`. **Slots:** `header`, default, `footer`, `close-button`. Camada 4.

```vue
<BrModal :show="confirmOpen" title-text="Excluir registro"
         size="small" align-footer="end" @br-modal-close="confirmOpen = false">
  <p>Esta ação não pode ser desfeita. Deseja excluir <strong>{{ alvo.nome }}</strong>?</p>
  <template #footer>
    <BrButton emphasis="secondary" @click="confirmOpen = false">Cancelar</BrButton>
    <BrButton emphasis="primary" class="ml-2x" @click="confirmarExclusao">Excluir Registro</BrButton>
  </template>
</BrModal>
```

> **Microcopy de modal:** use o mesmo verbo da pergunta nos botões. ✅ "Cancelar / Excluir" · ❌ "Não / Sim".

#### `BrNotification` + `BrNotificationHeader` / `Body` / `Item`

| `BrNotification` | Tipo |
| :-- | :-- |
| `label` | `string` |
| `open` | `boolean` |
| `placement` | `'bottom-start' \| 'bottom-end' \| 'top-start' \| 'top-end'` |
| `closeOnOutsideClick` / `closeOnTriggerClick` | `boolean` |
| `mobileFullscreen` / `internalScroll` / `isStatic` | `boolean` |
| `width` / `maxWidth` / `maxHeight` | `string` |

**Slots:** `trigger`, `header`, `tabs`, default, `footer`.
`BrNotificationItem`: `itemId`, `label`, `description`, `datetime`, `unread`, `clickable`, `state`. Slots: `status`, `title`, `meta`, default, `actions`.

#### `BrTooltip`

| Prop | Tipo |
| :-- | :-- |
| `position` / `type` / `state` / `density` | tipos internos |
| `isAutoVisible` | `boolean` |
| `showDelay` / `hideDelay` | `number` |

**Slots:** `trigger`, `content`. Camada 4. **Nunca desabilite.**

#### `BrScrim`

| Prop | Tipo |
| :-- | :-- |
| `variant` | `'focus' \| 'spotlight' \| 'legibility'` |
| `isOpen` | `boolean` |
| `displayMode` | `'fullscreen' \| 'parent'` |
| `positionContent` | `'top' \| 'center' \| 'right' \| 'left' \| 'bottom'` |
| `scrollStrategy` | `'block' \| 'close'` |
| `spotlightShape` | `'rect' \| 'rounded' \| 'circle'` |
| `spotlightTargetId` / `activator` / `bgColor` / `ariaLabel` | `string` |
| `customOpacity` / `zIndex` / `spotlightPadding` / `scrollThreshold` | `number` |
| `disableCloseOnClick` | `boolean` |

**Slots:** default, `activator`.

### 12.4 Dados e listagem

#### `BrTable` + subcomponentes

| `BrTable` | Tipo |
| :-- | :-- |
| `density` | `'small' \| 'medium' \| 'large'` |
| `columnWidth` | `'fill' \| 'content' \| '${n}px' \| '${n}fr'` |
| `horizontalAlignment` / `verticalAlignment` | `'start' \| 'center' \| 'end'` |
| `dividerStyle` | `'solid' \| 'dashed'` |
| `hasColumnDivider` / `hasRowDivider` | `boolean` |
| `overflow` | `'hidden' \| 'visible'` |
| `tooltipState` | `'info' \| 'warning' \| 'danger' \| 'success'` |
| `tooltipMode` | `'inherit' \| 'enabled' \| 'disabled'` |

`BrTableHeaderCell` aceita `columnWidth` + alinhamentos; `BrTableRow` e `BrTableCell` aceitam alinhamentos e overflow. `BrTableHeader` e `BrTableBody` não têm props.

> ⚠️ **Alinhamento usa `start` / `center` / `end`** — não `left` / `right`.

**Cores (fundo claro):** header `background: --gray-5`, `color: --blue-warm-vivid-70`. Linhas `background: --pure-0`, `color: --gray-80`. Barra de título `--pure-0` / `--gray-80`.
**Tipografia:** barra de título `up-01`/semi-bold; header `base`/semi-bold; linhas `base`/medium.

> ⚠️ O header de tabela é **cinza claro com texto azul** — não azul sólido com texto branco.

**Alinhamento por tipo de dado:** texto e nomes → `start`; datas e status → `center`; valores monetários e quantidades → `end`.

```vue
<BrTable density="medium" has-row-divider>
  <BrTableHeader>
    <BrTableRow>
      <BrTableHeaderCell column-width="250px">Nome do Cidadão</BrTableHeaderCell>
      <BrTableHeaderCell column-width="160px">CPF</BrTableHeaderCell>
      <BrTableHeaderCell column-width="140px" horizontal-alignment="center">Solicitação</BrTableHeaderCell>
      <BrTableHeaderCell column-width="120px" horizontal-alignment="center">Situação</BrTableHeaderCell>
      <BrTableHeaderCell column-width="content" horizontal-alignment="end">Ações</BrTableHeaderCell>
    </BrTableRow>
  </BrTableHeader>
  <BrTableBody>
    <BrTableRow v-for="item in pagina" :key="item.id">
      <BrTableCell>{{ item.nome }}</BrTableCell>
      <BrTableCell>{{ item.cpf }}</BrTableCell>
      <BrTableCell horizontal-alignment="center">{{ formatarData(item.data) }}</BrTableCell>
      <BrTableCell horizontal-alignment="center">
        <BrTag status :label="item.ativo ? 'Ativo' : 'Pendente'" />
      </BrTableCell>
      <BrTableCell horizontal-alignment="end">
        <BrButton shape="circle" emphasis="tertiary" density="small"
                  aria-label="Visualizar registro" @click="ver(item.id)">
          <i class="fas fa-eye" aria-hidden="true"></i>
        </BrButton>
        <BrButton shape="circle" emphasis="tertiary" density="small"
                  aria-label="Editar registro" @click="editar(item.id)">
          <i class="fas fa-edit" aria-hidden="true"></i>
        </BrButton>
        <BrButton shape="circle" emphasis="tertiary" density="small"
                  aria-label="Excluir registro" @click="pedirExclusao(item)">
          <i class="fas fa-trash-alt" aria-hidden="true"></i>
        </BrButton>
      </BrTableCell>
    </BrTableRow>
  </BrTableBody>
</BrTable>

<BrPagination :total="totalPaginas" :current="pagina" :total-items="totalItens"
              :per-page="10" @page-change="onPageChange" />
```

> Máximo de **3 ícones de ação** por linha; acima disso, agrupe em um menu `fa-ellipsis-v`.
> Em grid de 4 colunas, a tabela deve rolar horizontalmente ou reconfigurar-se em cartões.

#### `BrPagination`

| Prop | Tipo |
| :-- | :-- |
| `total` / `current` / `totalItems` / `perPage` | `number` |
| `perPageOptions` | `number[]` |
| `variant` | `'default' \| 'contextual'` |
| `density` | `'small' \| 'medium' \| 'large'` |
| `colorMode` | `'dark'` |
| `ariaLabel` / `previousLabel` / `nextLabel` / `ellipsisLabel` / `perPageLabel` / `goToPageLabel` / `itemsText` | `string` |

**Eventos:** `pageChange`, `perPageChange`.

#### `BrList` / `BrItem`

| `BrList` | Tipo |
| :-- | :-- |
| `header` / `accordion` | `string` |
| `isHorizontal` / `collapse` / `hideHeaderDivider` | `boolean` |

| `BrItem` | Tipo |
| :-- | :-- |
| `href` / `target` / `value` | `string` |
| `type` | `'submit' \| 'reset' \| 'button'` |
| `density` | `'small' \| 'medium' \| 'large'` |
| `isActive` / `isSelected` / `isInteractive` / `isButton` / `disabled` | `boolean` |

**Eventos:** `brDidClick`, `brDidSelect`. **Slots:** default, `start`, `end`.

#### `BrCard`

| Prop | Tipo | Default |
| :-- | :-- | :-- |
| `hover` | `boolean` | `false` |
| `disabled` | `boolean` | `false` |

**Slots:** default, `header`, `content`, `footer`. **Camada 1** (`.shadow-sm`).

```vue
<BrCard hover>
  <template #header>
    <h4 class="mb-0">Solicitações no Mês</h4>
  </template>
  <template #content>
    <p class="text-up-05 text-weight-semi-bold text-primary mb-half">1.248</p>
    <p class="text-base text-success mb-0">
      <i class="fas fa-arrow-up mr-half" aria-hidden="true"></i> 12% acima do mês anterior
    </p>
  </template>
</BrCard>
```

#### `BrAvatar`

| Prop | Tipo |
| :-- | :-- |
| `src` / `alt` / `text` / `bgColor` | `string` |
| `isIconic` | `boolean` |
| `density` | `'small' \| 'medium' \| 'large'` |

Três tipos: fotográfico (`src`), letra (`text`), icônico (`isIconic`).

#### `BrDivider`

| Prop | Tipo |
| :-- | :-- |
| `orientation` / `thickness` / `borderStyle` / `align` | tipos internos |
| `color` | `string` |
| `isDarkMode` / `bleed` | `boolean` |
| `marginTop` / `marginBottom` / `marginLeft` / `marginRight` | `number` |

#### `BrCarousel` / `BrCarouselPage`

| `BrCarousel` | Tipo | Default |
| :-- | :-- | :-- |
| `indicatorType` | `'simple' \| 'textual' \| 'none'` | — |
| `indicatorPosition` / `navPosition` | `'outside' \| 'inside'` | — |
| `direction` | `'left' \| 'right'` | `'right'` |
| `autoPlay` / `isCircular` / `mobileNav` | `boolean` | `false` |
| `interval` | `number` | — |
| `colorMode` | `'dark'` | — |
| `ariaLabel` | `string` (não use a palavra "carrossel") | — |

**Eventos:** `brDidPageChange`, `brDidAutoPlayStart`, `brDidAutoPlayPause`.

### 12.5 Navegação e estrutura

#### `BrHeader` + subcomponentes

| `BrHeader` | Tipo |
| :-- | :-- |
| `caption` / `captionUrl` / `subcaption` / `subcaptionUrl` / `signature` | `string` |
| `density` | `HeaderDensity` |
| `isCompact` / `isSticky` | `boolean` |
| `shrinkFirst` | `'links' \| 'functions'` |

**Slots:** `logo`, `signature`, `links`, `functions`, `search`, `access`, `menu-trigger`, `caption`, `subcaption`.
**Eventos:** `headerCompactChange`, `headerWidthChange`. **Camada 1** (ou 3 se `isSticky`).

Subcomponentes: `BrHeaderLogo` (`src`, `description`, `href`, `width`, `height`, `isCompact`), `BrHeaderLink` (`href`, `target`), `BrHeaderFunction` (`href`, `iconName`, `target`), `BrHeaderList` (`listTitle`).

#### `BrFooter` + subcomponentes

| `BrFooter` | Tipo |
| :-- | :-- |
| `theme` | `'dark' \| 'light'` |
| `layoutWidth` | `'contained' \| 'full'` |
| `socialNetworkTitle` | `string` |

**Slots:** `logo`, default (categorias), `social-network`, `partner-logo`, `legal`.

```vue
<BrFooter theme="dark">
  <BrFooterLogo slot="logo" :src="logoNegativo" description="Logotipo do órgão" />

  <BrFooterCategory v-for="cat in categorias" :key="cat.label" :label="cat.label">
    <BrFooterItem v-for="it in cat.items" :key="it.href" :href="it.href">{{ it.text }}</BrFooterItem>
  </BrFooterCategory>

  <BrFooterSocial v-for="s in redes" :key="s.href" slot="social-network"
                  :href="s.href" :icon="s.icon" :description="s.description" />

  <BrFooterLegal slot="legal">
    <p class="mb-0">Texto sobre a <strong>licença de uso</strong>.</p>
  </BrFooterLegal>
</BrFooter>
```

> Os slots nomeados dos Web Components usam o **atributo `slot="nome"`** no filho (não `v-slot`), porque a projeção acontece no Shadow DOM. Ver Seção 17.

#### `BrMenu` + subcomponentes

| `BrMenu` | Tipo |
| :-- | :-- |
| `isOpen` / `push` / `contextual` | `boolean` |
| `breakpoints` / `socialTitle` / `contextualLabel` | `string` |
| `density` | `'small' \| 'medium' \| 'large'` |

**Slots:** default, `header`, `logo`, `link`, `social`, `info`.
Subcomponentes: `BrMenuHeader` (`logoSrc`, `logoAlt`, `signature`), `BrMenuItem` (`icon`, `href`, `target`, `divider`, `active`, `expanded`, `displayMode: 'accordion' | 'drill-down' | 'none'`), `BrMenuList` (`icon`, `label`, `menuLevel`, `divider`, `expanded`), `BrMenuLink`, `BrMenuLogo`, `BrMenuSocial`, `BrMenuInfo`.

Menu **flutuante** (camada 3) não afeta a grid; menu **persistente** empurra o conteúdo. Sempre à esquerda em desktop; pode ir para a base em mobile.

#### `BrBreadcrumb` / `BrCrumb`

| `BrBreadcrumb` | Tipo |
| :-- | :-- |
| `crumbs` | `string \| BreadcrumbItem[]` |
| `homeUrl` | `string` |

| `BrCrumb` | Tipo |
| :-- | :-- |
| `label` / `url` / `target` | `string` |
| `home` / `active` | `boolean` |

Quando presente, é o **primeiro** elemento da Área de Conteúdo. Dispensável em telas iniciais e mobile.

#### `BrSkipLink` / `BrSkiplinkItem`

| `BrSkipLink` | Tipo |
| :-- | :-- |
| `variant` | `'simple' \| 'compound'` |
| `position` | `'top-left' \| 'top-center' \| 'top-right'` |
| `showItemCount` | `boolean` |
| `zIndex` | `number` |

| `BrSkiplinkItem` | Tipo |
| :-- | :-- |
| `target` | `string` (seletor/âncora) |
| `keyNumber` | `number` |
| `hideTag` | `boolean` |

**Obrigatório** em toda aplicação — âncoras para conteúdo principal, navegação e áreas de apoio.

#### `BrTab` / `BrTabItem`

| `BrTab` | Tipo |
| :-- | :-- |
| `label` / `height` | `string` |
| `density` / `alignItemsTab` / `colorMode` | tipos internos |
| `scrollDisabled` | `boolean` |

| `BrTabItem` | Tipo |
| :-- | :-- |
| `tabItemId` / `tabItemTitle` / `icon` / `counter` | `string` |
| `isActive` / `onlyIcon` | `boolean` |
| `colorMode` | `'dark'` |

**Evento:** `brTabChange`. Nunca desabilite abas — oculte-as.

#### `BrStep` / `BrStepItem`

| `BrStep` | Tipo |
| :-- | :-- |
| `layout` | `'horizontal' \| 'vertical'` |
| `mode` | `'controller' \| 'step'` |
| `progressionType` | `'linear' \| 'nonlinear'` |
| `contentType` | `'default' \| 'br-icon' \| 'no-content' \| 'slotted'` |
| `labelPosition` | `'top' \| 'right' \| 'bottom' \| 'left'` |
| `initialStep` | `string` |

| `BrStepItem` | Tipo |
| :-- | :-- |
| `label` / `ariaLabel` / `controlsId` / `brIconName` / `brIconAria` | `string` |
| `highlight` | `'success' \| 'info' \| 'danger' \| 'warning'` |
| `active` / `disabled` | `boolean` |

**Evento:** `brStepChange`. **Slot de `BrStepItem`:** `content-area`.

#### `BrWizard` / `BrWizardPanel`

| `BrWizard` | Tipo |
| :-- | :-- |
| `initialStep` | `number` |
| `orientation` | `'horizontal' \| 'vertical'` |
| `progressionType` | `'linear' \| 'nonlinear'` |

**Eventos:** `brWizardStepChange`, `brWizardBeforeStepChange`, `brWizardComplete`, `brWizardCancel`, `brWizardNavigationBlocked`.
`BrWizardPanel`: `label`.

#### `BrCollapse` / `BrDropdown`

| `BrCollapse` | Tipo |
| :-- | :-- |
| `open` | `boolean` |
| `accordionGroup` | `string` (agrupa em acordeão) |
| `iconToShow` / `iconToHide` | `string` |
| `iconPosition` | `'left' \| 'right'` |

**Slots:** default, `trigger`. **Eventos:** `brDidOpen`, `brDidClose`.

| `BrDropdown` | Tipo |
| :-- | :-- |
| `isOpen` | `boolean` |
| `placement` | `'bottom-start' \| 'bottom-end' \| 'top-start' \| 'top-end'` |
| `preventAutoDismiss` | `boolean` |
| `targetZIndex` | `string` |

**Slots:** `trigger`, `target`. **Eventos:** `brDropdownChange`, `brDidOpen`.

#### `BrIcon`

| Prop | Tipo |
| :-- | :-- |
| `iconName` / `width` / `height` / `cssClasses` | `string` |
| `rotate` | `'90deg' \| '180deg' \| '270deg'` |
| `flip` | `'horizontal' \| 'vertical'` |
| `isInline` / `isFocusable` / `lazy` | `boolean` |

---

## 13. Padrões de Tela

### 13.1 Formulário

**Anatomia:** Título (+ subtítulo) → Campos agrupados → Botões.

**Rótulos**
- Todo campo tem rótulo visível. **Placeholder nunca substitui rótulo.**
- Apenas a primeira letra maiúscula. Sem verbo.
- Acima do campo, próximo dele.
- Prefira que todos os campos sejam obrigatórios e diga isso no subtítulo. Se precisar sinalizar, marque **ou** os opcionais **ou** os obrigatórios — nunca ambos — com o termo entre parênteses à direita: `Telefone (Opcional)`.

**Layout**
- Prefira **coluna única**. Campos lado a lado apenas quando forem complementares (Data início/Data fim, Estado/Cidade).
- Agrupe por afinidade com `<fieldset>` + `<legend>` (`border: --surface-width-none`, `margin-bottom: --spacing-scale-5x`). O `fieldset` no DS é apenas semântico — sem borda visual.
- No máximo **um** nível de subagrupamento.

**Ritmo de espaçamento**
- Maior entre agrupamentos e antes dos botões (`5x` / 40px)
- Intermediário entre título e primeiro campo (`3x` / 24px)
- Menor entre campos (`2x` / 16px)

**Validação (lazy validation)**
Dispare o erro no `blur`; limpe durante a digitação. O primeiro campo obrigatório recebe foco ao carregar.

```vue
<template>
  <form class="container-lg" novalidate @submit.prevent="onSubmit">
    <h2>Cadastro de Solicitante</h2>
    <p class="mb-3x">Todos os campos são obrigatórios, salvo indicação em contrário.</p>

    <BrMessage v-if="alerta" :state="alerta.state" :message="alerta.texto"
               show-icon is-closable class="mb-3x d-block" />

    <fieldset class="mb-5x">
      <legend>Identificação</legend>
      <div class="row">
        <div class="col-12 col-md-6 mb-2x">
          <BrInput id="nome" label="Nome completo" v-model="form.nome" required
                   :state="estado('nome')" @blur="tocar('nome')" />
          <BrMessage v-if="erro('nome')" state="danger" is-feedback :message="erro('nome')" />
        </div>
        <div class="col-12 col-md-6 mb-2x">
          <BrInput id="cpf" label="CPF" mask="###.###.###-##" placeholder="000.000.000-00"
                   v-model="form.cpf" required :state="estado('cpf')" @blur="tocar('cpf')" />
          <BrMessage v-if="erro('cpf')" state="danger" is-feedback :message="erro('cpf')" />
        </div>
      </div>
    </fieldset>

    <fieldset class="mb-5x">
      <legend>Contato</legend>
      <div class="row">
        <div class="col-12 col-md-8 mb-2x">
          <BrInput id="email" type="email" label="E-mail" v-model="form.email" required
                   :state="estado('email')" @blur="tocar('email')" />
          <BrMessage v-if="erro('email')" state="danger" is-feedback :message="erro('email')" />
        </div>
        <div class="col-12 col-md-4 mb-2x">
          <BrInput id="celular" label="Celular (Opcional)" mask="(##) #####-####"
                   v-model="form.celular" />
        </div>
      </div>
    </fieldset>

    <div class="d-flex justify-content-end">
      <BrButton emphasis="secondary" type="button" class="mr-2x" @click="voltar">Cancelar</BrButton>
      <BrButton emphasis="primary" type="submit" :disabled="!podeEnviar" :is-loading="enviando">
        Enviar Solicitação
      </BrButton>
    </div>
  </form>
</template>
```

**LGPD** — colete o mínimo necessário; explique por que cada dado é pedido; consentimento **nunca** vem pré-marcado; ofereça sempre como revogar; anonimize dados sensíveis e sinalize a anonimização com `BrMessage`.

### 13.2 Listagem com CRUD

Estrutura: filtros em `BrCard` colapsável → `BrTable` → `BrPagination` → `BrModal` de confirmação. Vazio → *empty state* (13.5). Carregando → `BrLoading`.

Reutilize **o mesmo componente `.vue`** para criar e editar, alternando pelo parâmetro de rota (`/registros/novo` vs. `/registros/:id`). No modo edição, oculte campos de senha e trate anexos já existentes como opcionais.

### 13.3 Login

- Card centralizado sobre `bg-gray-2`.
- Logotipo do órgão + título.
- `BrSignIn shape="block"` como ação **prioritária**.
- Divisor com texto ("ou acesse com credenciais locais").
- Formulário local com alternância de visibilidade da senha via slot `action` do `BrInput`.
- Falha de autenticação → `BrMessage state="danger"` no topo do card.

```vue
<div class="d-flex align-items-center justify-content-center bg-gray-2" style="min-height: 100vh">
  <BrCard class="p-4x" style="max-width: 420px; width: 100%">
    <template #content>
      <div class="text-center mb-4x">
        <img :src="logo" alt="Logotipo do órgão" class="mb-2x" style="max-height: 60px" />
        <h2 class="mb-0">Acesso ao Sistema</h2>
      </div>

      <BrMessage v-if="erroLogin" state="danger" show-icon
                 :message="erroLogin" class="mb-3x d-block" />

      <BrSignIn emphasis="primary" shape="block" @click="entrarComGovBr">
        Entrar com gov.br
      </BrSignIn>

      <p class="text-center text-gray-70 my-3x">ou acesse com credenciais locais</p>

      <form @submit.prevent="entrarLocal">
        <BrInput id="usuario" label="E-mail ou CPF" v-model="credenciais.usuario"
                 required class="mb-2x" />
        <BrInput id="senha" :type="verSenha ? 'text' : 'password'" label="Senha"
                 v-model="credenciais.senha" required class="mb-3x">
          <template #action>
            <BrButton shape="circle" emphasis="tertiary" density="small" type="button"
                      :aria-label="verSenha ? 'Ocultar senha' : 'Exibir senha'"
                      @click="verSenha = !verSenha">
              <i :class="verSenha ? 'fas fa-eye-slash' : 'fas fa-eye'" aria-hidden="true"></i>
            </BrButton>
          </template>
        </BrInput>
        <BrButton emphasis="primary" shape="block" type="submit" :is-loading="autenticando">
          Entrar
        </BrButton>
      </form>
    </template>
  </BrCard>
</div>
```

### 13.4 Dashboard / Indicadores

Grid de `BrCard` (camada 1) com uma métrica por card: rótulo em `text-base`/`text-gray-70`, valor em `text-up-05`/`semi-bold`/`text-primary`, tendência em `text-base` com ícone e cor semântica.

```vue
<div class="row">
  <div class="col-12 col-md-6 col-lg-3 mb-3x" v-for="kpi in indicadores" :key="kpi.id">
    <BrCard>
      <template #content>
        <p class="text-base text-gray-70 mb-half">{{ kpi.rotulo }}</p>
        <p class="text-up-05 text-weight-semi-bold text-primary mb-half">{{ kpi.valor }}</p>
        <p class="text-base mb-0" :class="kpi.subiu ? 'text-success' : 'text-danger'">
          <i :class="kpi.subiu ? 'fas fa-arrow-up' : 'fas fa-arrow-down'"
             class="mr-half" aria-hidden="true"></i>
          {{ kpi.variacao }} em relação ao mês anterior
        </p>
      </template>
    </BrCard>
  </div>
</div>
```

### 13.5 Empty state

Anatomia: imagem (opcional) → **título** (obrigatório) → corpo da mensagem (obrigatório) → elementos interativos (opcional).

Três tipos: ausência de dados; ausência por ação do usuário (filtro sem resultado); erro interno. Sempre ofereça o próximo passo — nunca só "Nenhum resultado".

### 13.6 Tela de erro (404, 500…)

Header compacto → área informativa (até 3 níveis de texto + imagem) → área de suporte (busca e/ou botões terciários) → footer compacto.

Regras dos textos:
- Linguagem simples, não técnica.
- **Nunca** sugira que o erro foi culpa do usuário.
- Seja específico sobre o que aconteceu — não genérico.
- Seja construtivo: diga o que fazer em seguida.
- Com 3+ botões, disponha-os verticalmente.

### 13.7 Fluxo em etapas

`BrWizard` para fluxos com painéis de conteúdo por etapa; `BrStep` para o indicador de progresso. Progressão `linear` quando cada etapa depende da anterior; `nonlinear` quando o usuário pode saltar.

---

## 14. Acessibilidade

Conformidade **eMAG** + **WCAG 2.1 nível AA** é requisito legal (Lei 13.146/2015).

### Checklist obrigatório de toda tela gerada

- [ ] `<html lang="pt-BR">`
- [ ] `BrSkipLink` com âncoras para conteúdo, navegação e apoio
- [ ] Hierarquia de headings sem pular níveis (`h1` → `h2` → `h3`)
- [ ] Um único `h1` por página
- [ ] Todo campo com rótulo visível e associado (`id` ↔ `label`)
- [ ] Todo botão de ícone com `aria-label`
- [ ] Todo `<i class="fas …">` com `aria-hidden="true"`
- [ ] Toda imagem com `alt` (vazio se decorativa)
- [ ] Contraste ≥ 4.5:1 (texto normal) e ≥ 3:1 (texto grande e elementos gráficos)
- [ ] Foco visível em todo elemento interativo, sem `outline: none`
- [ ] Ordem de tabulação lógica; navegação 100% por teclado
- [ ] Feedback nunca depende só de cor — sempre ícone + texto
- [ ] Tamanhos em `rem`/`em`; layout íntegro com zoom de 200%
- [ ] Área de interação ≥ 24×24px (mouse) / 48×48px (toque)
- [ ] Estados expostos via ARIA (`aria-expanded`, `aria-checked`, `aria-invalid`, `aria-describedby`)
- [ ] Erros de formulário anunciados e ligados ao campo

**Texto grande** = ≥ 18pt (24px / 1.71em) ou ≥ 14pt bold (19px). Abaixo disso é texto normal.

O estado **desabilitado** é a única exceção às regras de contraste.

---

## 15. Writing e Microcopy

**Voz:** neutra, formal, direta e desburocratizada. **Tom:** ajusta-se ao momento do usuário na jornada.

### Regras gerais

- Seja consistente: um conceito, uma palavra ("Agendar" em toda a aplicação, nunca "Programar").
- Seja conciso sem omitir o essencial.
- Escreva para **uma pessoa, no singular**.
- Prefira **voz ativa**.
- Evite negação de termos negativos, gírias, regionalismos, abreviações e jargão técnico.
- Estrangeirismos: só quando consagrados.
- Siglas: escreva por extenso na primeira ocorrência.
- Use números em algarismos.
- Alinhamento: à esquerda para frases longas; centralizado só para trechos curtos.

### Regras por componente

| Componente | Regra |
| :-- | :-- |
| **Button** | Verbo no infinitivo + resultado. **Iniciais maiúsculas em todas as palavras**, exceto artigos e preposições. Máx. 2 palavras (3 se inevitável). Sem artigos. Uma única linha. ✅ "Enviar Documento" · ❌ "Enviar" · ❌ "Enviar o documento" |
| **Tag** | Substantivo ou adjetivo — **nunca verbo**. Só a primeira letra maiúscula. ✅ "Criação de conteúdo" |
| **Hyperlink** | Palavra-chave descritiva do destino. ❌ "Clique aqui", ❌ "Saiba mais" |
| **Rótulo de campo** | Só a primeira letra maiúscula. Conciso. Consistente quanto ao uso de ":" |
| **Placeholder** | Exemplo genérico do formato esperado. Nunca substitui o rótulo. |
| **Modal** | Botões repetem o verbo da pergunta. ❌ "Sim / Não" |
| **Magic Button** | Call-to-action curto, informativo e impactante. |
| **Mensagem de erro** | Diga o que aconteceu e o que fazer. Nunca culpe o usuário. |

---

## 16. Do's e Don'ts

### ✅ Faça

- Use os componentes `Br*` em vez de recriar HTML.
- Mantenha Header e Footer oficiais em todas as views.
- Use classes utilitárias (`.mb-3x`, `.text-primary`, `.d-flex`) em vez de CSS customizado.
- Escute eventos Stencil corretamente e leia `event.detail`.
- Respeite a ordem hierárquica dos botões (primária à direita / embaixo).
- Alinhe dados de tabela por tipo: texto `start`, data/status `center`, número `end`.
- Reutilize a mesma view para criar e editar registros.
- Use `density` para adaptar telas densas em vez de inventar espaçamentos.
- Prefira remover um elemento a desabilitá-lo.

### ❌ Não faça

- **Não** escreva hexadecimais nem `px` em CSS customizado.
- **Não** use props inexistentes (`circle`, `block`, `feedback` em vez de `is-feedback`).
- **Não** aplique `state="danger"` em `BrCheckbox`/`BrRadio` — o valor é `invalid`.
- **Não** misture outra biblioteca de UI.
- **Não** remova o outline de foco.
- **Não** use imagem para exibir texto.
- **Não** aplique sombra em componentes da camada 0 (inputs, botões, tabelas).
- **Não** faça `h1` em bold — o padrão é `light` (300).
- **Não** pinte o header da tabela de azul sólido — é `--gray-5` com texto azul.
- **Não** aninhe container dentro de container, linha dentro de linha, nem coluna dentro de coluna.
- **Não** substitua rótulo por placeholder.
- **Não** desabilite Menu, Tabs, Modal, Tooltip ou Magic Button.
- **Não** sinalize campos obrigatórios *e* opcionais ao mesmo tempo.
- **Não** invente um modo "alto contraste" — o modelo é fundo claro vs. fundo escuro.

---

## 17. Armadilhas Stencil + Vue

Estes são os erros que mais custam tempo neste stack.

### 17.1 Eventos customizados chegam em `event.detail`

Componentes Stencil emitem `CustomEvent`. O payload **não** é o primeiro argumento — está em `detail`:

```vue
<!-- Registre nas duas grafias: o Vue normaliza de formas diferentes conforme o contexto -->
<BrSelect @value-change="onChange" @valueChange="onChange" />
```

```ts
function onChange(event: CustomEvent) {
  const d = event.detail
  const valor = Array.isArray(d) ? d[0] : (typeof d === 'object' ? d?.value : d)
}
```

### 17.2 `v-model` só funciona nos componentes de formulário

Os wrappers declaram o modelo apenas para: `BrInput` (`value`), `BrTextarea` (`value`), `BrSelect` (`value`), `BrCheckbox` (`checked`), `BrRadio` (`checked`), `BrSwitch` (`checked`).

Para os demais, controle manualmente com prop + listener.

### 17.3 Booleanos vindos do DOM podem ser string

```ts
function onCheckedChange(event: CustomEvent) {
  const raw = event.detail ?? (event.target as HTMLInputElement)?.checked
  return typeof raw === 'string' ? raw === 'true' : !!raw
}
```

### 17.4 Slots nomeados usam `slot="nome"`, não `v-slot`

A projeção acontece no Shadow DOM do Web Component. Para filhos que são eles próprios Web Components (`BrFooterLogo`, `BrFooterSocial`), use o **atributo**:

```vue
<BrFooter theme="dark">
  <BrFooterLogo slot="logo" :src="logo" description="Logotipo" />
</BrFooter>
```

Para conteúdo comum, `<template #nome>` funciona (`BrCard`, `BrModal`, `BrInput`).

### 17.5 Nomes de props: `camelCase` no Vue, `kebab-case` no DOM

`isFeedback` ↔ `is-feedback`, `showIcon` ↔ `show-icon`, `columnWidth` ↔ `column-width`. Ambos funcionam no template; **`feedback` e `icon` não existem**.

### 17.6 `state` tem domínios diferentes por componente

| Componente | Valores aceitos |
| :-- | :-- |
| `BrInput`, `BrUpload`, `BrMessage` | `info` \| `warning` \| `danger` \| `success` |
| `BrTextarea` | `danger` \| `success` \| `warning` (**sem `info`**) |
| `BrCheckbox`, `BrRadio` | `valid` \| `invalid` |
| `BrStepItem` (`highlight`) | `success` \| `info` \| `danger` \| `warning` |

### 17.7 Alinhamento de tabela é `start`/`center`/`end`

Não `left`/`right`. Aplica-se a `BrTable`, `BrTableRow`, `BrTableCell`, `BrTableHeaderCell`.

---

## 18. Receitas de Prompt

Prompts prontos para pedir telas neste padrão.

### Barra de ações de formulário

```text
Crie a barra de ações de um formulário no padrão GovBR-DS usando @govbr-ds/webcomponents-vue.
Da esquerda para a direita: um BrButton terciário "Informações Adicionais"; um BrButton
secundário "Cancelar" que navega para /registros; e um BrButton primário type="submit"
"Salvar Alterações" com ícone fas fa-check (aria-hidden) e mr-base.
Use d-flex justify-content-end e espaçamento com utilitários (mr-2x). O botão primário deve
ficar à direita e desabilitar enquanto o formulário for inválido.
```

### Formulário completo com validação

```text
Desenvolva a view FormularioSolicitante.vue no padrão GovBR-DS.
- Container-lg, h2 de título e parágrafo de subtítulo informando a política de obrigatoriedade.
- Agrupe os campos em <fieldset> + <legend> ("Identificação", "Contato"), com mb-5x entre grupos.
- Campos com BrInput: Nome completo (obrigatório, mín. 5 caracteres), CPF (mask="###.###.###-##"),
  E-mail (type="email"), Celular (mask="(##) #####-####", opcional — marque "(Opcional)" no rótulo).
- BrSelect + BrSelectOption para Unidade Federativa; leia o valor de event.detail.
- Validação lazy: erro no @blur, limpeza durante a digitação.
- Erros via <BrMessage state="danger" is-feedback> imediatamente abaixo de cada campo.
- Mensagem global de sucesso/erro com <BrMessage show-icon is-closable> no topo.
- Grid: col-12 col-md-6 para campos complementares, coluna única para o resto.
- Botões conforme a hierarquia do DS (primário à direita, desabilitado se inválido).
```

### Listagem com ações e paginação

```text
Crie RelatorioSolicitacoes.vue no padrão GovBR-DS.
1. Filtros dentro de um BrCard com BrCollapse (busca por nome + BrSelect de situação).
2. BrTable com density="medium" e has-row-divider. Colunas: Nome do Cidadão (250px),
   CPF (160px), Data da Solicitação (140px, horizontal-alignment="center"),
   Situação (120px, center, exibida com BrTag status) e Ações (column-width="content",
   horizontal-alignment="end").
3. Na coluna Ações, três BrButton shape="circle" emphasis="tertiary" density="small" com
   aria-label obrigatório: Visualizar (fa-eye), Editar (fa-edit) e Excluir (fa-trash-alt).
4. Excluir abre um BrModal size="small" align-footer="end" com botões "Cancelar" e
   "Excluir Registro".
5. BrPagination ao final, 10 itens por página, ligado ao evento pageChange.
6. Empty state quando não houver resultados: título, mensagem e ação sugerida.
Lembre: alinhamento é start/center/end, nunca left/right.
```

### Tela de login gov.br

```text
Construa LoginView.vue no padrão GovBR-DS.
- BrCard centralizado (max-width 420px) sobre fundo bg-gray-2 ocupando a altura da viewport.
- Logotipo do órgão e h2 "Acesso ao Sistema".
- BrSignIn emphasis="primary" shape="block" com o texto "Entrar com gov.br" como ação prioritária.
- Divisor textual "ou acesse com credenciais locais".
- BrInput de E-mail/CPF e de Senha; no slot #action da senha, um BrButton shape="circle"
  emphasis="tertiary" alternando fa-eye / fa-eye-slash, com aria-label dinâmico.
- BrButton primário shape="block" "Entrar", com is-loading durante a autenticação.
- Falha de autenticação exibida em BrMessage state="danger" show-icon no topo do card.
```

### Dashboard de indicadores

```text
Crie PainelIndicadores.vue no padrão GovBR-DS.
- h1 da página e BrBreadcrumb acima dele.
- Row com quatro BrCard (col-12 col-md-6 col-lg-3, mb-3x), cada um com: rótulo em
  text-base text-gray-70, valor em text-up-05 text-weight-semi-bold text-primary, e linha de
  tendência em text-base com ícone fa-arrow-up/fa-arrow-down e classe text-success/text-danger.
- Abaixo, um BrCard ocupando col-12 com a série histórica em BrTable density="small".
- Não aplique sombra manual: BrCard já está na camada 1.
```

---

## 19. Referências

### Portais oficiais

| Recurso | URL |
| :-- | :-- |
| **Design System GovBR-DS** (diretrizes, fundamentos e componentes) | <https://www.gov.br/ds> |
| **Web Components** (documentação da biblioteca) | <https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/webcomponents/> |
| **Wrapper Vue** (integração com Vue 3) | <https://next-webcomponent-ds.estaleiro.serpro.gov.br/docs/frameworks/vue/> |
| **Utilitários CSS — Grid** | <https://www.gov.br/ds/utilities-css/grid> |
| **Wiki de desenvolvimento** | <https://gov.br/ds/wiki/> |

### Repositórios

| Repositório | URL |
| :-- | :-- |
| **Documentação do Design System** (fonte das diretrizes desta referência) | <https://gitlab.com/govbr-ds/govbr-ds> |
| **Web Components GovBR-DS** | <https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc/> |
| **Padrão de Implementação** (Web Components + frameworks) | [PADRAO_IMPLEMENTACAO_GOVBR.md](https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue/-/blob/acdff83a19d141361132c7defd9c6d4abd83c62e/PADRAO_IMPLEMENTACAO_GOVBR.md) |

> O **Padrão de Implementação** é complementar à Seção 17 deste documento: cobre `br-select` declarativo, sincronização de estado via `valueChange`, validação reativa no `blur` e estratégia de testes E2E com Playwright sobre Shadow DOM. Consulte-o ao escrever testes.

### Normas de acessibilidade

| Norma | URL |
| :-- | :-- |
| **eMAG** — Modelo de Acessibilidade em Governo Eletrônico | <https://emag.governoeletronico.gov.br/> |
| **WCAG 2.1** — Web Content Accessibility Guidelines | <https://www.w3.org/TR/WCAG21/> |
| **Lei 13.146/2015** — Estatuto da Pessoa com Deficiência | <https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2015/lei/l13146.htm> |

---

## Rastreabilidade

Cada seção deste documento deriva das fontes abaixo. As diretrizes de design vêm do portal
oficial (espelhado em <https://gitlab.com/govbr-ds/govbr-ds>); a API de componentes vem dos
artefatos publicados no pacote npm.

| Seção | Fonte canônica |
| :-- | :-- |
| 2. Princípios / Temas | <https://www.gov.br/ds/padroes/tema> |
| 3. Cores | <https://www.gov.br/ds/fundamentos-visuais/cores> · <https://www.gov.br/ds/utilities-css/cores> |
| 4. Tipografia | <https://www.gov.br/ds/fundamentos-visuais/tipografia> · <https://www.gov.br/ds/utilities-css/textos> · <https://www.gov.br/ds/utilities-css/tipografia> |
| 5. Espaçamento | <https://www.gov.br/ds/fundamentos-visuais/espacamento> · <https://www.gov.br/ds/utilities-css/espacamento> |
| 6. Grid e Layout | <https://www.gov.br/ds/fundamentos-visuais/grid> · <https://www.gov.br/ds/utilities-css/grid> · <https://www.gov.br/ds/templates/base?tab=designer> |
| 7. Superfície | <https://www.gov.br/ds/fundamentos-visuais/superficie> · <https://www.gov.br/ds/utilities-css/bordas> · <https://www.gov.br/ds/utilities-css/arredondamento> |
| 8. Elevação e Camadas | <https://www.gov.br/ds/fundamentos-visuais/elevacao> · <https://www.gov.br/ds/utilities-css/elevacao> |
| 9. Estados | <https://www.gov.br/ds/fundamentos-visuais/estados> |
| 10. Iconografia | <https://www.gov.br/ds/fundamentos-visuais/iconografia> |
| 11. Densidade | <https://www.gov.br/ds/padroes/design/densidade> |
| 12. Componentes — **API** | `@govbr-ds/webcomponents@2.0.0-next.70` → `dist/types/components.d.ts`, `dist/custom-elements.json`; wrappers em `@govbr-ds/webcomponents-vue` → `src/stencil-generated/components.js` |
| 12. Componentes — **diretrizes visuais** | <https://www.gov.br/ds/components/{componente}?tab=designer> (ex.: [button](https://www.gov.br/ds/components/button?tab=designer), [table](https://www.gov.br/ds/components/table?tab=designer), [message](https://www.gov.br/ds/components/message?tab=designer)) |
| 13.1 Formulário | <https://www.gov.br/ds/padroes/design/formulario> |
| 13.5 Empty state | <https://www.gov.br/ds/padroes/design/emptystates> |
| 13.6 Tela de erro | <https://www.gov.br/ds/templates/erro?tab=designer> |
| 14. Acessibilidade | Portal do DS, seção Acessibilidade; eMAG; WCAG 2.1 AA |
| 15. Writing e Microcopy | <https://www.gov.br/ds/padroes/writing/principios-writing> · <https://www.gov.br/ds/padroes/writing/microcopy> |
| 17. Armadilhas Stencil + Vue | Tipos da biblioteca + [PADRAO_IMPLEMENTACAO_GOVBR.md](https://gitlab.com/govbr-ds/bibliotecas/wbc/govbr-ds-wbc-quickstart-vue/-/blob/acdff83a19d141361132c7defd9c6d4abd83c62e/PADRAO_IMPLEMENTACAO_GOVBR.md) |

**Ao atualizar a versão da biblioteca**, reconfira a Seção 12 contra os artefatos do novo pacote:

```bash
# Props e tipos de cada componente
tar -xzOf govbr-ds-webcomponents-<versão>.tgz package/dist/types/components.d.ts

# Slots e eventos
tar -xzOf govbr-ds-webcomponents-<versão>.tgz package/dist/custom-elements.json

# Lista de props expostas pelos wrappers Vue
tar -xzOf govbr-ds-webcomponents-vue-<versão>.tgz package/src/stencil-generated/components.js
```

---

*DESIGN.md — Única Fonte da Verdade de UI/UX do GovBR-DS neste projeto.*
*Ao alterar, atualize também as seções de Referências e Rastreabilidade.*
