# Discovery — Aplicação de Previsão do Tempo

## Contexto

A empresa solicitou uma aplicação para que usuários consultem informações
meteorológicas de cidades de seu interesse. O produto deve permitir buscar
cidades, consultar o clima atual e visualizar a previsão dos próximos cinco
dias. A experiência também deve oferecer alternância entre Celsius e
Fahrenheit e funcionar em dispositivos móveis.

O briefing define as capacidades principais, mas ainda não detalha regras de
busca, conteúdo meteorológico, atualização dos dados ou preferências do
usuário. Esses pontos estão registrados como perguntas em aberto para orientar
a especificação do produto.

## Personas

Personas hipotéticas, a validar com usuários reais. As métricas representam
sucesso do ponto de vista de cada persona, não metas de negócio.

| Persona | Objetivo principal | Contexto de uso | Métrica de sucesso pessoal |
|---|---|---|---|
| **Marina — planeja o dia:** consulta o tempo antes de sair de casa. | Saber rapidamente as condições atuais e a previsão para decidir o que vestir ou levar. | Principalmente mobile, pela manhã ou antes de sair. | Encontra a cidade e entende o clima atual e a previsão em menos de um minuto, sem precisar consultar outra fonte. |
| **Rafael — viajante frequente:** consulta o tempo no destino antes e durante uma viagem. | Verificar as condições de outra cidade e planejar atividades considerando a previsão local. | Mobile durante a viagem; desktop ao planejar com antecedência. | Consegue selecionar a localidade correta e identificar a previsão dos próximos cinco dias no fuso e nas unidades que espera. |
| **Cláudia — consulta várias localidades, uma de cada vez:** consulta o tempo da própria cidade e de lugares onde familiares vivem. | Verificar rapidamente as condições de diferentes cidades em consultas sucessivas. | Mais frequentemente desktop; também mobile em consultas rápidas. | Consegue localizar cada cidade desejada (inclusive homônimas) e entender suas condições sem confundir localidade, data ou unidade. |

**Validação de escopo:** acompanhar várias cidades simultaneamente foi definido
como **fora do escopo da v1** (ver Decisões). A persona Cláudia foi ajustada
para consultas sequenciais; se a validação com usuários indicar necessidade de
comparação lado a lado, o recurso será candidato para v2.

## Decisões

- **Fonte de dados — Open-Meteo, sem API key:** usar Open-Meteo para obter
  geocodificação e dados meteorológicos. A escolha evita exigir uma chave de API
  para o produto e resolve a indefinição sobre a fonte de dados.
- **Período da previsão — hoje + 4 dias:** apresentar o dia atual e os quatro
  dias seguintes. A decisão define o significado de “cinco dias” e resolve se
  o dia atual faz parte do período.
- **Unidade padrão — Celsius (°C):** iniciar as consultas com temperaturas em
  Celsius, mantendo a possibilidade de alternar para Fahrenheit. A decisão
  define a unidade inicial.
- **Autenticação e persistência — sem autenticação e sem persistência de
  servidor:** permitir o uso sem conta e não armazenar dados de usuário no
  servidor. A decisão resolve se é necessário criar contas ou manter dados
  persistentes no backend.
- **Idioma da interface — português do Brasil (pt-BR):** apresentar a UI em
  pt-BR. A decisão resolve a indefinição sobre o idioma inicial do produto.

### Decisões complementares (revisão da spec, 27/10/2026)

Fechadas para destravar o plano técnico; detalhes e critérios de aceite em
`weather-app-spec.md`.

- **Conteúdo mínimo:** clima atual = temperatura + condição + horário da última
  atualização; previsão diária = data + máxima + mínima + condição. A condição
  é texto pt-BR derivado do `weather_code` (WMO), com fallback para código
  desconhecido. Extras (vento, umidade, sensação térmica) ficam para v2.
- **Desambiguação de homônimos:** cada resultado exibe nome, região
  administrativa (`admin1`) e país, quando a fonte os fornecer.
