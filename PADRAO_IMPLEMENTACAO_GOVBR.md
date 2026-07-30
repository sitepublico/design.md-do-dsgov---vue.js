# Guia de Padronização: GovBR-DS Web Components + Frameworks

Este documento descreve as melhores práticas e padrões estabelecidos na versão Vue 3 para garantir a sincronização robusta entre Web Components do GovBR-DS e o estado da aplicação.

## 1. Componentes Declarativos (Ex: `br-select`)

**Problema:** O uso da propriedade `:options` (via string JSON) dificulta a reatividade e a manutenção.
**Padrão:** Sempre utilize subcomponentes declarativos para definir opções.

```html
<!-- PADRÃO RECOMENDADO -->
<br-select label="Cidade">
  <br-select-option
    v-for="item in opcoes"
    :key="item.value"
    :label="item.label"
    :value="item.value"
  ></br-select-option>
</br-select>
```

## 2. Sincronização de Estado e Eventos

**Problema:** A API de Web Components emite eventos CustomEvent que muitas vezes não são capturados pelo `v-model` padrão.
**Padrão:** Escute o evento oficial `valueChange` (emitido pelo Stencil) e capture o `detail`.

- **Vue:** `@value-change` ou `@valueChange`.
- **Angular:** `(valueChange)="seuMetodo($event)"`.
- **Tratamento:** O valor real está em `event.detail`. Em selects com múltipla escolha, pode vir como um Array.

## 3. Validação Reativa (UX)

**Padrão:** Dispare a validação no evento de `blur` (perda de foco) para evitar que o usuário veja erros enquanto ainda está digitando.
1. Use uma flag de `touched` (ex: `fieldsTouched.nome = true`).
2. A mensagem de erro deve ser exibida apenas se o campo foi "tocado" E for inválido.
3. No evento de `change`, atualize o valor e remova o erro instantaneamente.

## 4. Estratégia de Testes E2E (Playwright)

**Problema:** Web Components usam Shadow DOM, o que pode "esconder" inputs e botões de seletores CSS simples.
**Padrão:** Use interação real (cliques e teclado) em vez de injeção de estado (`page.evaluate`).

### Exemplo: Seleção em `br-select` com Busca
A busca interna do `br-select` é um componente `br-input`. O teste deve:
1. Clicar no `br-select` para abrir o menu.
2. Localizar o input real via `select.locator("br-input input")`.
3. Digitar o termo de busca.
4. Clicar fisicamente na opção aparecer no menu `.br-item`.
5. Validar que o menu foi fechado (`toBeHidden`).

### Exemplo: Perda de Foco (Blur)
Para simular a perda de foco de forma confiável em componentes com menus internos:
```typescript
await page.locator("br-select").click();
await page.keyboard.press("Escape"); // Fecha o menu
await page.keyboard.press("Tab");    // Move o foco para fora
```

---
*Este padrão garante acessibilidade, conformidade com o Design System e facilidade de teste em qualquer framework moderno.*

## 5. Especificação de Testes Detalhada

Esta seção deve ser usada como guia para a implementação da suíte de testes em Angular (Karma/Jasmine para unitários e Playwright para E2E).

### A. Testes Unitários (Lógica de Negócio)
Foque em testar as funções puras de validação e formatação, independente da UI.

1.  **Validação de CPF**:
    - Testar CPFs válidos conhecidos.
    - Testar CPFs com todos os dígitos iguais (deve falhar).
    - Testar CPFs com dígitos verificadores inválidos.
    - Validar a função de "limpeza" (remover pontos e traço antes da validação).
2.  **Validação de E-mail**:
    - Validar formatos padrão (`user@domain.com`).
    - Validar falhas em e-mails sem `@`, sem domínio ou com caracteres especiais inválidos.
3.  **Lógica de Senha**:
    - Validar requisito de comprimento mínimo (8 caracteres).
    - Validar lógica de igualdade entre "Senha" e "Confirmação de Senha".
4.  **Máscaras de Entrada**:
    - Garantir que as funções de máscara para Celular, CEP e CPF formatam as strings corretamente durante a digitação.

### B. Testes E2E (Fluxos de Usuário - Playwright)
Estes testes validam se a UI reage corretamente às interações.

#### 1. Ciclo de Vida da Mensagem de Erro
- **Cenário**: Exibição tardia (Lazy Validation).
  - **Ações**: Clicar no campo Nome, não digitar nada e sair do campo (`blur`).
  - **Expectativa**: A mensagem "Nome é obrigatório" deve aparecer.
