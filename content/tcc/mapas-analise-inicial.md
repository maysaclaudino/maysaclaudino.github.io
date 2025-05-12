+++
date = '2025-05-05T19:13:56-03:00'
draft = false
title = 'Análise inicial dos dados e visualização em mapas'
image = '/images/mapa-sp.jpg'
+++

# Análise preliminar dos dados de alagamentos e chuvas

Nesta etapa inicial do projeto, foram construídos alguns mapas interativos com base nos dados de alagamentos e precipitação no município de São Paulo, referentes ao período de 2018 a 2023. O objetivo é identificar padrões espaciais e temporais que possam fundamentar as decisões na etapa de simulação da mobilidade urbana sob eventos de chuva intensa.

Para facilitar a análise, consolidei em um único arquivo CSV os dados de [alagamentos](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/intranet/por/catalog.search#/metadata/432c06b1-03a1-4b1f-9210-423e1b58e869) registrados pela [Defesa Civil](https://www.defesacivil.sp.gov.br/), os registros de ruas intransitáveis fornecidos pela [Companhia de Engenharia de Tráfego](https://www.cetsp.com.br/), e os [índices pluviométricos](https://arquivos.saisp.br/nextcloud/index.php/s/qikdinFyAM33MJK?path=%2FBOLETIM_PLUVIOMETRICO) correspondentes à data de cada ocorrência, obtidos do [Centro de Gerenciamento de Emergências Climáticas](https://www.cgesp.org/v3/).

O conjunto abrange o período de 2018 a 2023. Optei por não incluir os dados de 2024, pois estavam disponíveis apenas até o início de novembro e poderiam comprometer a comparação anual.

## Ocorrências de alagamentos e ruas intransitáveis (2018–2023)

<iframe src="/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2018-2023.html" width="100%" height="500" style="border:none;"></iframe>

Para uma análise mais detalhada por ano, acesse os mapas individuais:

- [Ocorrências 2023](/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2023.html)
- [Ocorrências 2022](/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2022.html)
- [Ocorrências 2021](/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2021.html)
- [Ocorrências 2020](/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2020.html)
- [Ocorrências 2019](/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2019.html)
- [Ocorrências 2018](/tcc/01-mapas-analise-inicial/mapa_ocorrencias_2018.html)

A análise dos mapas indica que as subprefeituras com maior número de ocorrências são Sé e São Miguel. No entanto, essas regiões apresentam características diferentes:

- Em São Miguel, observa-se alto número de alagamentos, mas poucas ocorrências de vias intransitáveis. Além disso, os registros estão bastante concentrados geograficamente.
- Já na Sé, as ocorrências são mais diversificadas e distribuídas pela região central.

Dado o volume, a distribuição dos dados e a diversidade de meios de transporte (ônibus, metrô, ruas e avenidas), a região da Sé se mostra mais interessante para o desenvolvimento da simulação. Outro fator relevante é a presença de corpos hídricos como os rios Tamanduateí e Tietê, que podem influenciar nos eventos de alagamento.

## Volume de precipitação no dia de cada ocorrência

<iframe src="/tcc/01-mapas-analise-inicial/mapa_chuva_ocorrencias.html" width="100%" height="500" style="border:none;"></iframe>

## Ocorrências sem chuva significativa

Outro ponto que me chamou atenção são os registros de ocorrências em dias com baixa ou nenhuma precipitação (filtrando os dados com "< 5 mm" no mapa de chuvas). Um estudo do porque isso acontece pode ser relevante para implementar o simulador, algumas ideias são:

- A possibilidade de alagamentos residuais causados por chuvas anteriores devido a qualidade da infraestrutura de drenagem.
- A influência da topografia, que pode favorecer o acúmulo de água com mais facilidade.
- Inconsistências nos registros: os dados de precipitação são contabilizados das 00h às 23h59 de cada dia, será que existem casos em que a chuva ocorreu à noite, mas o alagamento foi registrado apenas no dia seguinte?
