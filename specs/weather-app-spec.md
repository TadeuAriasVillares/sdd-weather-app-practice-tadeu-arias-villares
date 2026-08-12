# Especificação de Produto - Weather App

## Overview

Aplicação web de previsão do tempo para consultas pessoais rápidas. A pessoa
usuária busca uma cidade, consulta as condições atuais e a previsão de hoje mais
quatro dias e pode alternar a temperatura entre Celsius e Fahrenheit.

O produto prioriza uso em dispositivos móveis, sem perder a utilidade em
desktop. Os dados são fornecidos pela Open-Meteo, sem autenticação e sem chave
de API.

## Functional Requirements

- **RF1 - Busca de cidade:** iniciar a busca após três caracteres, com debounce
  de 300 ms. Exibir no máximo cinco sugestões com cidade, país e região quando
  disponível, priorizando correspondências exatas e por início do nome. Somente
  a resposta da busca mais recente é exibida; respostas de buscas anteriores são
  descartadas. A pessoa usuária deve selecionar uma sugestão para consultar o
  clima.
- **RF2 - Clima atual:** após selecionar uma cidade, exibir temperatura,
  condição, umidade, velocidade do vento, pressão, precipitação, data, hora e
  fuso local da leitura. Temperaturas são exibidas arredondadas para o inteiro
  mais próximo. Informar quando os dados tiverem mais de 60 minutos.
- **RF3 - Previsão de cinco dias:** exibir hoje e os quatro dias seguintes. Para
  cada dia, exibir temperaturas mínima e máxima, condição prevista para 12h no
  fuso local, precipitação e velocidade do vento.
- **RF4 - Unidades:** iniciar em Celsius e permitir alternar temperaturas entre
  Celsius e Fahrenheit sem nova consulta. Pressão permanece em hPa, vento em
  km/h e precipitação em mm.
- **RF5 - Estados da interface:** exibir um estado inicial que oriente a busca
  enquanto nenhuma cidade foi selecionada; comunicar carregamento, ausência de
  resultados e falhas de geocoding ou previsão. Todo erro deve oferecer uma ação
  para tentar novamente.

## User Stories

- **US1** - Como decisor do dia a dia, quero buscar minha cidade para verificar
  rapidamente as condições atuais antes de sair de casa.
- **US2** - Como viajante planejador, quero consultar a previsão dos próximos
  cinco dias para organizar uma viagem ou a agenda da semana.
- **US3** - Como pessoa em deslocamento, quero distinguir cidades com o mesmo
  nome para consultar a localização correta antes de iniciar o trajeto.
- **US4** - Como decisor do dia a dia, quero alternar entre Celsius e Fahrenheit
  para compreender a temperatura na unidade que prefiro.
- **US5** - Como decisor do dia a dia, quero receber mensagens claras e poder
  tentar novamente quando uma consulta falhar por instabilidade de rede.
- **US6** - Como decisor do dia a dia, quero interagir com a aplicação no celular
  em retrato ou paisagem para consultar o clima com conforto onde estiver.

## Acceptance Criteria

### RF1 - Busca de cidade (US1, US3)

- Dado que o campo contém menos de três caracteres, quando a pessoa usuária
  digita, então nenhuma busca é iniciada.
- Dado que o campo contém três ou mais caracteres, quando passam 300 ms sem nova
  digitação, então a busca é iniciada.
- Dado que existem resultados, quando a busca conclui, então são exibidas no
  máximo cinco sugestões com cidade, país e região quando disponível, ordenadas
  com correspondências exatas e por início do nome antes das demais.
- Dado que sugestões estão visíveis, quando o texto é reduzido para menos de
  três caracteres, então as sugestões exibidas são ocultadas.
- Dado que uma nova busca é iniciada antes da resposta da busca anterior chegar,
  quando a resposta antiga chega depois da mais recente, então ela é descartada
  e não substitui as sugestões atuais.
- Dado que a pessoa usuária seleciona uma sugestão, quando a seleção é concluída,
  então o clima da localização selecionada é consultado.

### RF2 - Clima atual (US1)

- Dada uma cidade selecionada, quando a consulta de clima atual conclui, então
  são exibidos temperatura, condição, umidade, vento, pressão, precipitação,
  data, hora e fuso local.
- Dado que a leitura retornada tem mais de 60 minutos, quando os dados são
  exibidos, então a interface informa que podem estar desatualizados.

### RF3 - Previsão de cinco dias (US2)

- Dada uma cidade selecionada, quando a previsão conclui, então são exibidos
  exatamente cinco dias: hoje e os quatro dias seguintes no fuso da cidade.
- Dado um dia da previsão, quando ele é exibido, então apresenta temperaturas
  mínima e máxima, condição de 12h, precipitação e vento.

### RF4 - Unidades (US4)

- Dado que as temperaturas estão em Celsius, quando a pessoa usuária seleciona
  Fahrenheit, então todas as temperaturas atual e previstas são exibidas em
  Fahrenheit sem nova consulta à fonte de dados.
- Dado que a unidade foi alterada, quando os dados são exibidos, então pressão,
  vento e precipitação permanecem respectivamente em hPa, km/h e mm.

### RF5 - Estados da interface (US1, US2, US3, US5)

