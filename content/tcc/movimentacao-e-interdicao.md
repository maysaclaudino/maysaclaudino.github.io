+++
date = '2025-07-08T18:50:30-03:00'
draft = false
title = 'Movimentação e Interdição de Vias'
image = '/images/map-network.jpg'
+++

<!-- - tentar gerar visualização
- tese de mestrado 4.3 kanashiro
  - como fechar uma via
  - como mudar o trajeto
  - codigo no github

--- -->

Para simular o impacto de eventos pluviométricos na mobilidade, é fundamental modelar tanto o deslocamento padrão dos veículos quanto as alterações dinâmicas na malha viária, como a interdição de ruas. Esta seção detalha os mecanismos implementados no **InterSCsimulator** para essas duas funcionalidades.

## Modelo de Movimentação de Veículos

A malha viária da cidade é representada como um dígrafo, onde as vias são as arestas e os cruzamentos são os vértices. O movimento dos veículos se baseia no cálculo de rotas e na atualização contínua de sua posição e velocidade.

### Carros

O deslocamento de um carro entre um ponto de origem e um de destino segue as seguintes etapas:

1.  **Cálculo de Rota:** A rota é definida como uma lista de vértices que representa o menor caminho no dígrafo entre a origem e o destino.
2.  **Ocupação da Via:** Ao entrar em uma nova aresta (via), um contador que registra o número de veículos presentes nela é incrementado.
3.  **Cálculo de Velocidade:** A velocidade do veículo é calculada dinamicamente com base na ocupação da via. A fórmula considera o limite de velocidade da via, sua capacidade máxima e o número de veículos que nela trafegam.
    * **Fluxo Livre:** Com a ocupação de até **30%** da capacidade, os carros se movem na velocidade máxima permitida.
    * **Fluxo denso:** Entre **30% e 100%** da capacidade, a velocidade é progressivamente reduzida.
    * **Congestionamento:** Ao atingir a capacidade máxima, a velocidade é drasticamente reduzida para um valor fixo (0,8), simulando uma condição de congestionamento.
4.  **Agendamento:** O simulador calcula o tempo que o veículo levará para percorrer a via e agenda sua próxima ação para o final desse período.
5.  **Liberação da Via:** Ao sair da via, o contador de ocupação é decrementado, e a velocidade dos veículos restantes é recalculada.

### Ônibus

A lógica de movimentação para os ônibus é semelhante à dos carros, mas com ajustes que modelam o comportamento do transporte público:

* **Cálculo de Rota:** A rota não é calculada apenas entre a origem e o destino final, mas sim como uma sequência de menores caminhos entre cada um dos pontos de parada da linha.
* **Peso de Ocupação:** Ao contabilizar a ocupação da via, um ônibus possui peso 3, representando um impacto maior no tráfego em comparação com um carro.
* **Paradas:** Quando um vértice da rota corresponde a um ponto de ônibus, o simulador executa o gerenciamento de embarque e desembarque de passageiros.

## Implementação de Interdição e Alteração de Vias

A simulação de alagamentos exige que a malha viária possa ser alterada dinamicamente. Essa funcionalidade foi implementada na dissertação de mestrado de Lucas Kanashiro, que introduz um gerenciador de eventos e de grafos.

Diferente da versão original do simulador, esta implementação conta com um gerenciador de grafo. Este agente é executado em um laço infinito, permitindo que outros processos enviem mensagens para adicionar ou remover vértices e arestas do dígrafo a qualquer momento.

Um novo agente, o gerenciador de eventos, é responsável por ler um arquivo de entrada (`events.xml`) que descreve as alterações a serem aplicadas na malha viária durante a simulação.

Existem dois tipos principais de eventos:

### Fechamento Total de Via

Esse evento instrui o gerenciador de grafo a remover a aresta correspondente à via interditada. O ID da aresta removida é inserido em uma tabela ETS com o status `remove`. As tabelas ETS (Erlang Term Storage) são um recurso da linguagem Erlang que funciona como um banco de dados em memória, permitindo consultas e alterações, o que é ideal para gerenciar o estado dinâmico das vias em tempo de simulação.

Antes de acessar uma nova via, cada carro consulta essa tabela. Caso a próxima via de sua rota esteja marcada como removida, o carro recalcula seu trajeto a partir de sua posição atual, utilizando a versão mais recente do grafo.

### Redução de Capacidade da Via

Este evento não remove a via, mas sim diminui sua capacidade, o que afeta diretamente o cálculo de velocidade dos veículos que passam por ela.

Para ambos os casos, o gerenciador de eventos agenda uma ação futura para reverter a alteração, seja reinserindo a aresta no grafo ou restaurando a capacidade original da via.