- **Fuso horário:** requisições usam `timezone=auto`; "hoje" e os horários são
  os da cidade consultada. Datas civis são exibidas sem deslocamento pelo fuso
  do navegador.
- **Arredondamento:** temperaturas exibidas como inteiro mais próximo.
- **Última atualização:** o horário de `current.time` é exibido no clima atual.
- **Persistência da unidade:** apenas durante a sessão; recarregar volta a °C.
  Nenhum armazenamento local (`localStorage`, `sessionStorage`, cookies) na v1.
- **Múltiplas cidades:** fora do escopo da v1; a aplicação mostra uma cidade
  por vez. Persona Cláudia ajustada para consultas sequenciais.

### Defaults de v1 (spec 1.0, 27/10/2026)

Fechados para tornar a spec pronta para produção. Podem ser revistos sem
reabrir o discovery.

- **Modo de busca:** envio explícito (Enter ou botão); sem autocomplete.
  Termos com menos de 2 caracteres não disparam requisição. Até 10 resultados.
- **Estados de erro:** todo bloco em erro oferece "Tentar novamente" (retry
  manual). Timeout de 10 s por requisição. Sem retry automático nem polling.
- **Resposta parcial:** 1–4 dias de previsão são exibidos sem aviso; 0 dias é
  erro.
- **Datas:** primeira entrada "Hoje"; demais "qua., 28/10".
- **Alternância de unidade:** conversão local, sem nova requisição.
- **Textos da interface:** definidos no Apêndice C da spec; testes verificam
  papéis semânticos, não redação.
- **Atribuição:** rodapé com crédito à Open-Meteo (licença CC BY 4.0).
- **Previsão horária, geolocalização e dados extras:** fora da v1.

## Requisitos Funcionais

- **RF-01 — Buscar cidades:** permitir que o usuário pesquise uma cidade e
  selecione o local desejado para consultar as informações meteorológicas.
- **RF-02 — Consultar clima atual:** exibir as condições meteorológicas atuais
  da cidade selecionada.
- **RF-03 — Consultar previsão:** exibir a previsão do tempo para a cidade
  selecionada no período de hoje e dos quatro dias seguintes.
- **RF-04 — Alternar unidade de temperatura:** permitir alternar entre Celsius
  (°C) e Fahrenheit (°F) nas temperaturas apresentadas.

## Requisitos Não-Funcionais

- **RNF-01 — Responsividade:** a interface deve permitir o uso das
  funcionalidades principais em dispositivos móveis, tablets e desktops,
  adaptando-se a larguras a partir de 320 px sem rolagem horizontal.
- **RNF-02 — Clareza de unidade:** os valores de temperatura devem indicar
  claramente a unidade selecionada.
- **RNF-03 — Compreensibilidade:** a apresentação do clima atual e da previsão
  deve permitir que o usuário identifique a cidade e o período a que os dados
  correspondem.
- **RNF-04 — Desempenho:** em condições de rede a serem definidas, a página
  principal deve carregar em até 2,5 s no percentil 75, e a busca deve exibir
  resultados ou um estado de carregamento em até 2 s.
- **RNF-05 — Acessibilidade:** a aplicação deve atender ao nível AA da WCAG
  2.2, incluindo navegação por teclado, foco visível, contraste adequado e
  nomes acessíveis para controles.
- **RNF-06 — Disponibilidade:** a aplicação deve buscar uma disponibilidade
  mensal de 99,5%, excluindo janelas de manutenção previamente comunicadas.
- **RNF-07 — Resiliência e atualidade dos dados:** a interface deve informar
  quando os dados meteorológicos foram atualizados e apresentar um estado claro
  quando a fonte estiver indisponível, sem apresentar dados desatualizados como
  atuais.
- **RNF-08 — Compatibilidade:** a aplicação deve funcionar nos principais
  navegadores e sistemas operacionais móveis e desktop, cuja matriz de suporte
  deverá ser definida antes dos testes de aceitação.
- **RNF-09 — Idioma:** a interface deve ser apresentada em português do Brasil
  (pt-BR).

