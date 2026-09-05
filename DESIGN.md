# Design system da casca

A casca do gerador é um design system pequeno e **fechado**: 24 tokens e 13 primitivas, tudo dentro de um `<style>` só, sem framework e sem build. Ele é pequeno de propósito — a única cor da tela é a arte gerada, e uma UI com sistema de cor próprio competiria com ela.

Este documento é o inventário do que existe hoje, com o porquê de cada decisão e **o que ainda não é token**. Os números aqui são lidos do `index.html`, não idealizados: onde a implementação foge do sistema, está escrito que foge.

> A casca não sabe nada de dentro dos motores. O que ela desenha vem de um adaptador de 23 métodos — ver o bloco `CONTRATO` no topo do `index.html` e a seção *Como um motor entra na aplicação* do [README](README.md). Aqui só se fala da superfície.

---

## 1. Tokens

Vinte e quatro variáveis em `:root`. A regra é curta: **se o valor já existe como token, ele não pode aparecer escrito à mão.** A seção 2 lista onde essa regra ainda não é cumprida.

### Superfície — uma rampa de elevação, não uma paleta

Preto sobre preto não separa painel de card de campo: a UI inteira vira uma mancha e a borda passa a ser a única pista de que há uma caixa ali. A rampa existe para que cada nível seja legível **sem** borda.

| token | valor | nível | onde |
|---|---|---|---|
| `--bg` | `#161616` | 0 — o palco | fundo da página, atrás do canvas |
| `--s1` | `#262626` | 1 — o que flutua | painel, barra, popovers |
| `--s2` | `#333` | 2 — o que está dentro | cards (`.grupo`, `.ctl`) |
| `--s3` | `#404040` | 3 — o que se opera | campos, chips, botões neutros, `kbd` |
| `--s4` | `#4d4d4d` | 4 — o dedo em cima | hover de tudo que é `--s2`/`--s3` |

Três consequências que valem como regra:

- **Cada nível é o fundo do seguinte.** Um card `--s2` só existe dentro de uma caixa `--s1`; um campo `--s3` só dentro de um card. Nível pulado é bug de composição, não escolha de cor.
- **Hover sobe exatamente um degrau** (`--s3` → `--s4`). Não há um token de hover por componente.
- **Cinza médio, não quase-preto.** O escuro anterior lia como um buraco na tela, e separar duas superfícies exigia forçar a vista.

As três superfícies flutuantes são **translúcidas** — `color-mix(in srgb, var(--s1) 92%, transparent)` com `backdrop-filter: blur(22px) saturate(1.35)`, dentro de um `@supports`. A arte atravessa a casca, e a UI pesa menos sobre a composição. 92% e não menos porque abaixo disso, com paleta clara atrás, o texto secundário cai de 4.5:1 (seção 7).

> **A posição desse bloco no arquivo é carga, não arrumação.** Ele precisa vir depois das três regras que sobrescreve: enquanto morava logo abaixo do painel, `.barra` e `.pop` — definidas mais abaixo, com a mesma especificidade — devolviam o fundo para o `--s1` opaco, e as duas carregavam um `backdrop-filter` que, atrás de fundo opaco, não borra nada.

### Traço e texto

| token | valor | uso |
|---|---|---|
| `--line` | `#4a4a4a` | borda de tudo que é caixa ou card; divisores |
| `--line-2` | `#5e5e5e` | a mesma borda em hover ou em estado ligado |
| `--txt` | `#f5f5f5` | texto primário |
| `--dim` | `#b5b5b5` | secundário, rótulos caixa-alta, ícones em repouso |

`--dim` é claro o bastante para continuar legível **com paleta clara atrás da translucidez** — é isso que define o valor, não o gosto.

### Acento

| token | valor | uso |
|---|---|---|
| `--accent` | `#f0f0f0` | tinta da aba ativa, trilha preenchida do slider, anel de foco, chip vivo |
| `--accent-2` | `#fff` | texto de um controle ligado (`.seg`, `.exportar`) |
| `--accent-soft` | `rgba(255,255,255,.14)` | fundo de estado ligado e halo de foco |

**Cinza puro, sem viés de matiz.** Em cinza médio o contraste vem do valor, não do tom; um acento colorido competiria com a composição atrás e mudaria de significado a cada paleta da arte.

### Sombra e realce

