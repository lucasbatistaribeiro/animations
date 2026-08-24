# Stack Generators

Dois geradores de assets construídos sobre a mesma ideia — blocos que se reformatam, reorganizam e reconstroem — inspirados no sistema generativo do rebrand da Stack Overflow.

**▶ [lucasbatistaribeiro.github.io/animations](https://lucasbatistaribeiro.github.io/animations/)**

| | |
|---|---|
| [Block Grid 2D](https://lucasbatistaribeiro.github.io/animations/stack-generator/) | Grade de blocos construtivos que se reconstrói em tempo real |
| [Stacks 3D](https://lucasbatistaribeiro.github.io/animations/stack-generator/stacks-3d.html) | Blocos extrudados em projeção axonométrica, cinco layouts |

Cada gerador é **um arquivo HTML autocontido**: sem dependências, sem build, sem servidor. Dá para abrir direto do disco — só o "Copiar link" e os downloads exigem `http(s)`, por causa das restrições de contexto seguro do navegador.

---

## Estrutura

```
.
├── index.html                    # capa servida pelo GitHub Pages
├── stack-generator/
│   ├── index.html                # Block Grid 2D
│   └── stacks-3d.html            # Stacks 3D
├── html.html                     # demo antigo: barra de progresso em CSS
├── style.css                     # css do demo antigo
├── LICENSE                       # MIT
└── README.md
```

O Pages serve a branch `main` a partir da raiz.

---

## Block Grid 2D

Grade de células sorteadas a partir de uma semente, onde cada célula recebe uma das **18 formas construtivas** derivadas da geometria do logo: cheio, meio, terço, sexto, quarto, diagonal, barras, escada, bandeja, notch, moldura, quarto de círculo (dois tamanhos), arco, ponto, cruz, T e L.

### Controles

| | |
|---|---|
| **Colunas** | resolução da grade; as linhas vêm do formato escolhido |
| **Densidade** | proporção de células ocupadas |
| **Respiro** | folga em volta de cada bloco |
| **Variedade** | peso do repertório expressivo (curvas, notch, moldura) contra o estrutural (barras e retângulos) |
| **Formato** | 1:1, 16:9, 9:16, 4:5, 3:1 |
| **Modo** | `rebuild` blocos aleatórios · `cascade` onda diagonal · `columns` · `rows` · `shuffle` troca de posições · estático |
| **Ritmo / Intensidade** | frequência das reconstruções e quantos blocos entram em cada uma |
| **Transição** | `slide` · `pop` · `cut` |
| **Paleta** | 6 opções |
| **Semente** | reproduz a composição inicial |

`espaço` gera · `F` congela · `E` exporta PNG · clique na arte reconstrói os blocos sob o cursor.

### Export

PNG 1× e 2× (até 4800 px no banner), **SVG vetorial** e link compartilhável com o estado no hash da URL.

### Como funciona

As formas são definidas **uma única vez** em espaço unitário, através de uma abstração de caneta com duas implementações — uma escreve num `Path2D` para o canvas, a outra monta uma string `d` de SVG. Canvas e SVG compartilham a mesma geometria, então o vetor nunca sai diferente do que está na tela.

Cada célula guarda a forma anterior e a nova; na transição a antiga desliza para fora enquanto a nova entra, com `clip` na célula e atraso escalonado por posição.

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
