# Stack Generators

Dois geradores de assets: formas geométricas que se reformatam, reorganizam e reconstroem, inspirados no sistema generativo do rebrand da Stack Overflow.

**▶ [lucasbatistaribeiro.github.io/animations](https://lucasbatistaribeiro.github.io/animations/)**

| | |
|---|---|
| [Arc Bands 2D](https://lucasbatistaribeiro.github.io/animations/stack-generator/) | Faixas de cor chapada delimitadas por arcos |
| [Stacks 3D](https://lucasbatistaribeiro.github.io/animations/stack-generator/stacks-3d.html) | Blocos extrudados em projeção axonométrica, cinco layouts |

Cada gerador é **um arquivo HTML autocontido**: sem dependências, sem build, sem servidor. Dá para abrir direto do disco — só o "Copiar link" e os downloads exigem `http(s)`, por causa das restrições de contexto seguro do navegador.

---

## Estrutura

```
.
├── index.html                    # capa servida pelo GitHub Pages
├── stack-generator/
│   ├── index.html                # Arc Bands 2D
│   └── stacks-3d.html            # Stacks 3D
├── html.html                     # demo antigo: barra de progresso em CSS
├── style.css                     # css do demo antigo
├── LICENSE                       # MIT
└── README.md
```

O Pages serve a branch `main` a partir da raiz.

---

## Arc Bands 2D

A primitiva do sistema não é a faixa: é o **arco**. Cada arco pinta todo o meio-plano atrás de si, e a fila é pintada do último para o primeiro, alternando acento e fundo. Cada arco cobre o anterior, e o que sobra entre dois vizinhos é uma faixa de altura cheia — reta de um lado, curva do outro. Nenhuma faixa é desenhada diretamente.

Duas consequências úteis dessa construção:

- as faixas **não afinam** até virar ponta, como aconteceria com discos completos: elas chegam ao topo e à base com largura;
- o corte de cada arco na linha do meio fica exatamente um `passo` à frente do anterior — o raio se cancela na conta. Ou seja, **`Passo` controla a largura das faixas e `Raio` controla só a curvatura**, sem um interferir no outro.

### Controles

| | |
|---|---|
| **Arcos** | quantas fronteiras, logo quantas faixas |
| **Raio** | curvatura: raio grande deixa o arco quase reto, raio pequeno abauda |
| **Decaimento** | como o raio muda de um arco para o próximo. Perto de 100% os arcos ficam paralelos; longe disso as curvaturas divergem e as faixas se estrangulam nas pontas |
| **Passo** | distância entre cortes, isto é, a largura das faixas. Aceita negativo, e a fila inverte o sentido |
| **Início** | onde o primeiro arco corta o quadro |
| **Ângulo** | direção da fila: faixas verticais, horizontais ou diagonais |
| **Origem** | desloca o centro dos arcos na perpendicular, inclinando a composição |
| **Formato** | 16:9, 1:1, 9:16, 4:5, 3:1 |
| **Animação** | `deslizar` a fila vai e volta · `sanfona` o passo abre e fecha · `pulsar` o raio respira · `girar` a direção roda · estático |
| **Cores** | 6 paletas, em duas cores (acento e fundo) ou usando a paleta inteira, uma cor por faixa |
| **Semente** | irregulariza os cortes e as curvaturas sem sair do sistema |

`espaço` gera · `F` congela · `E` exporta PNG · clique na arte põe o primeiro corte ali.

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