| token | valor | uso |
|---|---|---|
| `--sombra` | `0 18px 44px -14px rgba(0,0,0,.6), 0 2px 10px rgba(0,0,0,.35)` | só no que flutua: painel, barra, popovers, aviso |
| `--topo` | `inset 0 1px 0 rgba(255,255,255,.07)` | em toda superfície elevada, inclusive cards |

Os dois papéis são distintos e não se substituem: `--sombra` diz *"isto está sobre a arte"*; `--topo` é o fio de luz na aresta superior que diz *"isto é uma superfície, não um retângulo pintado"*.

**As três curtas** são de outra ordem de grandeza: peça pequena dentro de um card, e não caixa flutuando sobre a arte. Por isso moram longe de `--sombra` na rampa de profundidade — não são versões dela.

| token | valor | uso |
|---|---|---|
| `--sombra-curta` | `0 8px 18px -8px rgba(0,0,0,.8)` | a miniatura levantando no hover |
| `--sombra-funda` | `0 6px 20px -8px rgba(0,0,0,.9)` | a miniatura escolhida, que assenta mais fundo |
| `--sombra-peca` | `0 1px 5px rgba(0,0,0,.55)` | o thumb do slider, solto da trilha |
| `--contorno` | `0 0 0 1px rgba(0,0,0,.4)` | **não é sombra, é contorno**: o anel escuro de 1px que faz um ponto branco sobreviver a qualquer cor da rampa por baixo dele |

O último está aqui por morar na mesma propriedade, `box-shadow`, e não por fazer a mesma coisa. Chamá-lo de sombra seria a documentação repetindo um acidente do CSS.

### Movimento

Duas curvas e três durações. As curvas são absolutas: **nenhuma `cubic-bezier` aparece escrita à mão** fora do `:root`. As durações têm três exceções, listadas na seção 2.

| token | valor | quando |
|---|---|---|
| `--e-out` | `cubic-bezier(.22,.61,.36,1)` | o padrão: entra rápido e assenta |
| `--e-mola` | `cubic-bezier(.34,1.38,.64,1)` | passa do ponto e volta — só onde algo **aparece** ou **é escolhido** |
| `--t-1` | `110ms` | resposta ao dedo: hover, `:active`, troca de cor |
| `--t-2` | `190ms` | aparecer e sumir: opacidade, popovers, chips |
| `--t-3` | `300ms` | deslocamento: caixas, tinta das abas, cascata dos cards |

`--t-3` é lido **pelo JS** (`SAIDA`, em `abrir()`) para saber quanto esperar antes de abrir a próxima caixa. O token é a fonte; o JS não tem cópia do número. Mexer em `--t-3` acerta os dois lados.

### Medida viva: `--barra`

`--barra` (`80px` de partida) não é uma escolha de design: é **medido em JS** a cada `render()` por `medirBarra()`, e vale a altura da barra mais o quanto ela está afastada da base. Três coisas dependem dele:

- onde as caixas param (`bottom: calc(var(--barra) + 12px)`);
- o teto de altura das caixas (`max-height: calc(100dvh - var(--barra) - 24px)`);
- portanto, se o botão que fecha a caixa continua alcançável.

A barra muda de altura no estreito (o rótulo *Exportar* some, o padding encolhe). Um valor fixo aqui esconderia o botão de fechar em algum aparelho.

---

## 2. O que ainda não é token

Esta seção é o passivo do sistema. Nada aqui está quebrado — está **fora do inventário**, que é como um DS pequeno começa a vazar. São oito entradas — seis cores e duas de tempo. Nenhuma duplica um token, e nenhuma é de sombra: as quatro sombras curtas que estavam aqui já foram nomeadas.

O que **está** padronizado, e vale registrar com a mesma precisão: os vinte e quatro tokens são todos usados (não há token morto); nenhuma curva de easing aparece escrita à mão; e o JS da casca não escreve cor nenhuma — as cores que existem no script são as paletas da arte, que são dado dos motores, não superfície.

### Valores fora da rampa