As metas quantitativas de desempenho e disponibilidade são propostas iniciais
para validação com as partes interessadas. O briefing não especifica as
condições de medição nem a matriz de navegadores e sistemas operacionais.

## Riscos

Probabilidades preliminares, a validar conforme as decisões de produto e a
arquitetura forem definidas.

| Tipo | Risco | Probabilidade | Impacto | Estratégia de mitigação |
|---|---|---|---|---|
| Produto | Cidades com nomes iguais ou semelhantes levam o usuário a selecionar a localidade errada. | Alta | Alto: exibição de dados para a cidade incorreta e perda de confiança no app. | Mostrar informações de desambiguação, como estado, país e região; exigir seleção explícita quando houver múltiplos resultados. |
| Produto | “Previsão de cinco dias” é interpretada de formas diferentes, especialmente quanto à inclusão do dia atual e ao fuso horário. | Média | Médio: previsão inconsistente com a expectativa do usuário. | Período definido como hoje + 4 dias; decidir o fuso que determina as datas e exibi-las claramente. |
| Produto | O conteúdo do “clima atual” e da previsão não atende às necessidades dos usuários ou apresenta informação em excesso. | Média | Médio: menor utilidade e dificuldade para encontrar informações relevantes. | Validar os dados prioritários com usuários ou stakeholders e definir um conjunto mínimo para a primeira versão. |
| Produto | A preferência Celsius/Fahrenheit é aplicada de modo inconsistente ou não é preservada como o usuário espera. | Média | Médio: leitura confusa ou incorreta das temperaturas. | Especificar unidade padrão, persistência da preferência e aplicação a todas as temperaturas; testar alternância no clima atual e na previsão. |
| Técnico | Indisponibilidade, limites ou mudanças na fonte meteorológica impedem consultas ou degradam o serviço. | Média | Alto: usuários não conseguem obter dados confiáveis. | Tratar erros e timeouts explicitamente, monitorar falhas, definir política de cache e comunicar indisponibilidade ou desatualização. |
| Técnico | Dados atrasados são apresentados como atuais, mesmo que a fonte retorne uma resposta válida. | Média | Alto: decisões do usuário podem se basear em informação enganosa. | Exibir horário da última atualização, definir tolerância máxima para idade dos dados e diferenciar dados em cache dos atuais. |
| Técnico | Latência da busca ou da previsão torna a experiência lenta, especialmente em redes móveis. | Média | Médio: abandono da consulta ou percepção de baixa qualidade. | Validar metas de performance, medir tempos reais, apresentar estados de carregamento e otimizar chamadas e payloads conforme medições. |
| Técnico | Layout ou interação falha em telas pequenas, navegadores ou dispositivos relevantes. | Média | Alto: funcionalidades centrais ficam difíceis ou impossíveis de usar. | Definir uma matriz de dispositivos e navegadores suportados; testar fluxos principais em tamanhos de tela e navegadores representativos. |
| Técnico | A aplicação não é acessível para pessoas que usam teclado, leitores de tela ou precisam de contraste adequado. | Média | Alto: parte dos usuários pode não conseguir buscar ou interpretar os dados. | Adotar critérios verificáveis de acessibilidade, como WCAG nível AA, e incluir testes com teclado, ferramentas automatizadas e revisão manual. |
| Produto e técnico | Requisitos ainda indefinidos geram aumento de escopo, retrabalho ou critérios de aceite incompatíveis. | Alta | Alto: atrasos, decisões inconsistentes e entrega diferente da necessidade do negócio. | Resolver perguntas prioritárias antes de fechar a especificação e registrar o escopo da primeira versão e o que fica fora dele. |
| Técnico | Conversão ou formatação incorreta de temperatura, datas ou horários causa informação enganosa. | Baixa | Alto: valores apresentados podem induzir o usuário ao erro. | Usar conversões e formatação consistentes, cobrir casos de teste de unidades e fusos horários e validar os valores contra os dados de origem. |

## Perguntas em Aberto

