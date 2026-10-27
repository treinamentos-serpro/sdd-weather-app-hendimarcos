# Weather App — Especificação de Produto

> Versão 1.0 — 27/10/2026. Fonte única da verdade para Plan, Tasks, Code e
> Test. Histórico de decisões e análise de discovery em
> [`discovery.md`](./discovery.md).

## Overview

- **Produto:** aplicação web responsiva para consultar o tempo de uma cidade.
- **Objetivo:** buscar uma cidade, ver o clima atual e a previsão de hoje e dos
  quatro dias seguintes, em °C ou °F, sem cadastro.
- **Personas (hipóteses a validar):** Marina planeja o dia pelo celular;
  Rafael consulta destinos de viagem; Cláudia consulta, uma de cada vez, a
  própria cidade e as de familiares. Detalhes em `discovery.md`.
- **Fonte de dados:** Open-Meteo (geocodificação + previsão), sem API key,
  licença CC BY 4.0 com atribuição obrigatória.
- **Escopo v1:** uma cidade por vez; estado apenas em memória; interface em
  pt-BR. Tudo que não está nos requisitos abaixo está em **Out of Scope**.

## Glossário

- **Bloco:** uma das três áreas independentes da interface — **busca**,
  **clima atual** e **previsão**. Cada bloco tem seus próprios estados.
- **Estados:** *carregamento* e *vazio* usam `role="status"`; *erro* usa
  `role="alert"` e contém o botão "Tentar novamente". Nos blocos de clima e
  previsão, esses estados são renderizados **dentro** da `region` do bloco,
  que persiste em todos os estados e é o alvo de foco.
- **Termo:** texto do campo de busca após remover espaços nas extremidades.
- **Sessão:** período entre carregar a página e fechá-la ou recarregá-la.
- **D:** primeira data retornada pela fonte em `daily.time` (hoje no fuso da
  cidade).
- **Fonte:** API pública da Open-Meteo (Apêndice A).

## Estrutura da interface (v1)

Ordem vertical fixa, mobile-first. A direção visual (tema dark glassmorphism,
paleta, grid da previsão) é normativa e está em
`.github/instructions/tailwind.instructions.md`.

1. **Cabeçalho** — título `h1` "Previsão do Tempo" e o controle de unidade
   (sempre visível, Celsius ativo por padrão).
2. **Busca** — campo rotulado "Cidade" e botão "Buscar".
3. **Lista de resultados** — aparece abaixo da busca após uma busca com
   resultados; permanece até uma seleção ou nova busca; não há fechamento
   manual (Esc) na v1.
4. **Clima atual** — `region` "Clima atual"; ausente até a primeira seleção.
5. **Previsão** — `region` "Previsão"; grid responsivo de 5 entradas; ausente
   até a primeira seleção.
6. **Rodapé** — atribuição à Open-Meteo com link.

Blocos 4 e 5 permanecem visíveis enquanto a lista (3) está aberta.

## Functional Requirements

- **RF-01 — Buscar e selecionar cidades.** O usuário digita um termo e envia
  explicitamente (Enter ou botão "Buscar"); digitar não dispara requisição.
  Termos com menos de 2 caracteres não disparam requisição. A lista exibe até
  10 resultados; cada um mostra nome, região administrativa (`admin1`) e país,
  quando presentes, para distinguir homônimos. Selecionar um resultado define a
  cidade ativa, remove a lista e **sempre** dispara nova consulta
  meteorológica, mesmo que a cidade já estivesse ativa.
- **RF-02 — Consultar clima atual.** Para a cidade ativa, exibir temperatura
  atual, condição do tempo em pt-BR (Apêndice B) e horário da última
  atualização informado pela fonte.
- **RF-03 — Consultar previsão.** Para a cidade ativa, exibir cinco entradas
  diárias (D a D+4, fuso da cidade), cada uma com data, máxima, mínima e
  condição. A primeira entrada é rotulada "Hoje"; as demais, dia da semana
  abreviado e `dd/MM` (ex.: "qua., 28/10").
- **RF-04 — Alternar unidade de temperatura.** Par de botões Celsius /
  Fahrenheit. Afeta todas as temperaturas visíveis; valores arredondados ao
  inteiro mais próximo (meio para cima). Inicial: Celsius. Válido apenas na
  sessão. A alternância é local e não dispara requisição.

### Restrições técnicas

Não geram histórias de usuário; têm critérios de aceite próprios (AC-05.x,
AC-06.x) para garantir conformidade.

- **RF-05 — Fonte de dados.** Geocodificação e previsão vêm exclusivamente da
  Open-Meteo, com `timezone=auto`, sem API key, e com atribuição visível
  conforme a licença CC BY 4.0.
- **RF-06 — Sem conta e sem persistência.** Nenhuma autenticação, nenhum
  servidor próprio, nenhum uso de `localStorage`, `sessionStorage` ou cookies.

## User Stories

- **US-01 (RF-01):** Como Cláudia, que consulta localidades de familiares,
  quero buscar e selecionar uma cidade para ver as condições do local que
  procuro.
