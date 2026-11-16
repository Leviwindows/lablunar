# Changelog — LabLunar

Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

## Nota sobre o protótipo de referência

O brief dizia que já existia um `index.html` de referência na pasta. **Esse arquivo não estava lá**
(a pasta `lablunar/` foi criada vazia, com apenas `data/`). A busca não encontrou nenhum protótipo do
jogo no projeto — os arquivos vizinhos são o deck de slides e o vídeo animatic, de outro escopo.

Por decisão do autor, seguimos **sem** o protótipo e registrando a ausência aqui e no README.
Consequência prática: este `index.html` foi escrito do zero a partir do brief, então não existe
"diff contra o protótipo".Quando o protótipo aparecer, este changelog deve ganhar uma seção
`## [0.2.0] — Comparação com o protótipo` listando o que foi mantido, alterado e removido.

---

## [0.3.0] — 2026-11-15

Idioma da interface e ajustes de leitura.

### Alterado

- **Interface traduzida para o inglês**: telas, botões, banners, registro do dia, relatório, gráficos e
  rodapé. Nomes internos (`economia`, `rigoroso`, `oxigenio`, `radiacao`, `energia`) foram mantidos para
  não quebrar o motor nem os testes; só o texto visível mudou. `<html lang>` e `<title>` também.
- **Rótulos dos modos** em `data/game_constants.json` passaram a `Economy` e `Power-save`.
- **Radiação com uma casa decimal** na barra, no registro e nos gráficos: antes a dose 99,6% aparecia
  como “100%” com a missão ainda viva, o que parecia contraditório.

### Motivo

O jogo foi submetido ao NASA Space Apps Challenge, cujo júri é internacional; o público-alvo original
(7–12 anos, Brasil) continua atendido, mas os textos agora estão em inglês.

---

## [0.2.0] — 2026-11-15

Dados solares reais e novo balanceamento.

### Adicionado

- **Coleta real do DONKI com chave pessoal**: 27 eventos SEP e 88 alertas FLR da janela
  2024-04-25..2024-06-18 (HTTP 200 nos dois endpoints), registrados em `data/sources.md` com URL,
  data e código HTTP. `sep_status` e `flr_status` passaram a `ok`.
- **`--key-file`** em `tools/fetch_donki.py`: a chave pode ficar em `secrets/nasa_key.txt` (pasta no
  `.gitignore`); o script só informa o tamanho da chave, nunca o valor, e recusa o texto de exemplo.
- **`.gitignore`** cobrindo `secrets/`, `.env*` e temporários de coleta.

### Alterado

- **Seleção de janela**: o plano de tempestades agora prefere janelas com pelo menos 3 datas distintas
  de evento SEP, para a missão ter mais de uma tempestade real (antes, janelas com 2 eventos no mesmo
  dia geravam missões com uma única tempestade).
- **Radiação de tempestade**: 28 → **32 %/dia**. Com os 27 eventos reais, ignorar o aviso passou a
  derrubar a taxa de vitória de 84% para 43,3%, sem tornar a derrota automática.
- **Balanceamento re-medido com calendário real** (1.000 partidas por configuração): base padrão 84,1%;
  ignorando avisos 43,3%; sem reciclagem, sem estufa, bateria pequena e sem escudo 0%.
  As causas de derrota na base padrão são oxigênio (136/1000) e energia (23/1000).
- **Aviso garantido**: a missão começa 3 dias antes da primeira tempestade sorteada, então o aviso do
  dia anterior sempre tem onde aparecer (antes, a missão podia começar no meio de uma tempestade).
- **Contador de dia e fase**: a missão mostrava “Dia 1” com “Sol — dia 2 de 14”. Agora o dia exibido é
  o próximo a jogar e a fase corresponde a ele (dia 14 = último de sol; dia 15 = noite 1 de 14).
- **Prévia da base**: “dá para 15.0 noite(s) de 14 dias” virou “cobrem as 14 noites com folga”.

---

