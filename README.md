# Stack Generators

Dois geradores de assets: formas geométricas que se reformatam, reorganizam e reconstroem, inspirados no sistema generativo do rebrand da Stack Overflow.

**▶ [lucasbatistaribeiro.github.io/animations](https://lucasbatistaribeiro.github.io/animations/)**

| | |
|---|---|
| [Bands 2D](https://lucasbatistaribeiro.github.io/animations/stack-generator/) | Formas chapadas: variantes de uma elipse, código de barras ou compassos |
| [Stacks 3D](https://lucasbatistaribeiro.github.io/animations/stack-generator/stacks-3d.html) | Blocos extrudados em projeção axonométrica, cinco layouts |

Cada gerador é **um arquivo HTML autocontido**: sem dependências, sem build, sem servidor. Dá para abrir direto do disco — só o "Copiar link" e os downloads exigem `http(s)`, por causa das restrições de contexto seguro do navegador.

---

## Estrutura

```
.
├── index.html                    # capa servida pelo GitHub Pages
├── stack-generator/
│   ├── index.html                # Bands 2D
│   └── stacks-3d.html            # Stacks 3D
├── html.html                     # demo antigo: barra de progresso em CSS
├── style.css                     # css do demo antigo
├── LICENSE                       # MIT
└── README.md
```

O Pages serve a branch `main` a partir da raiz.

---

## Bands 2D

Três layouts de formas chapadas, cada um construído a partir de uma única figura que se repete em variantes.

### Layout: Arc Bands

Existe uma **elipse principal**, e cada peça é ela mesma cortada por uma corda cada vez mais baixa: a primeira é a elipse inteira, as seguintes são calotas cada vez mais rasas. É a figura sendo quebrada em variantes de si.

As peças são empilhadas **pela própria altura**, então a pilha nunca se sobrepõe, qualquer que seja a progressão ou o número de peças — a `Abertura` só acrescenta respiro entre elas.

| | |
|---|---|
| **Peças** | quantas variantes, contando a elipse inteira |
| **Raio** | tamanho da elipse principal |
| **Achatamento** | razão entre os eixos: baixo deixa a elipse fina e larga, alto a aproxima do círculo |
| **Progressão** | como a corda sobe de uma peça para a próxima. Em 100% ela sobe em passo constante; acima disso as primeiras peças ficam parecidas e as últimas afinam de vez |
| **Abertura** | respiro entre as peças empilhadas |
| **Variação** | irregularidade da altura das cordas, a partir da semente |

### Layout: Barcode

A fronteira é um **corte reto**. Cada célula tem um peso e uma proporção de tinta, e essa proporção cai geometricamente ao longo da fila: densa de um lado, rarefeita do outro. Os pesos são normalizados no fim, então **a fila preenche o quadro exato** em qualquer contagem, com qualquer jitter.

| | |
|---|---|
| **Barras** | quantas células, e também o período do padrão |
| **Tinta** | proporção de tinta na primeira célula |
| **Progressão** | fator geométrico da queda. 100% mantém o ritmo constante; abaixo disso a tinta afina e o papel engorda ao longo da fila |
| **Direção** | esquerda, direita, cima ou baixo — orienta as barras e define para onde a fila caminha |
| **Variação** | irregularidade dos pesos das células, a partir da semente |

A animação padrão é **Caindo**: a barra de apoio não sai do lugar e as outras se soltam dela, afinando conforme se afastam.

A diferença em relação a *Andar* é de onde vem a largura. Na marcha, a largura viaja junto com a barra — o padrão inteiro translada. Em *Caindo*, a largura é função da **distância até o apoio**: a barra afina enquanto desce, em vez de carregar a própria espessura. Nos dois casos o loop fecha porque, ao fim de um período, cada barra assume exatamente o lugar (e a espessura) da anterior.

Na marcha, com `Progressão` em 100% não existe degrau entre um período e o seguinte e a fila anda sem costura alguma; abaixo disso o degrau da rampa atravessa a tela, como a zona de silêncio de um código de barras.

`Direção` orienta a fila e diz de qual borda o apoio segura: em `baixo`, ele fica no topo e as barras caem.

### Layout: Cadence

Cada **linha é um compasso próprio**: sua própria contagem de blocos e sua própria velocidade. A leitura vem do contraste entre elas — muitas listras finas em cima, poucas e largas embaixo, cada uma correndo no seu tempo.

O loop fecha por construção: o padrão de uma linha se repete a cada célula, e cada linha avança um número **inteiro** de células por ciclo. Então todas voltam ao lugar ao mesmo tempo, mesmo andando em velocidades diferentes.

| | |
|---|---|
| **Linhas** | quantos compassos empilhados |
| **Blocos** | quantos blocos na primeira linha |
| **Progressão** | como a contagem muda de uma linha para a próxima. Abaixo de 100% as linhas vão ficando mais largas para o fim |
| **Espessura** | fração de tinta dentro de cada bloco |
| **Ritmo** | o quanto as velocidades se espalham entre as linhas. Em 0 todas correm juntas |
| **Direção** | para onde as linhas correm; em `cima`/`baixo` os compassos viram colunas |
| **Variação** | irregularidade de contagem e espessura entre as linhas |

### Comum aos três

| | |
|---|---|
| **Ângulo** | só no Arc Bands: gira a pilha inteira, de -180° a 180°. No Barcode e no Cadence a direção é discreta, pelos quatro botões |
| **Formato** | 16:9, 1:1, 9:16, 4:5, 3:1 |
| **Animação** | os modos disponíveis mudam com o layout. Arc Bands: `deslizar` (as cordas sobem e descem, e cada peça atravessa suas variantes), `sanfona`, `pulsar`, `girar`. Barcode: `caindo`, `andar`, `pulsar`, `sanfona`. Cadence: `correr`, `pulsar` |
| **Cores** | 8 paletas, em duas cores (acento e fundo) ou usando a paleta inteira, uma cor por faixa |
| **Semente** | irregulariza a fila sem sair do sistema |

`espaço` gera · `F` congela · `E` exporta PNG · `1` `2` `3` trocam de layout · clique na arte move o primeiro corte.

### Export

PNG 1× e 2× (até 4800 px no banner), **SVG vetorial** e link compartilhável com o estado no hash da URL. O SVG usa a mesma primitiva e a mesma ordem de pintura do canvas, então o vetor não sai diferente da tela.

---

## Stacks 3D

Renderer axonométrico próprio em canvas 2D, com *painter's algorithm*. A unidade é uma **peça**: um footprint 2D em sentido anti-horário mais `z0`/`z1`. Cada layout só descreve polígonos e alturas — prisma, cor de face e ordenação por profundidade são compartilhados.

### Os cinco layouts

| layout | geometria | `Effector` | `Lóbulos` | folga |
|---|---|---|---|---|
| **Pyramid** | zigurate de placas maciças, cada uma apoiada na anterior | torção entre os níveis | — | **Recuo**: o quanto cada nível recua |
| **Twist** | torre de placas quadradas com rotação progressiva | torção total | ondulação da largura na altura | **Espaço** entre placas |
| **Stack** | campo em grade N×N de colunas | amplitude da onda | frequência da onda radial | folga entre colunas |
| **Tower** | fatias iguais empilhadas com folga | desalinhamento lateral | ondas do desalinhamento na altura | **Folga**: é ela que abre a listra do topo da fatia de baixo |
| **Circle** | coroa radial de braços em volta de um miolo | contraste entre o braço mais alto e o mais baixo | quantas rampas cabem na volta | folga angular entre braços |

Em `Altura 100`, a pirâmide fica com o degrau exatamente do tamanho do recuo — a escada a 45°.

### Wave e Rotate

- **Wave** é o único efeito que altera geometria. Desligado, tudo repousa no tamanho neutro: pirâmide reta, degraus iguais, colunas iguais, torre reta, braços todos do mesmo tamanho.
- **Rotate** apenas gira a cena. Não mexe em tamanho nenhum.

No Circle, o Wave modula a **altura** dos braços com onda triangular — sobe e desce linearmente, do maior para o menor, e todas as pontas ficam na mesma circunferência. Na Pyramid, o Wave **gira** cada nível, mais rápido quanto mais alto.

### Cores

Cor é propriedade da **face, no espaço da própria peça**: cada aresta do footprint carrega um papel fixo (arestas opostas iguais, adjacentes diferentes; os braços do Circle declaram flanco / ponta / flanco / interna). Nada no preenchimento lê a câmera ou a fase, então **nenhuma face troca de cor durante a animação** — não há luz simulada.

Papéis por paleta: tampa, dois flancos, ponta e miolo. São 7 paletas — Stack, Ziggurat, Ember, Cyan, Lime, Ink e Paper.

**Monocromático** reconstrói tudo a partir de uma cor base, com a rampa tonal na direção do fundo: em paleta escura os flancos escurecem, em paleta clara clareiam. No preto sobre preto a rampa inverte o sentido, senão a peça desapareceria no fundo.

**Sombra** é uma pista de profundidade estática por papel de face, não uma luz. Padrão 0 — totalmente chapado.

### Câmera e enquadramento

Arrastar orbita, `shift`+arrastar (ou botão direito) move, scroll dá zoom; e há sliders de elevação, giro e zoom para valores exatos.

O **guia de enquadramento** mostra exatamente a área que vai ser exportada, no formato escolhido, com o resto escurecido. Em zoom 100 a cena preenche o quadro. O enquadramento é calculado a partir da bounding box amostrada ao longo do ciclo (e do giro, quando Rotate está ligado), então ele não "respira" durante a animação: a câmera que você posiciona é a que sai no arquivo.

### Export

**Frame** é um scrubber sobre os frames reais do clipe (duração × fps): arrastar pausa e define a fase, e o PNG sai naquele instante exato.

O **WebM** não é captura de tela — cada frame é renderizado num canvas offscreen na resolução final e empurrado via `captureStream(0)` + `requestFrame()`, gravado por MediaRecorder em VP9 (com fallback VP8). Sai em **loop perfeito**: a fase avança exatamente `2π × ciclos` ao longo da duração, e as rotações avançam em múltiplos da simetria da peça.

Formatos 1:1, 16:9, 9:16, 4:5 e tela cheia; o tamanho é o lado menor, na convenção usual — 1080 dá 1920×1080, 1080×1920, 1080×1350.

> WebM/VP9 não abre nativamente no Premiere nem no After Effects. Para edição, converta: `ffmpeg -i entrada.webm -c:v prores_ks saida.mov`

### Atalhos

`1`–`5` layout · `espaço` play/pause · `M` monocromático · `G` guia · `N` nova semente

---

## Licença

MIT — ver [LICENSE](LICENSE).