- **US-02 (RF-02):** Como Marina, que planeja o dia antes de sair, quero ver o
  clima atual da cidade selecionada para decidir o que vestir ou levar.
- **US-03 (RF-03):** Como Rafael, que planeja uma viagem, quero ver a previsão
  de hoje e dos quatro dias seguintes no destino para me preparar.
- **US-04 (RF-04):** Como Marina, que precisa interpretar rapidamente as
  temperaturas, quero alternar entre Celsius e Fahrenheit para usar a unidade
  que prefiro.
- **US-05 (RF-02, RF-03, RNF-07):** Como Marina, quando a fonte de dados falhar
  ou demorar, quero ver claramente que a informação não está disponível e poder
  tentar de novo, para não decidir com base em dado errado ou antigo.
- **US-06 (RF-01):** Como Rafael, que viaja para cidades com nomes repetidos,
  quero ver região e país em cada resultado para selecionar o destino certo.

## Acceptance Criteria

**Convenções.** Cada critério descreve um resultado observável na interface ou
no tráfego de rede, com a fonte simulada. Estados e seções são localizados por
papel e nome acessível (Glossário); os textos estão no Apêndice C e podem
mudar sem afetar os testes, exceto onde o critério cita o literal. Timeout por
requisição: **10 s**. Temperaturas renderizadas são sempre inteiras, no formato
`"21°C"` / `"70°F"` (sem espaço), com sinal negativo em hífen-menos ASCII
(`"-5°C"`); zero negativo renderiza `"0°C"`.

**Modelo de requisições.** A busca usa uma requisição própria. Clima atual e
previsão vêm de **uma única requisição** (Apêndice A). Consequências:
falhas de transporte (HTTP de erro, rede, timeout) afetam os dois blocos
simultaneamente; falhas de **conteúdo** (campo ausente, arrays desalinhados)
são avaliadas por bloco, e um bloco pode ficar em erro enquanto o outro exibe
dados. "Tentar novamente" em qualquer dos dois blocos reenvia essa requisição
única e coloca ambos em carregamento. Critérios que mencionam "a previsão
falha" com clima atual válido (AC-03.5) referem-se a falhas de conteúdo.

### Transversais

- **AC-00.1 — Tentar novamente:** **Given** um bloco está em estado de erro,
  **When** o usuário aciona o botão "Tentar novamente" desse bloco, **Then** a
  requisição correspondente é reenviada, todos os blocos que dependem dela
  entram em carregamento, o foco permanece dentro do bloco (não recai no
  `body`) e, se a fonte responder com sucesso, os blocos exibem os dados.
- **AC-00.2 — Sem artefatos de renderização:** **Given** qualquer tela em
  qualquer estado, **When** o texto visível é inspecionado, **Then** não contém
  `undefined`, `NaN`, `null` nem `[object Object]`.

### RF-01 — Buscar e selecionar cidades (US-01, US-06)

- **AC-01.0 — Estado inicial:** **Given** o usuário abre a aplicação, **When**
  a página termina de carregar, **Then** o `h1`, o controle de unidade (com
  "Celsius" ativo), o campo de busca focável e o rodapé de atribuição estão
  visíveis; nenhuma lista, seção de clima atual ou previsão é renderizada e
  nenhum estado de erro ou vazio é exibido.