## [0.1.0] — 2026-11-15

Primeira versão jogável.

### Adicionado

- **Jogo completo em um único `index.html`**: HTML, CSS e JavaScript, sem backend, sem login e sem
  instalação. Único recurso externo: Google Fonts (Nunito).
- **Quatro telas**: introdução → montagem da base → missão → relatório.
- **Montagem com orçamento de massa (100)**: painéis (4), baterias (6), estufas (8), recicladora (6) e
  escudo nenhum/leve/pesado/parede d'água (0/5/9/12), com prévia do balanço diário médio e de quantas
  noites a bateria cobre.
- **Ciclo de 14 dias de sol e 14 de noite**, repetido até 56 dias; painéis só produzem de dia e a
  bateria é a única fonte à noite.
- **Três recursos visíveis**: energia (kWh), oxigênio (%) e radiação (dose acumulada, %). Água, comida
  e tripulação ficam internos.
- **Escolhas diárias**: modo de energia (normal / economia / rigoroso) e abrigar a tripulação
  (só com escudo).
- **Tempestades solares** com aviso um dia antes e radiação alta por dois dias.
- **Falhas de equipamento** aleatórias, consertadas com peças de reposição (3 no início).
- **Derrotas**: oxigênio em 0, dose acima de 100% e bateria zerada por 3 dias seguidos.
  **Vitória**: sobreviver aos 56 dias.
- **Relatório final**: resultado, post-mortem em linguagem simples, um gráfico por recurso e
  rejogabilidade (mesma base ou base nova).
- **Harness de balanceamento**: `LabLunar.runAll(n)` / `index.html?sim=1000`, com política de jogo
  atento usada nas medições.
- **Rodapé e tela final** com a atribuição de dados (DONKI, OSDR, Artemis) e o aviso de que os valores
  de radiação são ilustrativos.
- **`data/`**: `game_constants.json`, `donki_snapshot.json`, `osdr_snapshot.json`,
  `artemis_snapshot.json` e `sources.md` com URL exata, data e código HTTP de cada requisição.
- **`tools/`**: `fetch_donki.py` (coleta real, lê `NASA_API_KEY` do ambiente e nunca grava a chave),
  `build_data.py` (monta os snapshots) e `build.py` (injeta os dados no HTML para rodar via `file://`).

### Alterado

- Balanceamento calibrado com **1.000 simulações por configuração**:
  base padrão com jogo atento **83,2%** de vitória; ignorando os avisos de tempestade **8,4%**;
  sem reciclagem, sem estufa, com bateria pequena ou sem escudo **0%**.
  Ajustes que produziram isso: radiação de tempestade 8 → 28 %/dia, consumo de oxigênio por tripulação
  5,0 → 5,4 %/dia, chance de falha 6% → 9%/dia.

### Corrigido durante o desenvolvimento

- O build injetava os dados entre marcadores de comentário, então o `JSON.parse` recebia
  `/*DATA_START*/` e a página quebrava. Agora o `build.py` delimita pela própria tag
  `<script id="game-data">` e valida o JSON antes de terminar.
- O motor usava FLR como fonte alternativa de tempestade. **Removido**: SEP é a única fonte de
  tempestade; FLR só rotula o aviso do dia anterior.

### Limitações conhecidas (v0.1.0)

- **SEP em `pending_download`**: a coleta em massa exige `NASA_API_KEY`, que não estava configurada; a
  `DEMO_KEY` compartilhada respondeu HTTP 429. O jogo **não criava tempestades** e avisava na tela.
  Resolvido na 0.2.0 com a chave pessoal.
- Os 14 alertas FLR salvos não puderam ser reproduzidos (HTTP 429) e ficaram marcados
  `pending_download`; foram aposentados na 0.2.0, quando 88 alertas reais foram baixados.
- Dia e noite são 14/14 (arredondado de 29,5 dias); a realidade é ~14,75.
- Dose, massa, kWh e taxas de falha são valores de design, não medições.
