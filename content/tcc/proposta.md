+++
date = '2025-04-19T16:17:36-03:00'
draft = false
title = 'Proposta de trabalho'
image = '/images/aerial-view-cityscape.jpg'
+++


# Simulação do impacto de eventos pluviométricos na mobilidade urbana da cidade de São Paulo

[Abrir a proposta em PDF](/tcc/proposta.pdf)

## Motivação

De acordo com o Centro Nacional de Monitoramento e Alertas de Desastres Naturais, alagamentos são caracterizados pelo excesso na capacidade do escoamento do sistema de drenagem urbana. Essa condição é geralmente causada em função de precipitações intensas. Esses alagamentos ocasionam obstruções nas vias, levando à necessidade de alteração nos trajetos habituais da população, ou mesmo impossibilitando o deslocamento até o destino final.

Segundo Marengo (2020), as chuvas extremas têm aumentado sua frequência no decorrer dos anos, portanto os alagamentos serão cada vez mais presentes e severos na rotina urbana. Este trabalho visa possibilitar a simulação e a análise de cenários de precipitações e alagamentos na cidade de São Paulo além dos seus impactos na mobilidade urbana.

## Objetivos

Este trabalho tem como objetivo estender o software InterSCsimulator para que ele incorpore informações pluviométricas e de ocorrência de alagamentos. Dessa forma, busca-se estudar o impacto de eventos climáticos na mobilidade da cidade de São Paulo.

A simulação deve permitir a modelagem de diferentes volumes de precipitação e os consequentes impactos sobre o tráfego urbano, alguns deles sendo: quais vias são afetadas por alagamentos, como os veículos (carros, ônibus e bicicletas) alteram sua rota padrão considerando as diferentes limitações de cada um, dentre outras possibilidades.

Com base na simulação pretende-se extrair métricas relevantes relacionadas ao fenômeno e analisar o impacto causado na mobilidade geral, e a diferença dele em locais que foram ou não afetados pela chuva. Algumas dessas métricas podem ser:
alteração no tempo médio da viagem,
número de linhas de ônibus que passam pela área afetada e passageiros a bordo,
acessibilidade da população a serviços de primeira necessidade tais como mercados, farmácias, hospitais e instituições de ensino.

Dado que São Paulo possui ampla extensão territorial, simular alagamentos em toda a cidade pode ser computacionalmente custoso, portanto, é provável que apenas parte dela seja simulada. A área será escolhida com base na análise de dados históricos existentes.

## Metodologia

O InterSCsimulator é um simulador de tráfego para cidades inteligentes desenvolvido na linguagem Erlang, capaz de simular agentes urbanos tais como carros, pessoas e ônibus deslocando-se sobre a malha viária urbana. Será utilizada a rede viária da cidade São Paulo obtida por meio do projeto OpenStreetMap. As viagens da simulação serão baseadas na Pesquisa Origem Destino 2023 com mais de 35 milhões de viagens que representam os padrões de mobilidade da população metropolitana.

Para atender os objetivos apresentados acima, os dados das vias serão enriquecidos com informações que correlacionam os casos de alagamentos com índices pluviométricos. Os pontos de alagamento de 2013 a 2024 foram extraídos a partir dos registros da Companhia de Engenharia de Trânsito e da Defesa Civil. O histórico de índices pluvioméricos do período está disponível na base de dados das estações pluviométricas do Instituto Nacional de Meteorologia e do Centro de Gerenciamento de Emergências Climáticas.

Abaixo consta um cronograma das atividades planejadas, com a indicação do período estimado para sua conclusão.

| Atividade                                                       | mar | abr | mai | jun | jul | ago | set | out | nov | dez |
|------------------------------------------------------------------|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Estudo da linguagem Erlang                                       |  x  |  x  |     |     |     |     |     |     |     |     |
| Estudo do código-fonte do InterSCsimulator                       |  x  |  x  |  x  |     |     |     |     |     |     |     |
| Levantamento de dados históricos pluviométricos e de alagamentos|     |  x  |  x  |     |     |     |     |     |     |     |
| Integração dos dados ao simulador                                |     |     |     |  x  |  x  |     |     |     |     |     |
| Adaptar a simulação para considerar eventos pluviométricos       |     |     |     |     |  x  |  x  |  x  |     |     |     |
| Análise dos impactos dos alagamentos na mobilidade urbana        |     |     |     |     |     |     |     |  x  |  x  |  x  |
| Escrita da monografia                                            |     |     |     |     |     |  x  |  x  |  x  |  x  |  x  |


## Referências

CARVALHO, Raissa Barreto de. Análise de chuvas extremas e a relação com eventos de alagamentos na cidade de São Paulo – SP. 2021. 151 f. Dissertação (Mestrado em Desastres Naturais) – Instituto de Ciência e Tecnologia, Universidade Estadual Paulista (Unesp), São José dos Campos, 2021. Disponível em: http://hdl.handle.net/11449/217041. Acesso em: 18 abr. 2025.

CEMADEN – Centro Nacional de Monitoramento e Alertas de Desastres Naturais. Inundação. Disponível em: http://www2.cemaden.gov.br/inundacao/. Acesso em: 14 abr. 2025.

COMPANHIA DO METROPOLITANO DE SÃO PAULO – METRÔ. Pesquisa Origem e Destino 2023. São Paulo: Metrô, 2025. Disponível em: https://www.metro.sp.gov.br/pt_BR/pesquisa-od/​. Acesso em: 14 abr. 2025.

INSTITUTO NACIONAL DE METEOROLOGIA (INMET). Banco de Dados Meteorológicos para Ensino e Pesquisa – BDMEP. Brasília: INMET, [s.d.]. Disponível em: https://bdmep.inmet.gov.br/​. Acesso em: 18 abr. 2025.

INTERSCITY. InterSCSimulator – A Smart City Simulator. Disponível em: https://interscity.org/software/interscsimulator/. Acesso em: 10 mar. 2025.

MARENGO, J. A. et al. Changing trends in rainfall extremes in the Metropolitan Area of São Paulo: causes and impacts. Frontiers in Climate, v. 2, p. 3, 2020.

PREFEITURA DO MUNICÍPIO DE SÃO PAULO. Alagamento (dados de ocorrências). São Paulo: GeoSampa, 2024. Disponível em: https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869​. Acesso em: 8 abr. 2025.