| valor | onde | por quê está fora |
|---|---|---|
| `#1c1c1c` | fundo da `.capa`, tinta do `.chip.vivo` e do visto | é uma sexta superfície e, ao mesmo tempo, a "tinta escura sobre acento claro". Merece dois tokens, não um literal em três lugares |
| `#8f8f8f` | `::placeholder` do campo de busca | terceiro nível de texto que não existe como token (e o único par abaixo de 4.5:1 — seção 7) |
| `#2e2e2e` | `select option` | a lista é desenhada pelo navegador; o valor mora perto de `--s1` mas não é ele |
| `#303030` | borda das `.bolinhas` | recorte entre as bolinhas sobrepostas, calibrado contra `--s3` |
| `#555` / `#6b6b6b` | scrollbar | fora da rampa por serem cromo do navegador |
| `#fff` | thumb do slider, borda da amostra ativa, ponto da rampa | branco puro proposital, não `--accent-2` por acaso — mas indistinguível dele hoje |
| `420ms` e `60ms` | animação de entrada da barra | as únicas durações da casca que não saem de `--t-1/2/3` |
| `45ms` | passo da cascata dos cards (`--i * 45ms`) | é um **intervalo**, não uma duração — mas continua sendo um número de tempo sem token |

### Escalas que existem sem variável

Raio, tipo e espaço **têm escala** — só não têm token. Elas são consistentes por convenção, e é isso que este documento fixa.

**Raio** — uma escala inteira, e cada degrau significa um tamanho de caixa:

| raio | onde |
|---|---|
| `20px` | painel |
| `17px` | barra, popovers |
| `15px` | cards (`.grupo`, `.ctl`) |
| `12px` | botão *Exportar*, rampa de tons |
| `11px` | ícones da barra |
| `10px` | capa, marca do grupo, botão de segmento, select, item de menu |
| `8px` | anel de foco |
| `6px` / `5px` | `code`, `kbd` |
| `999px` | campo de busca, chip, botão da paleta, aviso, trilha do slider |
| `50%` | bolinhas, amostras, thumb, botão de limpar |

Regra implícita: **quanto maior a caixa, maior o raio**; e o que é redondo (`999px`/`50%`) é sempre algo que se lê como pastilha ou como cor, nunca como painel.

**Tipo** — uma família só, `ui-sans-serif, system-ui, "Segoe UI", Roboto, Helvetica, Arial, sans-serif`, base `13px/1.45`. Monoespaçada só em `.csub code`.

| tamanho | peso | papel |
|---|---|---|
| `13.5px` | 600 | aba, título de grupo |
| `13px` | 400 | base do corpo |
| `12.5px` | 600 / 500 / 400 | rótulo de controle, *Exportar*, aviso, item de menu, campo |
| `12px` | 400 | select, estado vazio |
| `11.5px` | 400 | subtítulo de grupo, botão de segmento |
| `11px` | 700 / 400 | cabeçalho caixa-alta (`letter-spacing: .09em`), chip, dica, atalhos |
| `10.5px` / `10px` | 400 | `code`, `kbd` |

Só três pesos: 400, 600 e 700 — e o 700 aparece **exclusivamente** em cabeçalho caixa-alta.

**Espaço** — múltiplos de aproximadamente 2, entre 4 e 26px. Os valores estruturais: `24px` (gap das abas), `20px` (respiro lateral do painel), `16px` (padding dos popovers), `14px` (padding dos cards e gap do corpo), `12px` (gap da barra, e a folga de toda caixa até a barra), `9–10px` (padding interno de campos), `6px` (gap de segmentos).

Tokenizar essas três escalas seria a próxima evolução honesta do sistema — e o custo de não fazê-lo já apareceu: a seta do `select` era desenhada pelo navegador a 4px da borda, enquanto o texto do mesmo campo respirava 10px.

---

## 3. Primitivas

Treze. Cada uma é uma composição da rampa, não uma peça com cor própria.

