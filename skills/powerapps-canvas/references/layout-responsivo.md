# Layout responsivo

Padrão do kit para tela de computador (TL-xx) e de celular (TM-xx). Substitui o "canvas fixo 1920 por
1080" (decisão T1): com *Dimensionar para ajustar* ligado o app só encolhe e corta texto; desligado,
todo literal de `X`/`Width`/`Height` fica parado e o conteúdo sobra ou corta. Origem: GestaoObras,
ADR-002 e Onda 7A (2026-11-23/24), Painel e Detalhe colados em HML.

## 1. Configuração do app

- Studio › **Configurações › Exibição**: *Dimensionar para ajustar*, *Bloquear taxa de proporção* e
  *Bloquear orientação* **desativados**.
- Confira `App.MinScreenHeight` (propriedade do App, não vem no YAML): o cálculo de altura supõe
  ≤ 720. Maior que isso, a tela ganha rolagem.
- Faixa a provar: de **1280×720 a 2560×1440**. Abaixo de 1280 de largura a tela corta em vez de quebrar.

## 2. Tela

Destino: YAML colado.

```yaml
Properties:
  Height: =Max(App.Height, App.MinScreenHeight)
  Width: =Max(App.Width, fxLarguraMinDesktop)
```

`fxLarguraMinDesktop = 1280`. Tela de celular usa `App.Width` e `App.Height` direto (o
`App.MinScreenHeight` pode passar da altura real e cortar o rodapé).

## 3. Largura

- **Nenhum `Width`/`X` literal.** Use `Parent.Width`, fração de `Parent.Width` ou token `fx*`.
  Literal que precisa ficar (largura de ícone, coluna de botões) leva `# fixo: <motivo>` na linha de cima.
- Colunas de tabela em fração da galeria, somando 1,00; cabeçalho e célula com a **mesma** fração.
- Posição encadeada no vizinho: `X: ='xx-lbl-a'.X + 'xx-lbl-a'.Width + fxLayoutGutter`.
- `fxIsCompact = App.Width < 1600` é o breakpoint de densidade (margem, fonte, altura de filtro).

## 4. Altura

- O conteúdo principal (galeria, painel, coluna, formulário) **ocupa a altura que sobra**:
  `Height: =Parent.Height - Self.Y - fxFooterHeight - fxLayoutGutter`.
- **Nenhum filho com `Y + Height` fixo maior que a altura do pai a 1280×720.** O Studio empurra o
  controle para cima para caber, e ele cobre o vizinho (galeria por cima dos filtros e do cabeçalho
  da tabela).
- Rodapé, contador ("Exibindo x de y") e barra de ações ancorados no fundo:
  `Y: =Parent.Height - Self.Height - fxLayoutGutter`.
- `Y` encadeado no controle de cima: `Y: ='xx-con-kpis'.Y + 'xx-con-kpis'.Height + fxLayoutGutter`.
- Rolagem fica na galeria (cada coluna com a sua), não em contêiner com `LayoutOverflowY`.
- Altura calculada que pode ficar negativa: `Max(0, …)`.

Tokens (em [`assets/app-formulas-tokens.md`](../assets/app-formulas-tokens.md)):

| Token | Valor |
|---|---|
| `fxHeaderHeight` | 64 |
| `fxFooterHeight` | 32 |
| `fxLayoutMargin` | `If(fxIsCompact, 24, 32)` |
| `fxContentTop` | `fxHeaderHeight + fxLayoutGutter` |
| `fxContentHeight` | `Max(App.Height, App.MinScreenHeight) - fxContentTop - fxFooterHeight - fxLayoutGutter` |

Cabeçalho da tela numa linha só de `fxHeaderHeight`: título, ação principal e usuário centralizados
na altura (`Y: =(Parent.Height - Self.Height) / 2`).

## 5. Grade de filtros

- Largura **por linha**: a linha sem botões divide a largura toda; a linha com Filtrar/Limpar/Exportar
  desconta os botões antes de dividir. Dividir as duas linhas pela mesma conta deixa sobra vazia numa
  e campos estreitos na outra.
- Campo de data `dd/mm/yyyy` precisa de ~130 px; combo e texto de ~110 px. Uma linha só quando cada
  campo passa disso: 11 campos pedem `App.Width >= 2200` (a 1600 dariam 60–79 px). Use um token
  (`fxFiltrosUmaLinha`), não o `fxIsCompact`.
- Rótulo curto ("Número", não "Número da demanda"): o rótulo do filtro tem uma linha de 20 px.
- `Classic/DatePicker` **não aceita `Size`** (PA2108); a largura é o único ajuste.

## 6. Texto que pode crescer

- Um rótulo, um dado. Dois dados num rótulo de uma linha (valor + natureza) quebram e cortam; leve o
  segundo para o rótulo de cima ou outro controle.
- Texto longo em célula, cartão ou linha do tempo: **2 linhas com reticências** e o texto inteiro no
  `Tooltip`. Fórmula em bloco `|-` (o `{t: …}` em linha simples quebra o YAML):

Destino: YAML colado.

```yaml
Text: |-
  =With({t: ThisItem.Texto, n: RoundDown((Parent.TemplateWidth - fxCardPaddingS * 2) / 6.5, 0) * 2 - 2}, If(Len(t) > n, Left(t, n) & "…", t))
Tooltip: =ThisItem.Texto
```

  6,5 px por caractere vale para fonte 11; altura do rótulo ≥ 38 para 2 linhas. Galeria flexível com
  `AutoHeight` é a alternativa `[não verificado]`.
- Coluna de data `dd/mm/yyyy` a fonte 11: ≥ 70 px (16% de uma tabela de ~500 px).
- Número em moeda no formato do Brasil: `"R$ " & Text(x, "[$-en-US]#,##0.00", "pt-BR")`. Só com
  `[$-pt-BR]` no formato sai `12,750.00` num navegador em inglês.

## 7. Modal, menu e cabeçalho de tabela

- Modal centralizado por `Parent` e com teto: `Height: =Min(640, Parent.Height - fxLayoutMargin * 2)`.
- Menu recolhido mostra **ícone** (`Classic/Icon`) + `Tooltip` do botão, nunca `Left(nome, 1)`.
- Cabeçalho de tabela arredondado só nas pontas: ver
  [galeria-tabela](../assets/componentes/galeria-tabela.md).

## 8. Celular (TM-xx)

Tela própria, uma coluna, `Parent.Width`, tokens `fxMobile*`, área de toque ≥ 44 px. A tela de
computador não vira tela de celular por responsividade; ação que não existe no celular mostra
"Disponível no computador".

## 9. Antes de colar

1. Tabela por tela com o `Y + Height` do último filho a **1280×720 e 2560×1440**, calculado pelos
   tokens, ≤ altura útil.
2. `validar-telas.py` → `0 erro(s)`.
3. Cole a tela **inteira** (controle que mudou de contêiner duplica num bloco parcial).
4. Não arraste nem redimensione no Studio: isso troca as fórmulas de `X`, `Y`, `Width` e `Height` por
   constantes ([Create responsive layout](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/create-responsive-layout)).
