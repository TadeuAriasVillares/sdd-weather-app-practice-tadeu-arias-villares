## Contexto

Aplicação web de previsão do tempo voltada a consultas pessoais e rápidas. A
pessoa usuária busca uma cidade e visualiza as condições atuais e a previsão
para cinco dias, podendo alternar entre Celsius e Fahrenheit. A experiência
deve priorizar dispositivos móveis sem perder a utilidade em desktop.

## Requisitos Funcionais

- **RF1** - Buscar cidades por nome, apresentando sugestões que permitam
	desambiguar localidades com o mesmo nome.
- **RF2** - Exibir o clima atual da cidade selecionada: temperatura, condição,
	umidade, vento, pressão e precipitação.
- **RF3** - Exibir a previsão para cinco dias, com temperaturas mínima e máxima,
	condição, precipitação e vento de cada dia.
- **RF4** - Alternar entre Celsius e Fahrenheit, atualizando todos os valores
	exibidos sem nova consulta à API.
- **RF5** - Comunicar estados de carregamento, erro e ausência de resultados.

## Requisitos Não-Funcionais

- **RNF1 - Performance:** a carga inicial deve ocorrer em menos de dois segundos
	em uma conexão típica, e a busca deve apresentar resposta percebida como
	imediata.
- **RNF2 - Responsividade:** a interface deve ser mobile-first e funcional de
	320px de largura até telas desktop.
- **RNF3 - Acessibilidade:** a navegação deve funcionar por teclado, usar
	semântica e rótulos adequados e manter contraste básico compatível com WCAG AA.
- **RNF4 - Resiliência:** falhas de rede ou da API devem degradar a experiência
	de forma clara, sem quebrar a interface.
- **RNF5 - Implantação:** a fonte de dados não deve exigir chave de API, para
	viabilizar hospedagem estática simples.
- **RNF6 - Observabilidade para a pessoa usuária:** mensagens de erro devem
	indicar o problema e oferecer uma forma de tentar novamente.

## Riscos

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Limite de requisições ou indisponibilidade da API | Média | Alto | Tratar falhas, exibir mensagem clara e avaliar cache leve no cliente. |
| Cidades homônimas retornarem uma localização incorreta | Alta | Médio | Exibir país, estado ou região nas sugestões de geocoding. |
| Interface inconsistente em diferentes dispositivos | Média | Médio | Aplicar design mobile-first e testes E2E em viewport móvel e desktop. |
| Falha de conexão da pessoa usuária | Média | Médio | Exibir estado de erro com ação para tentar novamente. |
| Conversão incorreta entre unidades | Baixa | Alto | Centralizar a conversão em função pura e cobri-la com testes unitários. |

## Perguntas em Aberto (Open Questions)

1. Qual fonte de dados meteorológicos será usada? **Impacto:** define contrato,
	 custo, limites de uso e necessidade de chave de API.
2. A previsão de cinco dias inclui o dia atual? **Impacto:** define a regra de
	 seleção das datas e os rótulos da interface.
3. Deve haver geolocalização automática? **Impacto:** adiciona permissões do
	 navegador, estados de recusa e complexidade de UX.
4. Qual unidade deve ser exibida na primeira visita? **Impacto:** afeta a
	 renderização inicial e a expectativa da pessoa usuária.
5. O produto precisa funcionar offline ou armazenar dados em cache? **Impacto:**
	 aumenta a complexidade de armazenamento e invalidação de dados.
6. Quais idiomas a interface deve suportar? **Impacto:** determina a necessidade
	 de internacionalização e formatação de datas.
7. O produto deve manter histórico ou cidades favoritas? **Impacto:** exige
	 persistência local ou de servidor e mudanças no escopo da v1.
8. Como a busca de cidades deve funcionar: ao digitar ou por botão, qual o
	 mínimo de caracteres, o limite de sugestões e sua ordenação? **Impacto:**
	 define a interação, o volume de requisições e o critério de aceitação da
	 busca.