> Os temas **Cidades homônimas**, **Entrada da busca**, **Clima atual**,
> **Contagem dos dias**, **Fuso horário**, **Unidades**, **Persistência da
> unidade**, **Unidade padrão**, **Múltiplas cidades**, **Conta e histórico**,
> **Estados vazios e erros**, **Dados indisponíveis**, **Conteúdo
> meteorológico** (atribuição) e **Atualização e cache** foram resolvidos pelas
> Decisões acima. As linhas permanecem na tabela como registro do raciocínio
> de impacto.

| Tema | Pergunta em aberto | Impacto de seguir sem resposta |
|---|---|---|
| Público e necessidade | Quem são os usuários principais e em que situações consultarão o app? | Pode resultar em uma experiência que atende mal ao contexto real de uso, como consultas rápidas em deslocamento ou planejamento de viagens. |
| Objetivo e sucesso | Como será medido o sucesso do produto — por exemplo, consultas concluídas, retorno de usuários ou satisfação? | Sem métricas, será difícil avaliar se a solução atende à necessidade ou decidir quais melhorias priorizar. |
| Busca de cidades | A busca aceitará nomes de cidades apenas ou também país, estado, código postal e acentos? | A busca pode não encontrar locais esperados ou retornar resultados irrelevantes. |
| Cidades homônimas | Como o usuário distinguirá cidades com o mesmo nome? | O usuário pode selecionar a localidade errada e receber dados meteorológicos incorretos para sua intenção. |
| Entrada da busca | A busca será por envio de formulário, sugestões enquanto digita ou ambos? | O comportamento da interface e as expectativas de resposta ficam indefinidos. |
| Seleção de localidade | O app oferecerá localização atual do usuário? Se sim, será opcional e solicitará permissão? | A experiência pode divergir do esperado; solicitar localização sem decisão clara traz implicações de privacidade e permissões. |
| Clima atual | Quais informações compõem “o clima atual”: temperatura, sensação térmica, condição, umidade, vento ou outras? | O conteúdo pode ser insuficiente para os usuários ou incluir dados que não são prioritários. |
| Significado de “atual” | O que será considerado atual e qual atraso máximo dos dados é aceitável? | Informações antigas podem ser exibidas como atuais, reduzindo a confiança no produto. |
| Previsão de cinco dias | A previsão será diária, por hora ou terá os dois níveis de detalhe? | Afeta a utilidade da previsão, a quantidade de informação e a complexidade da interface. |
| Contagem dos dias | Os cinco dias incluem o dia de hoje ou começam amanhã? | Usuários podem receber uma quantidade ou período diferente do que esperam. A decisão registrada é hoje + 4 dias. |
| Fuso horário | Os dias e horários serão apresentados conforme o fuso da cidade consultada ou o do dispositivo? | Datas e horários podem parecer incorretos, especialmente ao consultar cidades em outros fusos. |
| Cobertura dos dados | Quais cidades e regiões precisam estar disponíveis? Há cobertura global ou limitada? | A busca pode prometer locais para os quais não há dados meteorológicos adequados. |
| Fonte dos dados | Qual será a fonte meteorológica e quais são suas limitações de cobertura, atualização e uso? | Podem surgir lacunas nos dados, custos, restrições de uso ou dependência de um serviço não validado. A fonte decidida é Open-Meteo; suas limitações ainda precisam ser avaliadas. |
| Atualização e cache | Com que frequência os dados serão atualizados e o usuário verá o horário da última atualização? | A aplicação pode fazer consultas em excesso ou apresentar dados desatualizados sem deixar isso claro. |
| Unidades | Celsius/Fahrenheit se aplica a todas as temperaturas exibidas, incluindo sensação térmica e previsão? | A unidade pode ser aplicada de forma inconsistente entre telas ou tipos de temperatura. |
| Persistência da unidade | A preferência de unidade será mantida apenas durante a sessão ou entre visitas? | A unidade pode voltar inesperadamente ao padrão, causando fricção ou leitura equivocada. |
| Unidade padrão | Qual será a unidade inicial, e ela dependerá da região, do idioma ou de outra preferência? | Usuários podem ver valores em uma unidade que não compreendem ou não esperam. A decisão registrada é Celsius. |
| Múltiplas cidades | O usuário poderá salvar, acompanhar ou alternar entre várias cidades? | A experiência pode exigir novas buscas a cada consulta, ou a equipe pode implementar gestão de cidades sem necessidade confirmada. Sem persistência de servidor, será necessário decidir se isso é viável localmente. |
| Conta e histórico | Será necessário criar conta? Pesquisas ou cidades recentes serão armazenadas? | A decisão afeta privacidade, persistência de dados e complexidade; armazenar sem necessidade pode coletar dados desnecessários. A decisão registrada é sem autenticação e sem persistência de servidor. |
| Estados vazios e erros | O que deve acontecer quando não há resultados, a cidade não está disponível ou a fonte falha? | O usuário pode ficar sem orientação ou interpretar uma falha como ausência de dados meteorológicos. |
| Dados indisponíveis | É aceitável mostrar dados em cache durante uma indisponibilidade? Como serão identificados? | Sem distinção clara, dados antigos podem ser confundidos com informações atuais. |
| Escopo de uso móvel | “Usar em dispositivos móveis” significa um site responsivo no navegador ou também um aplicativo nativo? | A equipe pode desenvolver para plataformas e canais diferentes do que o negócio pretende. |
| Dispositivos e telas | Quais tamanhos de tela, navegadores e sistemas operacionais precisam ser suportados? | A aplicação pode falhar em dispositivos relevantes ou o escopo de testes ficar indefinido. |
| Acessibilidade | Quais padrões e necessidades de acessibilidade devem ser atendidos? | Usuários com deficiência podem não conseguir buscar cidades ou compreender os dados; também fica difícil validar a conformidade. |
| Desempenho | Quais tempos máximos são aceitáveis para carregar a interface, buscar uma cidade e exibir a previsão? | Sem metas, uma experiência lenta pode ser considerada aceitável durante o desenvolvimento e descoberta tarde demais. |
| Disponibilidade | Qual nível de disponibilidade é esperado e há horários de manutenção aceitáveis? | Não será possível dimensionar ou avaliar a confiabilidade esperada para o serviço. |
| Idiomas e localização | Quais idiomas, formatos de data/hora e convenções regionais serão suportados? | Textos, unidades, datas e horários podem ser inadequados para parte dos usuários. A decisão registrada é UI em pt-BR; formatos regionais e eventual suporte multilíngue permanecem a esclarecer. |
| Conteúdo meteorológico | Há exigências de atribuição, avisos ou explicações sobre a origem e a interpretação dos dados? | A interface pode omitir contexto importante ou deixar de cumprir condições da fonte de dados. |
| Fora de escopo | O produto incluirá alertas meteorológicos, mapas, notícias ou recomendações, ou essas capacidades estão explicitamente fora do escopo? | Expectativas podem crescer durante o projeto e comprometer prazo e foco do produto. |
| Personas e priorização | As personas propostas representam usuários reais e qual delas deve orientar as prioridades da primeira versão? | O produto pode priorizar fluxos irrelevantes; a persona de acompanhamento de múltiplas cidades depende de capacidade ainda não confirmada. |
| Métricas das personas | As metas pessoais propostas, como encontrar informações em menos de um minuto, são adequadas e observáveis? | Métricas arbitrárias podem direcionar decisões de UX ou validações para critérios que os usuários não valorizam. |

## Suposições

- O usuário seleciona uma cidade antes de visualizar o clima e a previsão.
- A alternância entre Celsius e Fahrenheit aplica-se às temperaturas do clima
  atual e da previsão.
- A previsão é apresentada por dia; a inclusão de dados horários ainda precisa
  ser decidida.
- “Dispositivos móveis” implica uma interface responsiva, sem pressupor a
  existência de um aplicativo nativo.
- A seleção de cidade é necessária para consultar as informações
  meteorológicas.
