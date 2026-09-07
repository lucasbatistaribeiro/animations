# Gerador

Gerador de assets generativos — formas 2D e blocos extrudados em 3D.

**▶ [lucasbatistaribeiro.github.io/animations](https://lucasbatistaribeiro.github.io/animations/)**

A home **é** o gerador: canvas em tela cheia, barra de ações embaixo e, acima dela, a caixa que a barra abre — painel, paleta ou menu, uma de cada vez. Não há página intermediária para escolher entre 2D e 3D — os dois convivem na aba *Assets*, como grupos de template.

| | |
|---|---|
| **Assets** | busca e os templates, agrupados em `2d` e `3d`. Cada miniatura é desenhada pelo próprio motor, então mostra o template de verdade |
| **Editor** | os controles do template ativo, em cards: forma, câmera (no 3D), movimento e saída |
| **Barra** | o hambúrguer esconde e mostra o painel; play/pause; sol/lua para o tema; paleta; e **Exportar**, que abre o menu de ações e lista os atalhos |

Um arquivo HTML autocontido: sem dependências, sem build, sem servidor. Só os downloads exigem `http(s)`.

### Como usar

| | |
|---|---|
| escolher | aba **Assets**, clique numa capa. O painel **fica na aba**, para você percorrer vários templates seguidos; o Editor está a uma aba de distância quando a escolha estiver feita |
| ajustar | aba **Editor**: *Forma*, *Câmera* (só nos 3D), *Movimento* e *Saída* |
| orbitar | nos templates 3D, arraste no canvas; a roda do mouse dá zoom |
| paleta | botão das bolinhas na barra: troca a paleta e mostra a rampa de tons. A bolinha na rampa marca o acento; clicar num tom troca |
| exportar | botão **Exportar**: abre o menu com PNG, SVG (2D), WebM (3D), nova semente, copiar link e as duas saídas em HTML |
| levar para um site | **Copiar HTML** (2D) cola a peça em vetor dentro do seu HTML; **Copiar embed** cola um `<iframe>` que mantém a animação. Os dois vêm com o CSS junto |
| esconder o painel | o botão do hambúrguer. Painel, paleta e menu dividem o mesmo lugar: abrir um fecha o outro, e a caixa que entra espera a que sai terminar de sair |

| link | o estado vai no hash da URL: recarregar não perde nada e o link é compartilhável. **Copiar link** está no menu |

`espaço` play/pause · `N` nova semente · `E` exporta PNG · `H` esconde o painel · `/` busca · `esc` fecha o que estiver aberto

Os mesmos atalhos estão no rodapé do menu **Exportar** — descobrir um atalho não depende de ler isto aqui.

---

## Estrutura

```
.
├── index.html                    # o gerador — a home
├── test.html                     # teste de fumaça
├── DESIGN.md                     # o inventário do design system da casca
├── LICENSE                       # MIT
└── README.md
```

O Pages serve a branch `main` a partir da raiz. O gerador é **um arquivo só**: os dois motores vivem em IIFEs separadas dentro dele, e não há mais cópia do código em lugar nenhum.

---

## Templates 2D

Três formas chapadas, cada uma construída a partir de uma única figura que se repete em variantes. Os controles abaixo são os que aparecem no card *Forma* do Editor.

### Arc Bands

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

### Barcode

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

### Cadence

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
| **Animação** | os modos disponíveis mudam com o template. Arc Bands: `deslizar` (as cordas sobem e descem, e cada peça atravessa suas variantes), `sanfona`, `pulsar`, `girar`. Barcode: `caindo`, `andar`, `pulsar`, `sanfona`. Cadence: `correr`, `pulsar` |
| **Cores** | 8 paletas, em duas cores (acento e fundo) ou usando a paleta inteira, uma cor por faixa |
| **Semente** | irregulariza a fila sem sair do sistema |

O **SVG** usa a mesma primitiva e a mesma ordem de pintura do canvas, então o vetor não sai diferente da tela.

---

## Templates 3D

Renderer axonométrico próprio em canvas 2D, com *painter's algorithm*. A unidade é uma **peça**: um footprint 2D em sentido anti-horário mais `z0`/`z1`. Cada layout só descreve polígonos e alturas — prisma, cor de face e ordenação por profundidade são compartilhados.

### Os cinco templates

| template | geometria | `Effector` | `Lóbulos` | folga |
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

### Câmera

Arraste no canvas para orbitar, roda do mouse para zoom, e os sliders de *Elevação* e *Zoom* no card Câmera para valores exatos.

O enquadramento é calculado a partir da bounding box amostrada ao longo do ciclo — e do giro, quando Rotate está ligado. Então ele **não "respira"** durante a animação: a câmera que você posiciona é a que sai no arquivo.

### WebM

O WebM **não é captura de tela** — cada frame é renderizado num canvas offscreen na resolução final e empurrado via `captureStream(0)` + `requestFrame()`, gravado por MediaRecorder em VP9 (com fallback VP8). Sai em **loop perfeito**: a fase avança exatamente `2π × ciclos` ao longo da duração, e as rotações avançam em múltiplos da simetria da peça.

O tamanho segue a convenção usual, pelo lado menor: 1080 dá 1920×1080, 1080×1920, 1080×1350.

> WebM/VP9 não abre nativamente no Premiere nem no After Effects. Para edição, converta: `ffmpeg -i entrada.webm -c:v prores_ks saida.mov`

---

## Export

| | |
|---|---|
| **PNG** | todos os templates. 2D sai em 2× do formato escolhido (até 3840×2160 no wide); 3D sai na resolução configurada |
| **SVG** | só nos templates 2D — a mesma primitiva e a mesma ordem de pintura do canvas |
| **WebM** | só nos templates 3D, em loop perfeito. Duração, fps, ciclos e o scrubber de frame ficam no card *Vídeo* |
| **Copiar HTML** | só nos templates 2D. Vai para a área de transferência um `<figure>` com o SVG inline e o `<style>` junto: HTML e CSS puros, sem JS, sem depender de nada continuar no ar. Congela um quadro |
| **Copiar embed** | todos os templates. Um `<iframe>` apontando para esta página com o estado no hash, mais o CSS que segura a proporção antes de carregar. Mantém a animação, ao preço de o site passar a depender daqui |

O embed abre a página com `?embed`: mesma página, sem barra, sem painel, sem teclado nem zoom pela roda — dentro de um retângulo alheio, a UI seria um controle que não é de quem hospeda, e roubar a rolagem do site seria pior ainda. A classe entra no `<html>` por um script no `<head>`, antes do primeiro quadro, senão a barra pisca antes de sumir.

O endereço do embed é o de onde a página está servida. Aberta do disco não há endereço que sirva a um site, então cai para o Pages.

Nos templates 3D o export recorta pelo **guia de enquadramento**: o botão no card *Saída* mostra exatamente a área que vai para o arquivo, com o resto escurecido. O guia é pintado por cima da cena e nunca entra no export.

---

## A casca

> O inventário completo — os 63 tokens, as 13 primitivas, os cinco estados, as cinco leis e o contraste medido de cada par — está em **[DESIGN.md](DESIGN.md)**. Aqui fica só o resumo.

Tem **dois temas**. O `prefers-color-scheme` do sistema manda, e o botão de sol/lua na barra passa por cima — a escolha fica no navegador de quem visita, e não no link, porque um link compartilhado não deve impor o tema de quem o mandou. A rampa não inverte de valor, inverte de sentido: no escuro cada nível sobe, no claro cada nível desce, porque um campo dentro de um card é um recesso. São 23 dos 63 tokens redefinidos.

Escura por padrão, em **cinza médio** e com **elevação**: painel `#262626`, card `#333`, campo `#404040` — degraus de uma mesma rampa, em vez de três quase-pretos separados só pela borda. Cinza puro, sem viés de matiz: a única cor da tela é a arte gerada, e o acento da UI é claro (`#f0f0f0`), não colorido. Painel, barra e popovers são translúcidos com desfoque — a arte atravessa a casca, e a opacidade é 92% porque abaixo disso, com paleta clara atrás, o texto secundário cai de 4.5:1.

O movimento é do sistema, não de cada componente: dois *easings* e três durações em variáveis CSS, usados em tudo. O que ele faz:

| | |
|---|---|
| **tinta das abas** | desliza de uma aba para a outra, em vez de piscar de lugar |
| **entrada em cascata** | os cards sobem em sequência — mas **só quando o conteúdo troca de verdade** (aba, template, motor). Um clique num toggle refaz o mesmo conteúdo: ali a cascata seria uma piscada |
| **trilha do slider** | pintada até o valor, e o número acende enquanto se arrasta: o olho está no canvas, e o valor se anuncia sem exigir um segundo olhar |
| **popovers** | saíram de `display:none` para poder animar — e, de quebra, dá para medir a caixa e posicionar **antes** de mostrar |
| **aviso** | copiar link, sortear semente e gravar WebM acontecem fora da tela e não davam sinal nenhum. O aviso é o recibo |

Refazer o painel não custa mais a rolagem nem o foco do teclado: cada controle carrega uma âncora (`data-fk`) e o `render` devolve os dois quando o conteúdo é o mesmo. E há **anel de foco** em tudo — antes dava para percorrer a UI inteira no teclado sem ver onde se estava.

`prefers-reduced-motion` desliga todas as transições e animações. Movimento aqui é acabamento, nunca requisito.

Cor, sombra, tempo, raio, tipo e espaço: **todo valor tem nome**. Não há número de estilo escrito à mão no CSS, no JS ou no markup — e o que fica de fora, fica documentado com o porquê em [DESIGN.md](DESIGN.md).

---

## Como um motor entra na aplicação

A casca não sabe nada de dentro dos motores — nem quantos templates existem, nem o que cada controle faz. Ela conversa com eles por um **adaptador de 23 métodos**, documentado num bloco no topo do `index.html`. Implementá-lo é tudo o que um motor novo precisa fazer.

O bloco cobre identidade, templates, desenho, controles, estado, cores e saída, mais os **opcionais** — direções, modos, toggles, câmera, órbita, zoom, SVG e vídeo. A casca detecta opcional por **capacidade, nunca por identidade do motor**: se o método existe, o controle aparece.

Três invariantes que a assinatura não conta e que já custaram bug:

- **`draw` pinta o quadro inteiro**, fundo incluído — a casca nunca limpa o canvas antes;
- **`advance` responde se algo mudou**. Responder `true` à toa custa um repaint por quadro para sempre; responder `false` quando mudou congela a tela;
- **`setLayout` reaplica o preset** do template, descartando os ajustes do usuário. Quem só quer espiar um template tem de usar `snapshot`/`restore` em volta. Foi esse detalhe que fez a busca da aba Assets apagar em silêncio os ajustes de quem digitava.

A constante `CONTRATO` do `test.html` é essa mesma lista em forma executável, então a documentação não pode divergir do que é cobrado sem o teste acusar.

---

## Teste de fumaça

**[/test.html](https://lucasbatistaribeiro.github.io/animations/test.html)** varre os dois motores e imprime uma tabela: contrato do adaptador, todos os templates em cada modo, direção, contagem mínima e máxima e três fases, todas as paletas, e a integridade do estado. São ~290 casos de desenho em menos de um segundo.

Ele dirige o `index.html` **de verdade**, dentro de um iframe — não uma cópia do código, senão passaria enquanto o app quebra. Por isso precisa de `http(s)`: em `file://` cada arquivo é uma origem opaca e o navegador bloqueia o acesso ao iframe. Localmente, `python -m http.server` na raiz resolve — e é isso que o `.claude/launch.json` sobe, para quem abrir o repo no Claude Code.

O que ele checa em cada caso: não lançar, não deixar pixel transparente, e não sair um quadro chapado de uma cor só. E fora do desenho: que `snapshot`/`restore` devolvam o estado exato e que `set`/`get` não divirjam.

> A verificação do estado usa **dois valores sentinela** em vez de comparar com o estado ambiente. Uma corrupção constante contamina a própria linha de base — foi o que aconteceu na primeira versão desta checagem, que passava enquanto o bug estava injetado.

---

## Licença

MIT — ver [LICENSE](LICENSE).