9. O que caracteriza o clima atual: horário e fuso da medição, fonte, frequência
	 de atualização e tratamento de dados desatualizados? **Impacto:** define a
	 confiança da informação exibida e como comunicar sua atualidade.
10. Qual resumo deve representar cada dia da previsão: condição predominante,
	 condição em horário específico ou pior condição do dia? **Impacto:** orienta
	 o mapeamento dos dados da API e a interpretação da previsão pela pessoa
	 usuária.
11. Quais unidades devem ser usadas para pressão, vento e precipitação e quais
	 delas devem mudar ao alternar entre Celsius e Fahrenheit? **Impacto:** evita
	 conversões inconsistentes e rótulos ambíguos na interface.
12. Quais critérios mensuráveis definem o suporte a dispositivos móveis: larguras
	 e orientações suportadas, navegadores-alvo e interações por toque? **Impacto:**
	 estabelece o escopo de responsividade e os cenários de teste.
13. Quais condições definem os SLAs de carga e busca: tipo de conexão, percentil,
	 região de referência e limite de timeout? **Impacto:** transforma metas vagas
	 de desempenho em requisitos verificáveis e orienta o tratamento de lentidão.

## Suposições (Assumptions)

- A pessoa usuária terá conexão com a internet na maior parte do tempo.
- O uso é individual e não exige autenticação.
- O público utiliza navegadores modernos com atualização automática.
- Os dados recebidos da API serão suficientes para compor as condições atuais e
	a previsão diária solicitadas.

## Decisões

- **Fonte de dados:** Open-Meteo, usando geocoding e forecast, sem chave de API.
- **Previsão de cinco dias:** hoje mais os quatro dias seguintes.
- **Unidade inicial:** Celsius.
- **Autenticação e persistência de servidor:** fora do escopo.
- **Idioma da interface:** pt-BR.
- **Geolocalização automática:** fora da v1; pode ser considerada futuramente.
- **Busca de cidades:** iniciar a busca após três
	caracteres, com debounce de 300 ms; exibir no máximo cinco sugestões, dando
	prioridade a correspondências exatas e por início do nome. A seleção ocorre
	pela lista de sugestões, não pelo envio de texto livre.
- **Clima atual:** exibir a leitura retornada pela
	Open-Meteo com data, hora e fuso da localidade. Os dados são consultados ao
	selecionar a cidade, sem atualização automática; se a leitura tiver mais de
	60 minutos, informar que pode estar desatualizada.
- **Resumo diário da previsão:** representar a
	condição prevista para 12h no fuso local da cidade, junto das temperaturas
	mínima e máxima do dia. Essa regra oferece uma referência consistente para
	comparar os cinco dias.
- **Unidades não térmicas:** manter pressão em hPa,
	vento em km/h e precipitação em mm. A alternância entre Celsius e Fahrenheit
	converte somente temperaturas.
- **Suporte mobile:** suportar larguras de 320 px a
	1440 px ou mais, em orientação retrato e paisagem, nos dois lançamentos mais
	recentes de Chrome, Firefox, Edge e Safari. Controles por toque devem ter área
	mínima de 44 por 44 px.
- **SLAs de desempenho:** para pessoas usuárias no
	Brasil em conexão 4G, a carga inicial deve concluir em até dois segundos no
	percentil 75; sugestões de busca devem aparecer em até um segundo no
	percentil 75. Requisições à API terão timeout de oito segundos, seguido de
	uma mensagem de erro com ação para tentar novamente.

## Personas

- **Viajante planejador:** consulta a previsão de cinco dias para organizar uma
	viagem ou a agenda da semana, em desktop ou celular.
- **Decisor do dia a dia:** verifica rapidamente as condições atuais para
	escolher roupa ou decidir levar guarda-chuva, principalmente no celular.
- **Pessoa em deslocamento:** compara a previsão de outra cidade antes de sair,
	valorizando busca rápida e resultados sem ambiguidade.