- **Cenário**: Limpeza instantânea.
  - **Ações**: Com o erro visível, digitar caracteres válidos.
  - **Expectativa**: A mensagem de erro deve desaparecer imediatamente, antes mesmo do próximo `blur`.

#### 2. Componentes Complexos (Shadow DOM)
- **Select (Cidade)**:
  - Validar que clicar no select abre o menu.
  - Validar que a busca filtra os resultados.
  - Validar que selecionar "Brasília - DF" fecha o menu e atualiza o estado interno.
  - **Crítico**: Garantir que o valor selecionado foi "comitado" no formulário (verificar se o erro de "campo obrigatório" sumiu).
- **Upload (Arquivos)**:
  - Validar que o componente aceita um arquivo real via `setInputFiles`.
  - Validar que o nome do arquivo aparece na lista de "Arquivos Enviados".
- **Radio e Checkbox**:
  - Validar que clicar no `label` ou no componente atualiza o estado binário/valor escolhido.

#### 3. Submissão e Tabela
- **Cenário**: Bloqueio de submissão inválida.
  - **Ações**: Clicar em "Cadastrar" com campos vazios.
  - **Expectativa**: Um banner global de erro aparece e o formulário foca no primeiro erro.
- **Cenário**: Fluxo Feliz (Happy Path).
  - **Ações**: Preencher 100% dos dados válidos e clicar em "Cadastrar".
  - **Expectativa**:
    1. O formulário é resetado (campos vazios).
    2. Uma nova linha aparece na `br-table` com os dados digitados (Nome e Cidade).
    3. Mensagens de sucesso são exibidas.
- **Cenário**: Persistência de Dados.
  - **Ações**: Cadastrar dois usuários em sequência.
  - **Expectativa**: A tabela deve obrigatoriamente conter 2 linhas, preservando o histórico da sessão.

#### 4. Reset de Estado (Botão Limpar)
- **Ações**: Preencher dados parciais, gerar mensagens de erro e clicar em "Limpar".
- **Expectativa**: Todos os `br-input` ficam vazios, o `br-select` volta para o estado inicial e **todas** as mensagens de erro (campos e topo) desaparecem.

## 6. Gestão de Estado Centralizada (Records Store)

Para aplicações CRUD, o estado deve ser movido dos componentes locais para um **Store Reativo** centralizado.

- **Padrão**: Utilize `reactive` do Vue 3 para criar um "Single Source of Truth".
- **Vantagem**: Persistência de dados entre navegações (ex: ir do formulário para a listagem sem perder o que foi cadastrado).
- **Implementação**: Crie um arquivo `src/data/recordsStore.ts` que exporta o estado e os métodos de mutação (`add`, `update`, `delete`).

## 7. Arquitetura de Formulário Bi-modal (Criação/Edição)

Em vez de criar componentes separados para criar e editar, utilize o mesmo componente `Formulario.vue` com lógica de detecção de contexto.

1.  **Detecção**: Use parâmetros de rota (`/formulario/:id`).
2.  **Pre-filling**: No hook `onMounted`, busque os dados no Store se houver um ID e popule o `formData`.
3.  **Segurança**: No modo edição, omita ou desabilite campos sensíveis (como senhas).
4.  **Preservação de Anexos**: Se o campo de upload for obrigatório no cadastro, na edição ele deve se tornar opcional caso já exista um arquivo registrado (exiba uma mensagem: "Documento atual preservado").

## 8. Tabelas Responsivas com `br-table`

Para garantir que a listagem de registros seja "premium" e responsiva:

1.  **Largura de Coluna**: Utilize a propriedade `column-width` em `br-table-header-cell` (ex: `column-width="200px"`) para evitar que o conteúdo seja esmagado.
2.  **Scroll Horizontal**: Envolva o `br-table` em uma `div` com `overflow-x: auto`. Isso permite que o usuário deslize a tabela lateralmente em telas pequenas enquanto as colunas mantêm sua legibilidade.
3.  **Ações Rápidas**: Utilize botões circulares (`circle`) na última coluna para facilitar o acesso visual às funções de Editar e Excluir.

## 9. Testes de Ciclo de Vida do CRUD (E2E)

A suíte de testes E2E deve cobrir a jornada completa do dado:
1.  **Create**: Preencher formulário + Submit + Verificar redirecionamento.
2.  **Read**: Localizar o novo registro na `br-table` da página de listagem.
3.  **Update**: Clicar em editar + Alterar um campo + Salvar + Verificar atualização na tabela.
4.  **Delete**: Clicar em excluir + Confirmar no `br-modal` + Verificar sumiço do registro.

---
*Atualizado em: 10/04/2026 - Inclusão de Padrão CRUD e Store Reativo.*