- **AC-01.1 — Busca com resultados:** **Given** a fonte retorna ao menos uma
  correspondência, **When** o usuário envia o termo, **Then** é exibida uma
  lista com um ou mais resultados selecionáveis, cada um apresentando o nome
  da cidade e, quando presentes, `admin1` e país (ex.: "Londrina, Paraná,
  Brasil").
- **AC-01.2 — Seleção de resultado:** **Given** a lista está visível, **When**
  o usuário seleciona um resultado, **Then** o texto desse item (nome,
  `admin1` e país, quando presentes) passa a ser exibido na seção "Clima
  atual" como contexto da consulta e a lista é removida da tela.
- **AC-01.3 — Busca sem resultados:** **Given** a fonte retorna zero
  correspondências, **When** o usuário envia o termo, **Then** um estado vazio
  é exibido e nenhuma lista é apresentada. Se não havia cidade ativa, nenhuma
  seção de clima é renderizada; se havia, ela permanece (AC-01.13).
- **AC-01.4a — Carregamento aparece:** **Given** o usuário enviou um termo,
  **When** a resposta ainda não chegou, **Then** o estado de carregamento da
  busca é exibido.
- **AC-01.4b — Carregamento desaparece:** **Given** o estado de carregamento
  está visível, **When** a resposta chega (sucesso, vazio ou erro), **Then** o
  estado de carregamento é removido.
- **AC-01.5 — Erro na busca:** **Given** a fonte responde com erro HTTP (4xx,
  incluindo 429, ou 5xx) ou a requisição falha (rede, CORS), **When** o usuário
  envia o termo, **Then** um estado de erro é exibido e nenhum resultado é
  apresentado.
- **AC-01.6 — Termo insuficiente:** **Given** o termo está vazio ou tem menos
  de 2 caracteres, **When** o usuário tenta enviar, **Then** nenhuma requisição
  é enviada, nenhum estado de erro ou vazio é exibido e o foco permanece no
  campo.
- **AC-01.7 — Cidade inexistente:** **Given** o usuário envia "Xyzabc",
  **When** a fonte responde sem correspondências, **Then** o estado vazio ecoa
  o termo pesquisado, o campo mantém o termo e nenhum estado de erro é
  apresentado.
- **AC-01.8 — Caracteres especiais e acentos:** **Given** o usuário envia
  "São Paulo", "Baden-Baden" ou "Coeur d'Alene", **When** a requisição é
  inspecionada, **Then** o termo é enviado URL-encoded sem remoção de acentos
  ou pontuação, e o nome retornado é exibido com seus caracteres originais.
- **AC-01.9 — Texto externo renderizado como texto:** **Given** o termo ou um
  nome retornado contém marcação (ex.: `<b>x</b>`), **When** é exibido,
  **Then** aparece como texto literal e nenhum elemento HTML é criado.
- **AC-01.10 — Formatos de resposta vazia:** **Given** a fonte responde HTTP
  200 com corpo `{}` (sem `results`) ou `{ "results": [] }`, **When** a
  resposta é processada, **Then** ambos produzem o estado vazio de AC-01.3.
- **AC-01.11 — Timeout da busca:** **Given** a fonte não responde em 10 s,
  **When** o limite é atingido, **Then** a requisição é cancelada, o
  carregamento é removido e o estado de erro de AC-01.5 é exibido.
- **AC-01.12 — Apenas a última busca prevalece:** **Given** o usuário enviou
  "Lon" e, antes da resposta, enviou "London", **When** a resposta de "Lon"
  chega depois, **Then** a lista corresponde a "London" e a resposta obsoleta
  é descartada.
- **AC-01.13 — Erro na busca preserva a cidade ativa:** **Given** uma cidade
  está ativa com clima e previsão exibidos, **When** uma nova busca falha,
  **Then** a cidade e seus dados permanecem visíveis e apenas o bloco de busca
  exibe erro.
- **AC-01.14 — Resultados navegáveis por teclado:** **Given** a lista está
  visível, **When** o usuário navega só com o teclado, **Then** cada resultado
  recebe foco em sequência com indicador visível, é acionável com Enter ou
  Espaço e, após a seleção, o foco é movido para a seção "Clima atual" (ou seu
  estado de carregamento).
- **AC-01.15 — Homônimos distinguíveis:** **Given** a fonte retorna "Santa
  Cruz, Bolívia" e "Santa Cruz, Califórnia, Estados Unidos" com coordenadas
  diferentes, **When** a lista é exibida, **Then** os dois itens têm textos
  distintos; e **When** o usuário seleciona cada um, **Then** a requisição
  meteorológica usa a latitude e longitude daquele item.
- **AC-01.16 — Campos de desambiguação ausentes:** **Given** um resultado sem
  `admin1` e/ou `country`, **When** a lista é exibida, **Then** o item mostra
  apenas os campos presentes, sem vírgulas soltas.
- **AC-01.17 — Envio explícito:** **Given** o usuário digita "Lisboa" sem
  enviar, **When** 2 s se passam, **Then** nenhuma requisição foi enviada;
  **When** pressiona Enter ou aciona "Buscar", **Then** a requisição é enviada.
- **AC-01.18 — Limite de resultados:** **Given** a fonte retorna 10
  resultados, **When** a lista é exibida, **Then** exatamente 10 itens são
  apresentados, na ordem retornada.
- **AC-01.19 — Blocos persistem com a lista aberta:** **Given** uma cidade
  está ativa com clima e previsão exibidos, **When** o usuário envia uma nova
  busca com resultados, **Then** a lista é exibida e as seções "Clima atual" e
  "Previsão" da cidade ativa permanecem visíveis e inalteradas até uma
  seleção.
- **AC-01.20 — Reseleção dispara nova consulta:** **Given** a cidade X está
  ativa, **When** o usuário busca e seleciona X novamente, **Then** uma nova
  requisição de previsão é enviada e os blocos passam por carregamento.

### RF-02 — Consultar clima atual (US-02, US-05)

- **AC-02.1 — Exibição:** **Given** a fonte retorna dados atuais com sucesso,
  **When** a resposta é recebida, **Then** uma `region` "Clima atual" exibe o
  nome da cidade, a temperatura com unidade, a condição em texto e o horário
  da última atualização.
- **AC-02.2 — Carregamento:** **Given** uma cidade acabou de ser selecionada,
  **When** a resposta ainda não chegou, **Then** a `region` "Clima atual"
  contém um estado de carregamento no lugar do conteúdo.
- **AC-02.3 — Erro:** **Given** a fonte responde com erro HTTP (4xx, incluindo
  429, ou 5xx) ou a requisição falha, **Then** o estado de erro é exibido e
  nenhuma temperatura é apresentada como atual.
- **AC-02.4 — Timeout:** **Given** a fonte não responde em 10 s, **Then** a
  requisição é cancelada, o carregamento é removido e o estado de erro de
  AC-02.3 é exibido.
- **AC-02.5 — Temperatura ausente:** **Given** a fonte responde HTTP 200 sem
  `current.temperature_2m`, **Then** o bloco exibe o estado de erro de AC-02.3
  e a previsão, se recebida, continua visível.
- **AC-02.6 — Campo secundário ausente:** **Given** a fonte responde com
  temperatura mas sem `current.weather_code`, **Then** a temperatura é
  apresentada e a condição é omitida.
- **AC-02.7 — Troca de cidade durante carregamento:** **Given** o usuário
  selecionou A e, antes da resposta, selecionou B, **When** a resposta de A
  chega depois, **Then** o clima exibido corresponde a B.
- **AC-02.8 — Horário da última atualização:** **Given** a fonte retorna
  `current.time = "2026-10-27T14:45"`, **When** o clima é exibido, **Then** a
  seção mostra "14:45" independentemente do fuso do navegador; **When** o
  bloco está em erro, **Then** nenhum horário é exibido.
- **AC-02.9 — Condição a partir do `weather_code`:** **Given** a fonte retorna
  `weather_code` 0, 61 ou 95, **Then** a condição exibida é, respectivamente,
  "Céu limpo", "Chuva fraca" e "Tempestade" (Apêndice B); **Given** um código
  não mapeado (ex.: 999), **Then** a condição é "Condição desconhecida".

### RF-03 — Consultar previsão (US-03, US-05)

- **AC-03.1 — Cinco dias consecutivos:** **Given** a fonte retorna ao menos
  cinco dias, **When** a previsão é exibida, **Then** uma `region` "Previsão"
  apresenta exatamente cinco entradas com datas D, D+1, D+2, D+3 e D+4, nessa
  ordem.
- **AC-03.2 — Conteúdo de cada dia:** **Given** as entradas estão visíveis,
  **Then** cada uma exibe sua data, máxima e mínima com unidade, e a condição
  (AC-02.9).
- **AC-03.3 — Carregamento:** **Given** uma cidade acabou de ser selecionada,
  **When** a resposta ainda não chegou, **Then** a `region` "Previsão" contém
  um estado de carregamento no lugar do conteúdo.
- **AC-03.4 — Erro:** **Given** a fonte responde com erro HTTP (4xx, incluindo
  429, ou 5xx) ou a requisição falha, **Then** o estado de erro é exibido e
  nenhuma entrada é apresentada.
- **AC-03.5 — Erro de conteúdo na previsão não afeta o clima atual:** **Given**
  a resposta contém `current` válido mas `daily` ausente ou inválido
  (AC-03.7), **Then** o clima atual é exibido e apenas o bloco de previsão
  exibe erro.
- **AC-03.6 — Timeout:** **Given** a fonte não responde em 10 s, **Then** a
  requisição é cancelada, o carregamento é removido e o estado de erro de
  AC-03.4 é exibido.
- **AC-03.7 — Arrays desalinhados:** **Given** a fonte responde com cinco datas
  e três máximas, **Then** o bloco exibe o estado de erro de AC-03.4.
- **AC-03.8 — Mais de cinco dias:** **Given** a fonte retorna sete dias,
  **Then** apenas D a D+4 são apresentados.
- **AC-03.9 — Data civil independente do fuso do navegador:** **Given**
  `daily.time` contém `"2026-10-27"`, **When** a previsão é exibida com o
  navegador em `America/Sao_Paulo` (UTC−3) ou `Pacific/Kiritimati` (UTC+14),
  **Then** a entrada exibe 27/10 em ambos os casos.
- **AC-03.10 — Rótulo e formato de data:** **Given** `daily.time` =
  `["2026-10-27", "2026-10-28", …]`, **Then** a primeira entrada exibe "Hoje"
  e a segunda exibe "qua., 28/10" (dia da semana abreviado em pt-BR + `dd/MM`).
- **AC-03.11 — Menos de cinco dias:** **Given** a fonte retorna três dias,
  **Then** três entradas são exibidas, sem estado de erro; **Given** a fonte
  retorna zero dias, **Then** o estado de erro de AC-03.4 é exibido.

### RF-04 — Alternar unidade de temperatura (US-04)

- **AC-04.1 — Unidade inicial:** **Given** o usuário não alterou a unidade,
  **When** clima e previsão são exibidos, **Then** todas as temperaturas
  terminam com "°C" e o botão "Celsius" tem `aria-pressed="true"`.
- **AC-04.2 — Conversão para Fahrenheit:** **Given** temperaturas em °C,
  **When** o usuário aciona "Fahrenheit", **Then** todas as temperaturas
  visíveis no clima atual e na previsão terminam com "°F" e seguem
  `°F = °C × 9/5 + 32` (20 → 68; 0 → 32; −10 → 14).
- **AC-04.3 — Retorno para Celsius:** **Given** temperaturas em °F, **When** o
  usuário aciona "Celsius", **Then** todas voltam a "°C" com os valores
  originais (68 → 20).
- **AC-04.4 — Controle acessível:** **Given** a aplicação está carregada,
  **When** o usuário navega até o controle só com o teclado, **Then** encontra
  dois botões com nomes acessíveis "Celsius" e "Fahrenheit", acionáveis por
  Enter ou Espaço; o ativo expõe `aria-pressed="true"` e o outro
  `aria-pressed="false"`.
- **AC-04.5 — Arredondamento e sinal:** **Given** a fonte retorna 21,3 °C,
  **Then** o texto em Celsius é "21°C" e em Fahrenheit (70,34) é "70°F";
  **Given** 21,5 °C, **Then** "22°C"; **Given** −3,7 °C, **Then** "-4°C";
  **Given** −0,4 °C, **Then** "0°C" (nunca "-0°C").
- **AC-04.6 — Não persiste entre recarregamentos:** **Given** o usuário
  alternou para Fahrenheit, **When** recarrega a página e seleciona uma cidade,
  **Then** as temperaturas estão em Celsius.
- **AC-04.7 — Alternância é local:** **Given** clima e previsão carregados,
  **When** o usuário alterna a unidade, **Then** nenhuma requisição de rede é
  disparada.

### RF-05 — Fonte de dados (restrição)

- **AC-05.1 — Geocodificação via Open-Meteo:** **When** a aplicação busca
  cidades, **Then** a requisição é dirigida a `geocoding-api.open-meteo.com`.
- **AC-05.2 — Previsão via Open-Meteo:** **When** a aplicação consulta o tempo,
  **Then** a requisição é dirigida a `api.open-meteo.com` e inclui `latitude`,
  `longitude` e `timezone=auto`.
- **AC-05.3 — Sem API key:** **Given** qualquer requisição à Open-Meteo,
  **Then** não contém parâmetro de query nem cabeçalho destinados a
  autenticação (`apikey`, `key`, `token`, `Authorization`), e a interface não
  tem campo para chave.
- **AC-05.4 — Atribuição:** **Given** qualquer tela, **Then** um rodapé exibe
  o texto de atribuição à Open-Meteo (Apêndice C) com link para
  `https://open-meteo.com/`.

### RF-06 — Sem conta e sem persistência (restrição)

- **AC-06.1 — Uso sem autenticação:** **Given** primeira visita, sem cookies ou
  armazenamento, **When** o usuário busca, seleciona e consulta clima e
  previsão, **Then** nenhuma etapa exibe login, cadastro ou redirecionamento.
- **AC-06.2 — Allowlist de rede:** **Given** o usuário realizou buscas,
  seleções e alternâncias, **When** todas as requisições da sessão são
  inspecionadas, **Then** todos os destinos são a própria origem da aplicação
  (incluindo recursos de desenvolvimento, como HMR) ou hosts terminados em
  `.open-meteo.com`.
- **AC-06.3 — Sem armazenamento local:** **Given** o mesmo cenário, **When**
  `localStorage`, `sessionStorage` e `document.cookie` são inspecionados,
  **Then** nenhum item foi gravado pela aplicação.

## Non-Functional Requirements

- **RNF-01 — Responsividade.** Larguras a partir de 320 px sem rolagem
  horizontal. *Verificação:* em 320, 768 e 1280 px,
  `documentElement.scrollWidth <= clientWidth` em todas as telas, e o fluxo
  E2E completo executa em cada viewport.
- **RNF-02 — Clareza das unidades.** *Verificação:* AC-02.1, AC-03.2, AC-04.1
  a AC-04.3.
- **RNF-03 — Compreensibilidade.** Cidade e período identificáveis.
  *Verificação:* AC-01.2, AC-02.1, AC-03.1, AC-03.10.
- **RNF-04 — Desempenho.** Indicador de carregamento em ≤ 100 ms após envio ou
  seleção (verificável em E2E com resposta atrasada). Metas de rede — página
  inicial ≤ 2,5 s e resultados ≤ 2 s no p75 — pendem de validação e condições
  de medição (Open Questions).
- **RNF-05 — Acessibilidade.** WCAG 2.2 AA. *Verificação:* axe-core no
  Playwright com zero violações `serious`/`critical` em quatro telas (inicial,
  resultados, clima carregado, erro); fluxo completo só por teclado; AC-01.14 e
  AC-04.4.
- **RNF-06 — Disponibilidade.** Aplicação estática; a disponibilidade efetiva
  é a da hospedagem e da Open-Meteo. Meta de 99,5 % mensal pende de validação.
- **RNF-07 — Resiliência e atualidade.** *Verificação:* AC-00.1, AC-02.3,
  AC-02.8, AC-03.4, AC-03.5.
- **RNF-08 — Compatibilidade.** *Default proposto, a confirmar:* últimas duas
  versões estáveis de Chrome, Firefox, Safari e Edge, desktop e mobile.
  *Verificação:* suíte E2E executada nos projetos Playwright correspondentes.
- **RNF-09 — Idioma.** pt-BR. *Verificação:* `<html lang="pt-BR">`; título do
  documento em pt-BR; datas e números via `Intl` com locale `pt-BR`; strings
  centralizadas em um único módulo (Apêndice C).
- **RNF-10 — Segurança.** Nenhum dado externo é interpretado como HTML
  (AC-01.9). No release, `pnpm audit` sem vulnerabilidades `critical` ou
  `high` sem mitigação registrada.

## Edge Cases

Regra geral: **degradar por bloco, nunca a tela inteira**.

| Edge case | Comportamento | Critérios |
|---|---|---|
| Cidade inexistente | Estado vazio ecoando o termo; campo mantém o termo; sem erro. | AC-01.3, AC-01.7 |
| Termo vazio ou < 2 caracteres | Nenhuma requisição; nenhum estado; foco no campo. | AC-01.6 |
| Caracteres especiais e acentos | URL-encoded sem normalização; renderização como texto literal. | AC-01.8, AC-01.9 |
| Falha de API (4xx/429/5xx, offline, CORS) | Erro só no bloco afetado; cidade ativa preservada; botão "Tentar novamente". | AC-00.1, AC-01.5, AC-01.13, AC-02.3, AC-03.4, AC-03.5 |
| Timeout (10 s) | Cancelamento; mesmo estado de erro da falha de API. | AC-01.11, AC-02.4, AC-03.6 |
| Geocoding sem resultados (`{}` ou `results: []`) | Estado vazio, sem erro de processamento. | AC-01.10 |
| Resposta parcial | Temperatura ausente → erro do bloco; `weather_code` ausente → condição omitida; arrays desalinhados → erro; > 5 dias → corta; 1–4 dias → exibe os disponíveis; 0 dias → erro. | AC-02.5, AC-02.6, AC-03.7, AC-03.8, AC-03.11 |
| Respostas fora de ordem | Só a última requisição prevalece. | AC-01.12, AC-02.7 |
| Homônimos | Nome + `admin1` + país; campos ausentes omitidos. | AC-01.1, AC-01.15, AC-01.16 |
| Fuso horário | `timezone=auto`; datas e horas civis sem deslocamento pelo fuso do navegador. | AC-02.8, AC-03.9, AC-05.2 |
| Código WMO desconhecido | "Condição desconhecida". | AC-02.9 |
| Mais de 10 resultados | A fonte é consultada com `count=10`; a lista nunca excede 10. | AC-01.18 |

## Assumptions

- O usuário seleciona uma cidade antes de ver clima e previsão.
- O plano gratuito da Open-Meteo (sem chave) atende ao volume esperado da v1;
  limites excedidos aparecem como HTTP 429 e seguem o fluxo de erro.
- "Dispositivos móveis" significa interface responsiva no navegador, não app
  nativo.
- Todo estado vive em memória durante a sessão.

## Risks

| Tipo | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| Produto | Seleção da cidade homônima errada. | Média | Alto | Definida: AC-01.1, AC-01.15, AC-01.16. |
| Produto | Conteúdo mínimo insuficiente para parte dos usuários. | Média | Médio | Validar com usuários; extras candidatos a v2 (Out of Scope). |
| Produto | Usuário espera que a unidade persista entre visitas. | Média | Baixo | Decisão v1 explícita (AC-04.6); reavaliar em v2. |
| Técnico | Indisponibilidade, 429 ou mudança na Open-Meteo. | Média | Alto | Estados de erro por bloco, retry manual, timeout; contrato de dados no Apêndice A facilita adaptação. |
| Técnico | Dado antigo lido como atual. | Baixa | Alto | AC-02.8 exibe horário da fonte. Sem atualização automática na v1 (Out of Scope). |
| Técnico | Latência em redes móveis. | Média | Médio | Indicador ≤ 100 ms; medir metas após definir condições. |
| Técnico | Falha de layout/teclado em dispositivos relevantes. | Média | Alto | RNF-01, RNF-05, RNF-08 com verificação automatizada. |
| Técnico | Data deslocada por conversão UTC implícita. | Média | Alto | AC-03.9, AC-02.8 com testes em dois fusos. |
| Processo | Escopo crescer durante Plan/Code. | Média | Alto | Out of Scope explícito; mudanças exigem revisão desta spec. |

## Out of Scope (v1)

**Funcionalidades**

- Autenticação, contas, histórico de buscas, favoritos.
- Várias cidades simultâneas ou comparação lado a lado.
- Previsão horária; dados além do mínimo (vento, umidade, sensação térmica,
  índice UV, nascer e pôr do sol, probabilidade de chuva, alertas, mapas).
- Geolocalização do dispositivo; busca por CEP, código postal ou coordenadas.
- Sugestões enquanto digita (autocomplete); fechamento da lista por Esc.
- Ícones ou ilustrações de condição do tempo — a condição é apenas texto.
- Compartilhamento, URL com a cidade selecionada (deep link) ou roteamento;
  a URL não muda durante o uso.
- Título do documento dinâmico (permanece "Previsão do Tempo").
- Outros idiomas além de pt-BR; infraestrutura de i18n.

**Comportamento**

- Persistência no servidor ou no navegador (inclui unidade escolhida).
- Atualização automática (polling), revalidação ao voltar à aba, retry
  automático com backoff. Dados só são recarregados ao selecionar uma cidade
  (inclusive a mesma) ou acionar "Tentar novamente".
- Cache de respostas entre seleções.
- Animações e transições além do padrão do navegador; suporte explícito a
  `prefers-reduced-motion`.

**Plataforma e operação**

- App nativo, PWA, service worker, uso offline.
- Analytics, telemetria, rastreamento de uso.
- Monitoramento de erros em serviço externo (ex.: Sentry) — incompatível com
  a allowlist de AC-06.2.
- Backend próprio, proxy ou BFF.

## Open Questions

Nenhuma pendência bloqueia Plan, Code ou Test. Itens restantes:

- **Fase Ship:** alvo de hospedagem estática (ex.: GitHub Pages, Vercel,
  Netlify) e domínio. Não afeta código nem testes.
- Condições de rede e método de medição para as metas de RNF-04 e RNF-06.
- Confirmação da matriz de navegadores de RNF-08.
- Validação das personas e das métricas pessoais com usuários reais; métrica de
  sucesso do produto (ex.: consultas concluídas, retorno).
- Quais extras de dados entram na v2 e se a alternância de unidade se aplica à
  sensação térmica.
- Textos do Apêndice C podem ser revisados por UX writing sem impacto nos
  testes, exceto os literais citados em critérios.

## Apêndice A — Contrato de dados consumido (Open-Meteo)

Define o que os testes simulam e o que a aplicação depende. Campos não listados
são ignorados; campos listados ausentes seguem Edge Cases.

**Geocodificação** — `GET https://geocoding-api.open-meteo.com/v1/search`

| Parâmetro | Valor |
|---|---|
| `name` | termo (URL-encoded) |
| `count` | `10` |
| `language` | `pt` |
| `format` | `json` |

Resposta: `results?: Array<{ id: number; name: string; latitude: number;
longitude: number; country?: string; admin1?: string }>`. Ausência de `results`
equivale a lista vazia.

**Previsão** — `GET https://api.open-meteo.com/v1/forecast`

| Parâmetro | Valor |
|---|---|
| `latitude`, `longitude` | do resultado selecionado |
| `current` | `temperature_2m,weather_code` |
| `daily` | `temperature_2m_max,temperature_2m_min,weather_code` |
| `timezone` | `auto` |
| `forecast_days` | `5` |

Resposta: `current: { time: string /* "YYYY-MM-DDTHH:mm" local */;
temperature_2m: number /* °C */; weather_code: number }`,
`daily: { time: string[] /* "YYYY-MM-DD" */; temperature_2m_max: number[];
temperature_2m_min: number[]; weather_code: number[] }`. Temperaturas chegam em
°C; a conversão para °F é local (AC-04.7).

## Apêndice B — `weather_code` (WMO) → texto pt-BR

| Códigos | Texto |
|---|---|
| 0 | Céu limpo |
| 1 | Predominantemente limpo |
| 2 | Parcialmente nublado |
| 3 | Nublado |
| 45, 48 | Nevoeiro |
| 51 / 53 / 55 | Garoa fraca / Garoa moderada / Garoa intensa |
| 56 / 57 | Garoa congelante fraca / Garoa congelante intensa |
| 61 / 63 / 65 | Chuva fraca / Chuva moderada / Chuva forte |
| 66 / 67 | Chuva congelante fraca / Chuva congelante forte |
| 71 / 73 / 75 | Neve fraca / Neve moderada / Neve forte |
| 77 | Grãos de neve |
| 80 / 81 / 82 | Pancadas de chuva fracas / moderadas / violentas |
| 85 / 86 | Pancadas de neve fracas / fortes |
| 95 | Tempestade |
| 96 / 99 | Tempestade com granizo fraco / Tempestade com granizo forte |
| outros | Condição desconhecida |

## Apêndice C — Textos da interface (v1)

Textos marcados com ★ são citados literalmente em critérios de aceite.
`{termo}` e `{hora}` são interpolações.

| Chave | Texto |
|---|---|
| `app.title` | Previsão do Tempo |
| `search.label` | Cidade |
| `search.placeholder` | Digite o nome de uma cidade |
| `search.button` | ★ Buscar |
| `search.loading` | Buscando cidades… |
| `search.empty` | Nenhuma cidade encontrada para “{termo}”. |
| `search.error` | Não foi possível buscar cidades. |
| `action.retry` | ★ Tentar novamente |
| `current.title` | ★ Clima atual |
| `current.loading` | Carregando clima atual… |
| `current.error` | Não foi possível carregar o clima atual. |
| `current.updatedAt` | Atualizado às {hora} |
| `forecast.title` | ★ Previsão |
| `forecast.loading` | Carregando previsão… |
| `forecast.error` | Não foi possível carregar a previsão. |
| `forecast.today` | ★ Hoje |
| `unit.celsius` | ★ Celsius |
| `unit.fahrenheit` | ★ Fahrenheit |
| `weather.unknown` | ★ Condição desconhecida |
| `attribution` | Dados meteorológicos: Open-Meteo.com (CC BY 4.0) |

## Apêndice D — Matriz de rastreabilidade

Indexada por história para orientar a quebra de tarefas (uma história ≈ um
grupo de tarefas) e a cobertura de testes. Um critério pode servir a mais de
uma história; critérios de falha estão concentrados em US-05 para que a
resiliência seja uma fatia de trabalho própria. Níveis de teste: **U** =
unitário (Vitest, funções puras), **C** = componente (Testing Library com
fonte simulada), **E** = E2E (Playwright com interceptação de rede).

| História | RF | Critérios de aceite | RNFs | Nível predominante |
|---|---|---|---|---|
| **US-01** — Cláudia busca e seleciona uma cidade | RF-01 | AC-01.0, AC-01.1, AC-01.2, AC-01.3, AC-01.4a, AC-01.4b, AC-01.6, AC-01.7, AC-01.8, AC-01.9, AC-01.10, AC-01.12, AC-01.14, AC-01.17, AC-01.18, AC-01.19, AC-01.20 | RNF-01, RNF-03, RNF-05, RNF-09, RNF-10 | C para estados e render; E para AC-01.8, AC-01.12, AC-01.14, AC-01.17 |
| **US-06** — Rafael distingue cidades homônimas | RF-01 | AC-01.1, AC-01.15, AC-01.16 | RNF-03 | C; E para AC-01.15 (lat/long na requisição) |
| **US-02** — Marina vê o clima atual | RF-02 | AC-02.1, AC-02.2, AC-02.6, AC-02.7, AC-02.8, AC-02.9 | RNF-02, RNF-03, RNF-07, RNF-09 | U para AC-02.8 (formatação de hora) e AC-02.9 (mapa WMO); C para o restante |
| **US-03** — Rafael vê a previsão de 5 dias | RF-03 | AC-03.1, AC-03.2, AC-03.3, AC-03.8, AC-03.9, AC-03.10, AC-03.11 | RNF-01, RNF-03, RNF-09 | U para AC-03.9 e AC-03.10 (datas, dois fusos); C para o restante |
| **US-04** — Marina alterna °C/°F | RF-04 | AC-04.1, AC-04.2, AC-04.3, AC-04.4, AC-04.5, AC-04.6, AC-04.7 | RNF-02, RNF-05 | U para AC-04.2, AC-04.3, AC-04.5 (conversão/arredondamento); C para AC-04.1, AC-04.4; E para AC-04.6 (reload) e AC-04.7 (rede) |
| **US-05** — Marina entende falhas e tenta de novo | RF-01, RF-02, RF-03 | AC-00.1, AC-00.2, AC-01.5, AC-01.11, AC-01.13, AC-02.3, AC-02.4, AC-02.5, AC-03.4, AC-03.5, AC-03.6, AC-03.7, AC-03.11 (ramo 0 dias) | RNF-04, RNF-07 | C com fonte simulada (erro HTTP, corpo parcial, timers falsos para timeout); E para AC-00.1 |
| **Restrição RF-05** — fonte Open-Meteo e atribuição | RF-05 | AC-05.1, AC-05.2, AC-05.3, AC-05.4 | — | E (inspeção de requisições); C para AC-05.4 |
| **Restrição RF-06** — sem conta, sem persistência | RF-06 | AC-06.1, AC-06.2, AC-06.3 | RNF-10 | E (allowlist de hosts, storage, cookies) |

**RNFs sem história própria** (verificados transversalmente, ver cada RNF):
RNF-01 responsividade em 3 viewports · RNF-05 axe-core em 4 telas + fluxo só
por teclado · RNF-08 projetos Playwright por navegador · RNF-09 `lang`, título,
`Intl` · RNF-10 `pnpm audit`.

**Ordem sugerida de implementação** (dependências): US-01 → US-06 → US-02 →
US-03 → US-04 → US-05 → restrições RF-05/RF-06 (parte já emerge das anteriores;
AC-05.4 e AC-06.x fecham ao final). RNF-01/05/08/09 entram no checklist de
release.

## Critérios de release (v1)

- Todos os critérios de aceite têm ao menos um teste automatizado (Vitest ou
  Playwright) passando.
- `pnpm lint`, `pnpm build` e `pnpm test` sem erros.
- axe-core sem violações `serious`/`critical` nas quatro telas de RNF-05.
- `pnpm audit` sem `critical`/`high` sem mitigação registrada.
- Atribuição à Open-Meteo visível (AC-05.4).
- Nenhum item de Out of Scope implementado sem revisão desta spec.