- Dado o acesso à aplicação, quando nenhuma cidade foi buscada ou selecionada,
  então é exibido um estado inicial orientando a buscar uma cidade.
- Dado que uma consulta está em andamento, quando a interface é exibida, então
  há um indicador de carregamento.
- Dado que a busca não encontra cidades, quando a consulta conclui, então é
  exibido um estado vazio informativo.
- Dada uma falha de rede, de geocoding ou de previsão, quando a requisição falha
  ou excede oito segundos, então é exibida uma mensagem clara com a ação "Tentar
  novamente", sem bloquear a interface.

### Responsividade (US6)

- Dado um viewport de 320 px em orientação retrato, quando a aplicação é
  exibida, então todas as funcionalidades ficam acessíveis sem rolagem
  horizontal.
- Dado um dispositivo móvel, quando a orientação muda entre retrato e paisagem,
  então o conteúdo se reorganiza sem perda de informação ou de estado.

## Non-Functional Requirements

- **Performance:** para pessoas usuárias no Brasil usando conexão 4G, a carga
  inicial deve concluir em até dois segundos no percentil 75 e as sugestões de
  busca devem aparecer em até um segundo no percentil 75.
- **Responsividade:** a interface deve funcionar de 320 px a 1440 px ou mais,
  nas orientações retrato e paisagem.
- **Compatibilidade:** suportar os dois lançamentos mais recentes de Chrome,
  Firefox, Edge e Safari.
- **Acessibilidade:** permitir navegação por teclado, usar semântica e rótulos
  adequados, foco visível, contraste compatível com WCAG AA e controles por toque
  com área mínima de 44 por 44 px.
- **Resiliência:** requisições devem expirar em oito segundos e apresentar um
  estado recuperável; a ausência de um campo de resposta não pode quebrar a
  interface.
- **Observabilidade para a pessoa usuária:** mensagens de erro devem indicar o
  tipo de falha (rede, serviço indisponível ou tempo excedido) e oferecer a ação
  "Tentar novamente". O estado vazio de busca sem resultados é informativo e não
  é tratado como erro.
- **Implantação:** a aplicação deve poder ser hospedada estaticamente e não pode
  depender de chave de API.

## Edge Cases

| Caso | Comportamento esperado |
| --- | --- |
| Primeiro acesso, sem cidade buscada | Exibir estado inicial orientando a busca. |
| Recarga da página | Retornar ao estado inicial; a última cidade não é lembrada na v1. |
| Campo de busca vazio ou com menos de três caracteres | Não iniciar busca nem exibir sugestões. |
| Texto reduzido para menos de três caracteres após exibir sugestões | Ocultar as sugestões exibidas anteriormente. |
| Respostas de busca fora de ordem (digitação rápida) | Descartar respostas antigas e manter apenas o resultado da busca mais recente. |
| Cidade inexistente ou geocoding sem resultados | Exibir estado vazio informativo. |
| Cidades homônimas | Exibir país e região para permitir seleção da localização correta. |
| Busca com acentos ou caracteres especiais | Preservar o termo e exibir os resultados retornados pela fonte de dados. |
| Falha da API de geocoding ou previsão | Exibir mensagem clara e ação para tentar novamente. |
| Timeout após oito segundos | Encerrar o carregamento, exibir erro recuperável e manter a interface utilizável. |
| Resposta parcial com campo ausente | Exibir "-" para o campo indisponível sem quebrar o layout. |
| Leitura atual com mais de 60 minutos | Exibir os dados e sinalizar que podem estar desatualizados. |

## Assumptions

- A pessoa usuária terá conexão com a internet na maior parte do tempo.
- O uso é individual e não requer login.
- O público utiliza navegadores modernos com atualização automática.
- A Open-Meteo fornece os dados necessários para as condições atuais e a
  previsão diária definida nesta especificação.

## Risks

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Indisponibilidade ou limite de requisições da API | Média | Alto | Comunicar falhas, permitir nova tentativa e avaliar cache leve no cliente. |
| Seleção incorreta de cidade homônima | Alta | Médio | Exibir país e região em todas as sugestões. |
| Inconsistência entre dispositivos | Média | Médio | Adotar abordagem mobile-first e validar os viewports e orientações definidos. |
| Falha de conexão da pessoa usuária | Média | Médio | Aplicar timeout, estado de erro recuperável e nova tentativa. |
| Conversão de temperatura incorreta | Baixa | Alto | Definir critérios de aceite para todas as temperaturas atual e previstas. |

## Out of Scope

- Autenticação, contas de usuário e persistência em servidor.
- Favoritos, histórico de pesquisas e cidades salvas.
- Lembrar a última cidade consultada entre sessões.
- Geolocalização automática.
- Funcionamento offline e cache persistente.
- Idiomas além de pt-BR.
- Notificações push, alertas meteorológicos e previsões além de cinco dias.

## Open Questions

- Qual critério de desempate deve ser usado quando várias sugestões têm
  correspondência exata ou começam com o termo buscado (por exemplo, população
  ou ordem alfabética)? **Impacto:** define a ordenação final das sugestões
  quando o critério atual não é suficiente para desempatar. **Comportamento
  interino (não bloqueante):** manter a ordem retornada pela fonte de dados.
- Fora este ponto, não há bloqueantes para a v1. As demais escolhas de produto
  relevantes foram fechadas no discovery.