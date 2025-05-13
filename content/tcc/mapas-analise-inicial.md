+++
date = '2025-05-05T19:13:56-03:00'
draft = false
title = 'Análise inicial dos dados e visualização em mapas'
image = '/images/mapa-sp.jpg'
+++

# Análise preliminar dos dados de alagamentos e chuvas

Nesta etapa inicial do projeto, foram construídos alguns mapas interativos com base nos dados de alagamentos e precipitação no município de São Paulo, referentes ao período de 2018 a 2023. O objetivo é identificar padrões espaciais que possam fundamentar as decisões na etapa de simulação da mobilidade urbana sob eventos de chuva intensa.

Para facilitar a análise, consolidei em um único arquivo CSV os dados de [alagamentos](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/intranet/por/catalog.search#/metadata/432c06b1-03a1-4b1f-9210-423e1b58e869) registrados pela [Defesa Civil](https://www.defesacivil.sp.gov.br/), os registros de ruas intransitáveis fornecidos pela [Companhia de Engenharia de Tráfego](https://www.cetsp.com.br/), e os [índices pluviométricos](https://arquivos.saisp.br/nextcloud/index.php/s/qikdinFyAM33MJK?path=%2FBOLETIM_PLUVIOMETRICO) correspondentes à data de cada ocorrência, obtidos do [Centro de Gerenciamento de Emergências Climáticas](https://www.cgesp.org/v3/).

O conjunto abrange o período de 2018 a 2023. Optei por não incluir ano de 2024, pois os dados da CET deste ano estavam disponíveis apenas até o início de novembro e poderiam comprometer a comparação anual.

## Ocorrências de alagamentos e ruas intransitáveis (2018–2023)

<iframe id="mapa_ocorrencias" src="/tcc/01-mapas-analise-inicial/mapa_pontos_ocorrencias_2018-2023.html" width="100%" height="600" style="border:none;"></iframe>

<div class="tabs">
  <button onclick="trocarMapaOcorrecias('2018-2023')">2018-2023</button>
  <button onclick="trocarMapaOcorrecias('2018')">2018</button>
  <button onclick="trocarMapaOcorrecias('2019')">2019</button>
  <button onclick="trocarMapaOcorrecias('2020')">2020</button>
  <button onclick="trocarMapaOcorrecias('2021')">2021</button>
  <button onclick="trocarMapaOcorrecias('2022')">2022</button>
  <button onclick="trocarMapaOcorrecias('2023')">2023</button>
</div>

<script>
  function trocarMapaOcorrecias(ano) {
    const iframe = document.getElementById('mapa_ocorrencias');
    iframe.src = `/tcc/01-mapas-analise-inicial/mapa_pontos_ocorrencias_${ano}.html`;
  }
</script>

<style>
  .tabs {
    display: flex;          /* Exibe os botões em linha */
    gap: 10px;              /* Espaçamento entre as abas */
    margin-bottom: 20px;    /* Espaço abaixo das abas */
  }

  .tabs button {
    padding: 10px 20px;     /* Tamanho do botão */
    background-color:rgb(150, 150, 150); /* Cor de fundo */
    color: white;           /* Cor do texto */
    border: none;           /* Remove borda */
    border-radius: 5px;     /* Bordas arredondadas */
    cursor: pointer;       /* Mãozinha ao passar o mouse */
    font-size: 16px;        /* Tamanho da fonte */
  }

  .tabs button:hover {
    background-color: rgb(90, 90, 90); /* Cor ao passar o mouse */
  }

  .tabs button:focus {
    outline: none; /* Remove o contorno de foco */
  }
</style>


## Densidade de ocorrências por km² – Subprefeituras de SP (2018–2023)

<iframe id="mapa_densidade" src="/tcc/01-mapas-analise-inicial/mapa_subdistrito_km2_2018-2023.html" width="100%" height="600" style="border:none;"></iframe>

<div class="tabs">
  <button onclick="trocarMapaDensidade('2018-2023')">2018-2023</button>
  <button onclick="trocarMapaDensidade('2018')">2018</button>
  <button onclick="trocarMapaDensidade('2019')">2019</button>
  <button onclick="trocarMapaDensidade('2020')">2020</button>
  <button onclick="trocarMapaDensidade('2021')">2021</button>
  <button onclick="trocarMapaDensidade('2022')">2022</button>
  <button onclick="trocarMapaDensidade('2023')">2023</button>
</div>

<script>
  function trocarMapaDensidade(ano) {
    const iframe = document.getElementById('mapa_densidade');
    iframe.src = `/tcc/01-mapas-analise-inicial/mapa_subdistrito_km2_${ano}.html`;
  }
</script>

<style>
  .tabs {
    display: flex;          /* Exibe os botões em linha */
    gap: 10px;              /* Espaçamento entre as abas */
    margin-bottom: 20px;    /* Espaço abaixo das abas */
  }

  .tabs button {
    padding: 10px 20px;     /* Tamanho do botão */
    background-color:rgb(150, 150, 150); /* Cor de fundo */
    color: white;           /* Cor do texto */
    border: none;           /* Remove borda */
    border-radius: 5px;     /* Bordas arredondadas */
    cursor: pointer;       /* Mãozinha ao passar o mouse */
    font-size: 16px;        /* Tamanho da fonte */
  }

  .tabs button:hover {
    background-color: rgb(90, 90, 90); /* Cor ao passar o mouse */
  }

  .tabs button:focus {
    outline: none; /* Remove o contorno de foco */
  }
</style>


## Perfil das regiões mais afetadas

A análise dos mapas indica que as subprefeituras com maior número de ocorrências são Sé e São Miguel. No entanto, essas regiões apresentam características diferentes:

- Em São Miguel, observa-se alto número de alagamentos, mas poucas ocorrências de vias intransitáveis. Além disso, os registros estão bastante concentrados geograficamente.
- Já na Sé, as ocorrências são mais diversificadas e distribuídas pela região central.

Dado o volume, a distribuição dos dados e a diversidade de meios de transporte (ônibus, metrô, ruas e avenidas), a região da Sé se mostra mais interessante para o desenvolvimento da simulação. Outro fator relevante é a presença de corpos hídricos como os rios Tamanduateí e Tietê, que podem influenciar nos eventos de alagamento.

## Ocorrências sem chuva significativa

Outro ponto que me chamou atenção são os registros de ocorrências em dias com baixa ou nenhum a precipitação (filtrando os dados com "< 5 mm" no mapa de ocorrências). Um estudo do porque isso acontece pode ser relevante para implementar o simulador, algumas ideias são:

- A possibilidade de alagamentos residuais causados por chuvas anteriores devido a qualidade da infraestrutura de drenagem.
- A influência da topografia, que pode favorecer o acúmulo de água com mais facilidade.
- Inconsistências nos registros: os dados de precipitação são contabilizados das 00h às 23h59 de cada dia, será que existem casos em que a chuva ocorreu à noite, mas o alagamento foi registrado apenas no dia seguinte?