| primitiva | superfície | raio | notas |
|---|---|---|---|
| **caixa** (`.painel`, `.pop`) | `--s1` translúcido + `--line` + `--sombra`/`--topo` | 20 / 17 | tudo que a barra abre; ver *Leis* |
| **barra** | `--s1` translúcido | 17 | fixa, `left:50%` + `translateX(-50%)`, `bottom:26px` |
| **card** (`.grupo`, `.ctl`) | `--s2` + `--line` + `--topo` | 15 | sem `--sombra`: card não flutua, está dentro |
| **campo** (`input`, `select`) | `--s3`, borda transparente | 999 / 10 | foco: `--accent` na borda + halo `--accent-soft` de 3px |
| **chip** (`.chip`) | `--s3`, `tabular-nums` | 999 | `.vivo` durante o arraste: fundo `--accent`, escala 1.08 |
| **segmento** (`.seg button`) | `--s3` | 10 | ligado: `--accent-soft` + `--line-2` + `--accent-2` |
| **capa** (`.capa`) | `#1c1c1c` + `--line` | 10 | `aspect-ratio:1`; hover levanta 2px e amplia o canvas 1.07 |
| **slider** (`input[type=range]`) | trilha `--s3`, preenchida com `--accent` via `--fill` | 999 | thumb branco de 14px; halo de 6px no `:active` |
| **ícone de barra** (`.ico`) | transparente → `--s3` no hover | 11 | 36×36, ícone de 17px |
| **item de menu** | transparente → `--s3` no hover | 10 | o hover também empurra o `padding-left` de 10 para 14 |
| **aviso** (`.aviso`) | `--s3` + `--line` + `--sombra` | 999 | `role="status"`, `aria-live="polite"` |
| **tecla** (`kbd`) | `--s3`/`--s4` + `--line`, borda de baixo dobrada | 5 | a borda de 2px embaixo é o relevo da tecla |
| **tinta** (`.tinta`) | `--accent` | 2 | 2px sob a aba ativa; desliza em `--t-3` |

**Ícones** são todos do mesmo desenho: `viewBox="0 0 24 24"`, `fill="none"`, `stroke="currentColor"`, traço 1.8–2.4, pontas e junções redondas. Tamanhos: 17 (barra), 16 (marca), 15 (aba, *Exportar*), 14 (lupa), 12 (setas), 10 (limpar). Quando o navegador insiste em desenhar o seu — a seta do `select` —, ela é substituída por uma nossa, no mesmo desenho e no mesmo respiro do campo.

---

## 4. Estados

### Onde o estado mora

Duas regras, e a distinção entre elas é o que evita `.ativo` espalhado pelo CSS:

- **Estado de controle mora no ARIA.** `[aria-pressed]`, `[aria-selected]` e `[aria-expanded]` são os seletores de estilo. Não existe classe de estado ligado em botão nenhum. O efeito colateral é o que interessa: um controle que se pinta de ligado **é** um controle que se anuncia ligado ao leitor de tela — as duas coisas não podem divergir porque são a mesma coisa.
- **Estado de visibilidade mora na classe.** `.on` (popover, aviso, atalho, botão de limpar), `.oculto` (painel), `.vivo` (chip em arraste), `.anima` (cascata dos cards). São estados da casca, não do controle, e não têm o que anunciar.

### Os cinco estados

| estado | como se manifesta |
|---|---|
| **repouso** | superfície da rampa, borda transparente ou `--line`, texto `--dim` quando secundário |
| **hover** | sobe um degrau (`--s3`→`--s4`), ou a borda vai para `--line-2`; texto sobe de `--dim` para `--txt`. Sempre em `--t-1` |
| **pressionado** | `transform: scale(.96)` — `.9` nos ícones da barra, que são menores. Nunca troca de cor |
| **ligado** | `--accent-soft` de fundo, `--line-2` de borda, `--accent-2` de texto. Em superfícies de arte (capa, amostra) vira borda `--accent` mais um visto que entra com mola |
| **foco** | **um anel só para tudo**: `:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; border-radius: 8px }`, e `:focus:not(:focus-visible)` sem nada. Campos e slider trocam o anel pelo halo `--accent-soft`, que é o mesmo acento em outra forma |

Não há estado **desabilitado** em lugar nenhum: um controle que não se aplica ao template ativo **não é desenhado**. A casca decide por capacidade (`if (eng.exportSVG)`), nunca por identidade do motor — a mesma lei que rege o adaptador rege a UI.

---

## 5. Como o movimento é aplicado

| onde | duração | curva | por quê |
|---|---|---|---|
| hover, `:active`, troca de cor | `--t-1` | `--e-out` | resposta ao dedo, tem de parecer instantânea |
| opacidade de popover e aviso | `--t-2` | `--e-out` | aparecer e sumir |
| entrada de popover, chip vivo, amostra, visto | `--t-2`/`--t-3` | `--e-mola` | algo **apareceu** ou **foi escolhido** — a mola é o acento do gesto |
| caixas abrindo e fechando, tinta das abas, cascata dos cards | `--t-3` | `--e-out` | deslocamento; a cascata soma `45ms × índice` |

Três decisões de movimento que são de sistema, não de componente:

- **A cascata só roda quando o conteúdo troca de verdade** — aba, template, motor. Um clique num toggle refaz o mesmo conteúdo: ali a cascata seria uma piscada de tela inteira. Quem decide é uma assinatura de conteúdo comparada a cada `render()`.
- **O hambúrguer e o X são o mesmo desenho em dois estados.** As três linhas se juntam e duas giram (`transform-origin: 12px 12px` com `transform-box: view-box`). Trocar o `innerHTML` mataria a transição, e o botão voltaria a mudar de significado sem mostrar a mudança.
- **`prefers-reduced-motion: reduce` zera tudo** — `animation-duration`, `animation-delay` e `transition-duration` em `*`, `::before` e `::after`. A mesma UI, sem transição nenhuma. Movimento aqui é acabamento, nunca requisito.

---

## 6. Leis da casca

Cinco invariantes. São elas que fazem 24 tokens bastarem.

1. **Uma caixa de cada vez.** Painel, paleta e menu são um estado só (`APP.caixa`), não três liga-desligas. Abrir uma fecha a outra, e a que entra **espera a que sai terminar de sair** — cruzar as duas animações ainda é ver duas caixas ao mesmo tempo, só que em movimento.
2. **Tudo que a barra abre para 12px acima da barra**, centrado na horizontal. A caixa aparece perto do botão que a abriu e do que a fecha.
3. **Detecção por capacidade, nunca por identidade.** Se o método existe no motor, o controle aparece. Nenhum `if (motor === '3d')` na casca.
4. **Um anel de foco para tudo.** Antes não havia nenhum, e dava para percorrer a UI inteira no teclado sem ver onde se estava.
5. **Refazer o painel não custa a rolagem nem o foco.** Cada controle carrega uma âncora (`data-fk`), e o `render()` devolve os dois quando o conteúdo é o mesmo.

---

## 7. Contraste medido

Valores reais dos pares que a UI usa (WCAG 2.1, texto normal exige 4.5:1; elemento gráfico, 3:1).

| par | razão | |
|---|---|---|
| `--txt` sobre `--bg` | 16.6:1 | ✅ |
| `--txt` sobre `--s1` | 13.9:1 | ✅ |
| `--txt` sobre `--s2` | 11.6:1 | ✅ |
| `--accent` sobre `--s1` | 13.3:1 | ✅ |
| `#1c1c1c` sobre `--accent` (chip vivo) | 15.0:1 | ✅ |
| `--dim` sobre `--s1` | 7.4:1 | ✅ |
| `--dim` sobre `--s2` | 6.2:1 | ✅ |
| `--dim` sobre `--s1` **a 92% com arte branca atrás** | 5.8:1 | ✅ — é este número que fixa a opacidade em 92% |
| `--dim` sobre `--s3` | 5.1:1 | ✅ |
| `--dim` sobre `--s4` | 4.1:1 | ⚠️ abaixo de 4.5 |
| `#8f8f8f` (placeholder) sobre `--s3` | 3.2:1 | ⚠️ abaixo de 4.5 |

Os dois avisos são reais e estreitos:

- `--dim` sobre `--s4` só aparece como **texto** no `kbd` do atalho `/` dentro do campo de busca — e esse `kbd` some assim que o campo recebe foco. Nos demais casos `--s4` é hover, e o hover sobe o texto para `--txt` junto.
- O placeholder é o par mais fraco da UI. Subi-lo para `--dim` resolveria (5.1:1) ao custo de ele deixar de se distinguir do texto digitado.

---

## 8. Lacunas conhecidas

Em ordem de quanto custam:

1. **Raio, tipo e espaço não são tokens.** São escalas por convenção, documentadas na seção 2. É a lacuna que mais provavelmente vira divergência.
2. **Oito valores fora do inventário** (seção 2): seis cores e as duas durações da entrada da barra e da cascata. Nenhum duplica token, e cada um tem um motivo próprio para estar solto — o que falta é decidir se viram token ou se ficam como exceção assumida.
3. **Não há tema claro.** A rampa suporta — é só inverter os cinco degraus —, mas os literais da seção 2 e os `rgba(0,0,0,…)` das sombras estão presos ao escuro.
4. **O placeholder está abaixo de 4.5:1** (seção 7).
5. **`select option` não é estilizável** de forma confiável fora do Chromium; a lista aberta é do sistema.
