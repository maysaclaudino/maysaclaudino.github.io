+++
draft = true
title = 'cursor'
+++

# Diferenças entre simuladores de rota
_Exported on 7/2/2025 at 17:16:06 GMT-3 from Cursor (1.0.0)_

---

**User**

me conte as principais diferenças do simulador na versao do francisco e do kanashiro. estou particulamente interessada em saber como é implementada o recalculo da rota de um veiculo se a via esta interditada

---

## Principais Diferenças entre os Simuladores Francisco e Kanashiro

### 1. **Sistema de Eventos e Interdição de Vias**

**Francisco:**
- **NÃO possui** sistema de eventos para interdição de vias
- As rotas são calculadas uma única vez no início da simulação
- Não há mecanismo para recálculo dinâmico de rotas

**Kanashiro:**
- **Possui sistema completo de eventos** (`class_EventsManager.erl`)
- Suporta três tipos de eventos:
  - `close_street`: Fecha uma via completamente
  - `open_street`: Reabre uma via previamente fechada
  - `reduce_capacity`: Reduz a capacidade de uma via
  - `restore_capacity`: Restaura a capacidade original

### 2. **Estrutura de Dados das Ruas**

**Francisco:**
```erlang
ets:insert(list_streets, {Vertices, Id, Length, Capacity, Freespeed, Count})
```

**Kanashiro:**
```erlang
ets:insert(list_streets, {Vertices, Id, Length, Capacity, Freespeed, Count, From, To, 0, 1, 4})
```
- Adiciona campos extras: `From`, `To`, contadores de carros por tick, e capacidade máxima

### 3. **Recálculo de Rotas**

**Francisco:**
- Usa `digraph:get_path()` para cálculo de rotas
- Rotas são calculadas apenas uma vez no início
- Não há verificação de vias interditadas durante a navegação

**Kanashiro:**
- Usa `digraph:get_best_path()` para cálculo de rotas
- **Implementa recálculo dinâmico** na função `get_next_vertex()`:

```erlang
get_next_vertex( State , Path , _Mode ) ->
    Origin = lists:nth( 1 , Path ),
    Vertices = list_to_atom( lists:concat( [ Origin  , lists:nth( 2 , Path ) ] )),
    
    % Verifica se a via está interditada
    PathChangedState = case ets:info(events) of
        undefined -> ok;
        _ ->
            case ets:lookup( events, Vertices ) of
                [{ _ , _ }] -> best_path( State, Path, Origin ); % Recalcula rota
                _ -> ok
            end
    end,
    
    % Verifica eventos de tráfego em tempo real
    IntermediateState = case PathChangedState of
        ok -> case ets:info(traffic_events) of
            undefined -> ok;
            _ ->
                case ets:lookup( traffic_events, Origin ) of
                    [{ _ , { FromNode, ToNode } }] ->
                        case edgeInPath( Path, FromNode, ToNode ) of
                            true -> best_path( State, Path, Origin ); % Recalcula rota
                            false -> ok
                        end;
                    _ -> ok
                end
            end;
        ChangedState -> ChangedState
    end,
    % ... continua com a navegação
```

### 4. **Gerenciamento de Grafos**

**Francisco:**
- Usa diretamente o grafo da cidade para cálculos de rota
- Não possui gerenciador de grafo separado

**Kanashiro:**
- Possui `graph_manager.erl` que gerencia o grafo dinamicamente
- Suporta operações de adicionar/remover arestas em tempo real:
  ```erlang
  delete_edge( G, IdV1, IdV2 ) ->
      [ { _, V1 } ] = ets:lookup( graph, atom_to_list( IdV1 ) ), 
      [ { _, V2 } ] = ets:lookup( graph, atom_to_list( IdV2 ) ), 
      { Edge, _, _, _ } = find_edge( G, V1, V2 ),
      digraph:del_edge( G, Edge ).
  ```

### 5. **Eventos em Tempo Real**

**Kanashiro:**
- Possui `events_handler.erl` que escuta eventos via RabbitMQ
- Processa eventos de tráfego em tempo real
- Atualiza a tabela `traffic_events` com informações dinâmicas

### 6. **Controle de Capacidade**

**Kanashiro:**
- Implementa controle de capacidade por via
- Veículos podem esperar quando a via está congestionada:
  ```erlang
  case NumCars >= MaxCar of
      true ->
          FinalState = setAttributes( State , [ { wait, true } ] ),
          executeOneway( FinalState , addSpontaneousTick , CurrentTick + 1 );
      false ->
          % Continua navegação
  end
  ```

### 7. **Integração com Sistemas Externos**

**Kanashiro:**
- Integração com RabbitMQ para eventos externos
- Publicação de dados de posição em tempo real
- Suporte a coordenadas geográficas (lat/long)

## Resumo da Implementação de Recálculo de Rotas

O **simulador do Kanashiro** implementa um sistema sofisticado de recálculo de rotas que:

1. **Monitora eventos de interdição** através do `EventsManager`
2. **Verifica vias interditadas** antes de cada movimento
3. **Recalcula rotas dinamicamente** usando `best_path()`
4. **Remove arestas do grafo** quando vias são fechadas
5. **Restaura arestas** quando vias são reabertas
6. **Processa eventos externos** via RabbitMQ

O **simulador do Francisco** é mais simples e estático, sem capacidade de adaptação a mudanças na rede viária durante a simulação.

Essa diferença fundamental torna o simulador do Kanashiro muito mais realista para simulações de cenários dinâmicos onde vias podem ser interditadas por acidentes, obras, ou outros eventos.

```erlang
ets:insert(list_streets, {Vertices, Id, Length, Capacity, Freespeed, Count})
```

```erlang
ets:insert(list_streets, {Vertices, Id, Length, Capacity, Freespeed, Count, From, To, 0, 1, 4})
```

```erlang
get_next_vertex( State , Path , _Mode ) ->
    Origin = lists:nth( 1 , Path ),
    Vertices = list_to_atom( lists:concat( [ Origin  , lists:nth( 2 , Path ) ] )),
    
    % Verifica se a via está interditada
    PathChangedState = case ets:info(events) of
        undefined -> ok;
        _ ->
            case ets:lookup( events, Vertices ) of
                [{ _ , _ }] -> best_path( State, Path, Origin ); % Recalcula rota
                _ -> ok
            end
    end,
    
    % Verifica eventos de tráfego em tempo real
    IntermediateState = case PathChangedState of
        ok -> case ets:info(traffic_events) of
            undefined -> ok;
            _ ->
                case ets:lookup( traffic_events, Origin ) of
                    [{ _ , { FromNode, ToNode } }] ->
                        case edgeInPath( Path, FromNode, ToNode ) of
                            true -> best_path( State, Path, Origin ); % Recalcula rota
                            false -> ok
                        end;
                    _ -> ok
                end
            end;
        ChangedState -> ChangedState
    end,
    % ... continua com a navegação
```

```erlang
  delete_edge( G, IdV1, IdV2 ) ->
      [ { _, V1 } ] = ets:lookup( graph, atom_to_list( IdV1 ) ), 
      [ { _, V2 } ] = ets:lookup( graph, atom_to_list( IdV2 ) ), 
      { Edge, _, _, _ } = find_edge( G, V1, V2 ),
      digraph:del_edge( G, Edge ).
```

```erlang
  case NumCars >= MaxCar of
      true ->
          FinalState = setAttributes( State , [ { wait, true } ] ),
          executeOneway( FinalState , addSpontaneousTick , CurrentTick + 1 );
      false ->
          % Continua navegação
  end
```

---

**User**

me explique como é gerenciado cada um dos eventos em EventsManager

---

**Cursor**

Com certeza! Analisando o arquivo `kanashiro/class_EventsManager.erl`, o gerenciamento de eventos acontece principalmente na função `iterate_events/2`.

O processo geral é o seguinte:

1.  O `EventsManager` é um ator que, a cada *tick* (unidade de tempo da simulação), verifica se há eventos agendados para aquele momento no dicionário `events`.
2.  Se houver eventos, a função `iterate_events/2` é chamada para processar cada um deles.

Vamos detalhar o que acontece para cada tipo de evento:

### 1. `"open_street"`

Este evento é responsável por abrir uma rua que estava previamente fechada.

```erlang
"open_street" ->
    io:format("OPEN STREET~n"),

    V1 = element( 2, Event ), % Vértice de origem da rua
    V2 = element( 3, Event ), % Vértice de destino da rua

    GraphManagerPid = getAttribute( State, pid ),

    % Envia uma mensagem para o graph_manager para adicionar a aresta de volta ao grafo
    GraphManagerPid ! { add_edge, V1, V2 },

    % Cria um ID para a aresta para poder removê-la da tabela de eventos
    EdgeID = list_to_atom( string:concat(atom_to_list(V1), atom_to_list(V2)) ),
    ets:delete( events, EdgeID ), % Remove o evento de interdição da tabela 'events'

    State;
```

**Resumo da operação:**
-   Obtém os vértices da rua.
-   Envia uma mensagem ao `graph_manager` para que ele adicione a aresta (a rua) de volta ao grafo da cidade, tornando-a roteável novamente.
-   Remove a si mesmo da tabela `events` para que a rua não seja considerada bloqueada em verificações futuras.

### 2. `"close_street"`

Este evento fecha uma rua por uma determinada duração.

```erlang
"close_street" ->
    io:format("CLOSE STREET~n"),

    V1 = element( 2, Event ),       % Vértice de origem
    V2 = element( 3, Event ),       % Vértice de destino
    Duration = element( 4, Event ), % Duração do fechamento

    GraphManagerPid = getAttribute( State, pid ),

    % Envia uma mensagem para o graph_manager remover a aresta do grafo
    GraphManagerPid ! { delete_edge, V1, V2 },

    % Cria um ID para a aresta e a insere na tabela 'events' com o status 'remove'
    % Isso é usado pelos carros para saber que a via está interditada
    EdgeID = list_to_atom( string:concat(atom_to_list(V1), atom_to_list(V2)) ),
    ets:insert( events, { EdgeID , remove }),

    % Cria um novo evento "open_street" para reabrir a rua no futuro
    OpenStreetEvent = { "open_street", V1, V2 },
    CurrentTickOffset = class_Actor:get_current_tick_offset( State ),
    EventsDict = getAttribute( State, events ),

    % Agenda o evento "open_street" para o tick atual + a duração do fechamento
    NewEvents = dict:append( CurrentTickOffset + Duration, OpenStreetEvent, EventsDict ),
    setAttribute( State, events, NewEvents );
```

**Resumo da operação:**
-   Obtém os vértices e a duração do fechamento.
-   Manda o `graph_manager` remover a aresta do grafo, o que impede que novas rotas a utilizem.
-   Insere um registro na tabela `events` para que os carros que já estão nessa rota possam verificar que a rua foi fechada e precisam recalcular seu caminho.
-   Agenda um evento futuro do tipo `"open_street"` para reabrir a rua assim que a `Duration` terminar.

### 3. `"reduce_capacity"`

Este evento reduz a capacidade de uma rua, tornando-a mais lenta, mas não a fecha completamente.

```erlang
"reduce_capacity" ->
    V1 = atom_to_list( element( 2, Event ) ),
    V2 = atom_to_list( element( 3, Event ) ),
    EdgeID = list_to_atom( string:concat(V1, V2) ),
    Duration = element( 4, Event ),
    CapacityFactor = element( 5, Event ), % Percentual da nova capacidade
    CAPACITY_INDEX = 4,

    % Busca os dados da rua na tabela 'list_streets'
    Street = lists:nth( 1, ets:lookup( list_streets, EdgeID )),
    Capacity = element( CAPACITY_INDEX, Street ),

    % Calcula a nova capacidade reduzida
    ReducedCapacity = Capacity * ( CapacityFactor / 100.0 ),
    ets:update_element( list_streets, EdgeID, [ { CAPACITY_INDEX, ReducedCapacity } ] ),

    % Agenda um evento "restore_capacity" para o futuro
    RestoreCapacityEvent = { "restore_capacity", EdgeID, CapacityFactor },
    CurrentTickOffset = class_Actor:get_current_tick_offset( State ),
    EventsDict = getAttribute( State, events ),
    NewEvents = dict:append( CurrentTickOffset + Duration, RestoreCapacityEvent, EventsDict ),
    setAttribute( State, events, NewEvents )
```

**Resumo da operação:**
-   Calcula a nova capacidade da rua com base no `CapacityFactor`.
-   Atualiza diretamente na tabela `list_streets` a capacidade da rua. Isso afetará os cálculos de velocidade e congestionamento para os carros que passarem por ela.
-   Agenda um evento `"restore_capacity"` para reverter a mudança após a `Duration`.

### 4. `"restore_capacity"`

Este evento restaura a capacidade original de uma rua que havia sido reduzida.

```erlang
"restore_capacity" ->
    EdgeID = element( 2, Event ),
    CapacityFactor = element( 3 , Event ),
    CAPACITY_INDEX = 4,
    Street = lists:nth( 1, ets:lookup( list_streets, EdgeID )),
    Capacity = element( CAPACITY_INDEX, Street ),

    % Restaura a capacidade original
    RestoredCapacity = Capacity * ( 100.0 / CapacityFactor ),
    ets:update_element( list_streets, EdgeID, [ { CAPACITY_INDEX, RestoredCapacity } ] ),
    State;
```

**Resumo da operação:**
-   Busca os dados da rua.
-   Calcula a capacidade original (invertendo o `CapacityFactor`).
-   Atualiza a capacidade da rua na tabela `list_streets`, retornando-a ao seu estado normal.

Este sistema de eventos permite que o simulador do Kanashiro modele um ambiente de tráfego muito mais dinâmico e realista em comparação com a versão do Francisco.

```erlang
"open_street" ->
    io:format("OPEN STREET~n"),

    V1 = element( 2, Event ), % Vértice de origem da rua
    V2 = element( 3, Event ), % Vértice de destino da rua

    GraphManagerPid = getAttribute( State, pid ),

    % Envia uma mensagem para o graph_manager para adicionar a aresta de volta ao grafo
    GraphManagerPid ! { add_edge, V1, V2 },

    % Cria um ID para a aresta para poder removê-la da tabela de eventos
    EdgeID = list_to_atom( string:concat(atom_to_list(V1), atom_to_list(V2)) ),
    ets:delete( events, EdgeID ), % Remove o evento de interdição da tabela 'events'

    State;
```

```erlang
"close_street" ->
    io:format("CLOSE STREET~n"),

    V1 = element( 2, Event ),       % Vértice de origem
    V2 = element( 3, Event ),       % Vértice de destino
    Duration = element( 4, Event ), % Duração do fechamento

    GraphManagerPid = getAttribute( State, pid ),

    % Envia uma mensagem para o graph_manager remover a aresta do grafo
    GraphManagerPid ! { delete_edge, V1, V2 },

    % Cria um ID para a aresta e a insere na tabela 'events' com o status 'remove'
    % Isso é usado pelos carros para saber que a via está interditada
    EdgeID = list_to_atom( string:concat(atom_to_list(V1), atom_to_list(V2)) ),
    ets:insert( events, { EdgeID , remove }),

    % Cria um novo evento "open_street" para reabrir a rua no futuro
    OpenStreetEvent = { "open_street", V1, V2 },
    CurrentTickOffset = class_Actor:get_current_tick_offset( State ),
    EventsDict = getAttribute( State, events ),

    % Agenda o evento "open_street" para o tick atual + a duração do fechamento
    NewEvents = dict:append( CurrentTickOffset + Duration, OpenStreetEvent, EventsDict ),
    setAttribute( State, events, NewEvents );
```

```erlang
"reduce_capacity" ->
    V1 = atom_to_list( element( 2, Event ) ),
    V2 = atom_to_list( element( 3, Event ) ),
    EdgeID = list_to_atom( string:concat(V1, V2) ),
    Duration = element( 4, Event ),
    CapacityFactor = element( 5, Event ), % Percentual da nova capacidade
    CAPACITY_INDEX = 4,

    % Busca os dados da rua na tabela 'list_streets'
    Street = lists:nth( 1, ets:lookup( list_streets, EdgeID )),
    Capacity = element( CAPACITY_INDEX, Street ),

    % Calcula a nova capacidade reduzida
    ReducedCapacity = Capacity * ( CapacityFactor / 100.0 ),
    ets:update_element( list_streets, EdgeID, [ { CAPACITY_INDEX, ReducedCapacity } ] ),

    % Agenda um evento "restore_capacity" para o futuro
    RestoreCapacityEvent = { "restore_capacity", EdgeID, CapacityFactor },
    CurrentTickOffset = class_Actor:get_current_tick_offset( State ),
    EventsDict = getAttribute( State, events ),
    NewEvents = dict:append( CurrentTickOffset + Duration, RestoreCapacityEvent, EventsDict ),
    setAttribute( State, events, NewEvents )
```

```erlang
"restore_capacity" ->
    EdgeID = element( 2, Event ),
    CapacityFactor = element( 3 , Event ),
    CAPACITY_INDEX = 4,
    Street = lists:nth( 1, ets:lookup( list_streets, EdgeID )),
    Capacity = element( CAPACITY_INDEX, Street ),

    % Restaura a capacidade original
    RestoredCapacity = Capacity * ( 100.0 / CapacityFactor ),
    ets:update_element( list_streets, EdgeID, [ { CAPACITY_INDEX, RestoredCapacity } ] ),
    State;
```

---

**User**

me explique o gerenciador do grafo

---

**Cursor**

Com certeza! O `graph_manager.erl` é um componente central na versão do Kanashiro para o gerenciamento dinâmico do mapa da cidade. Ele funciona como um servidor (um processo/ator) que mantém o estado do grafo de ruas e responde a requisições de outros atores, como `class_Car` e `class_EventsManager`.

Analisando o código de `kanashiro/graph_manager.erl`, podemos entender seu funcionamento:

### Inicialização (`init/0`)

```erlang
init() ->
    ets:new( graph, [public, set, named_table]),
    ets:insert( graph, { mypid, self() }),
    G = digraph:new(),
    ets:insert( graph, { mygraph, G }),
    loop(G).
```

1.  **Criação da Tabela ETS**: Ele cria uma tabela ETS chamada `graph`. Esta tabela é fundamental e serve como um "dicionário" para traduzir os IDs amigáveis das ruas e cruzamentos (ex: `'n1'`, `'n2'`) para os identificadores internos que o módulo `digraph` do Erlang usa.
2.  **Criação do Grafo**: Ele instancia um novo grafo direcionado (`digraph:new()`) para representar a malha viária.
3.  **Loop Principal**: Inicia o loop infinito (`loop/1`), passando o grafo recém-criado como seu estado. A partir daqui, o ator está pronto para receber e processar mensagens.

### Loop de Mensagens (`loop/1`)

Este é o coração do `graph_manager`. Ele espera por mensagens e age de acordo com o padrão de cada uma.

```erlang
loop(G) ->
    receive
        { add_vertex, Id }                 -> add_vertex(G, Id);
        { add_edge, IdV1, IdV2 }           -> add_edge(G, IdV1, IdV2);
        { delete_edge, IdV1, IdV2}         -> delete_edge(G, IdV1, IdV2);
        % ... outras mensagens ...
	    { get_best_path, IdV1, IdV2, PID } -> get_shortest_path( G, IdV1, IdV2, PID )
    end,
    loop(G).
```

Vamos ver as principais operações:

#### `add_vertex` e `add_edge`

-   Quando recebe uma mensagem `{add_vertex, Id}` ou `{add_edge, IdV1, IdV2}`, ele adiciona um vértice (cruzamento) ou uma aresta (rua) ao grafo.
-   A função `add_vertex/2` é interessante: ela cria um vértice no `digraph` e depois insere o mapeamento entre o ID amigável e o ID interno na tabela ETS `graph`.

#### `delete_edge`

```erlang
delete_edge( G, IdV1, IdV2 ) ->
    [ { _, V1 } ] = ets:lookup( graph, atom_to_list( IdV1 ) ),
    [ { _, V2 } ] = ets:lookup( graph, atom_to_list( IdV2 ) ),
    { Edge, _, _, _ } = find_edge( G, V1, V2 ),
    digraph:del_edge( G, Edge ).
```

-   Esta é a função chave para a interdição de vias.
-   Quando o `class_EventsManager` decide fechar uma rua, ele envia uma mensagem `{delete_edge, V1, V2}` para o `graph_manager`.
-   O `graph_manager` então remove a aresta correspondente do `digraph`.
-   **Consequência**: Qualquer chamada futura para calcular uma rota (`get_best_path`) não considerará mais esta rua, pois ela simplesmente "não existe" mais no grafo até que seja adicionada de volta.

#### `get_best_path` / `get_shortest_path`

```erlang
get_shortest_path( G, IdV1, IdV2, PID ) ->
    [ { _, V1 } ] = ets:lookup( graph, atom_to_list( IdV1 ) ),
    [ { _, V2 } ] = ets:lookup( graph, atom_to_list( IdV2 ) ),
    Path = digraph:get_short_path( G, V1, V2 ),
    Ids = get_vertices_ids( Path, [] ),
    PID ! { best_path, lists:reverse(Ids) }.
```

-   Um ator (como um carro) que precisa de uma rota envia uma mensagem `{get_best_path, Origem, Destino, self()}`.
-   O `graph_manager` usa `digraph:get_short_path/3` para encontrar o caminho mais curto entre os dois pontos no estado **atual** do grafo. Se uma rua foi removida por um evento de interdição, ela não fará parte do resultado.
-   A função auxiliar `get_vertices_ids/2` traduz a lista de vértices internos do `digraph` de volta para os IDs amigáveis (ex: `['n1', 'n5', 'n8']`) usando a tabela ETS `graph`.
-   Finalmente, ele envia o caminho calculado de volta para o processo que o solicitou (`PID ! {best_path, ...}`).

### Resumo

O `graph_manager` atua como uma **fonte única de verdade** para a topologia da malha viária. Ao centralizar as operações de manipulação do grafo (adicionar/remover arestas) e de cálculo de rotas, ele garante que todo o sistema opere com uma visão consistente e atualizada do mapa, permitindo que a simulação reaja dinamicamente a eventos como o fechamento de ruas.

```erlang
init() ->
    ets:new( graph, [public, set, named_table]),
    ets:insert( graph, { mypid, self() }),
    G = digraph:new(),
    ets:insert( graph, { mygraph, G }),
    loop(G).
```

```erlang
loop(G) ->
    receive
        { add_vertex, Id }                 -> add_vertex(G, Id);
        { add_edge, IdV1, IdV2 }           -> add_edge(G, IdV1, IdV2);
        { delete_edge, IdV1, IdV2}         -> delete_edge(G, IdV1, IdV2);
        % ... outras mensagens ...
	    { get_best_path, IdV1, IdV2, PID } -> get_shortest_path( G, IdV1, IdV2, PID )
    end,
    loop(G).
```

```erlang
delete_edge( G, IdV1, IdV2 ) ->
    [ { _, V1 } ] = ets:lookup( graph, atom_to_list( IdV1 ) ),
    [ { _, V2 } ] = ets:lookup( graph, atom_to_list( IdV2 ) ),
    { Edge, _, _, _ } = find_edge( G, V1, V2 ),
    digraph:del_edge( G, Edge ).
```

```erlang
get_shortest_path( G, IdV1, IdV2, PID ) ->
    [ { _, V1 } ] = ets:lookup( graph, atom_to_list( IdV1 ) ),
    [ { _, V2 } ] = ets:lookup( graph, atom_to_list( IdV2 ) ),
    Path = digraph:get_short_path( G, V1, V2 ),
    Ids = get_vertices_ids( Path, [] ),
    PID ! { best_path, lists:reverse(Ids) }.
```

---

**User**

me explique o events_handler

---

**Cursor**

Com certeza. O `events_handler.erl` é um componente fascinante no simulador Kanashiro, pois ele atua como uma ponte entre a simulação e o mundo exterior, permitindo que eventos de tráfego em tempo real afetem o comportamento dos veículos.

Analisando o código de `kanashiro/events_handler.erl`, podemos dividi-lo em duas partes principais:

### 1. Conexão com o RabbitMQ (`listen_for_events/0`)

O objetivo principal desta função é configurar e manter uma conexão com um servidor de mensagens RabbitMQ para "ouvir" por eventos.

```erlang
listen_for_events() ->
	setup(),
	timer:sleep(1000), // Espera um pouco para garantir que o resto da simulação subiu

	// Conecta-se ao RabbitMQ
	Hostname = os:getenv( "RABBITMQ_HOST", "localhost" ),
	{ok, Connection} = amqp_connection:start(#amqp_params_network{host=Hostname}),
	{ok, Channel} = amqp_connection:open_channel(Connection),

	// Define um "exchange" do tipo 'topic' chamado "traffic_sign"
	// Um exchange recebe mensagens de produtores e as empurra para as filas
	X = <<"traffic_sign">>,
	BindKey = <<"#">>, // O '#' é um curinga que significa "receba todas as mensagens"

	// Cria uma fila anônima e exclusiva para este handler
	#'queue.declare_ok'{queue = Q} = amqp_channel:call(Channel, #'queue.declare'{}),

	// Vincula a fila ao exchange
	QueueBind = #'queue.bind'{queue = Q,
				  exchange = X,
				  routing_key = BindKey},
	#'queue.bind_ok'{} = amqp_channel:call(Channel, QueueBind),

	// Começa a consumir mensagens da fila
	Sub = #'basic.consume'{queue = Q, no_ack = true},
	#'basic.consume_ok'{} = amqp_channel:call(Channel, Sub),

	loop(Channel). // Entra no loop para processar as mensagens recebidas
```

**Em resumo, esta parte:**
1.  Conecta-se a um servidor RabbitMQ (o endereço pode ser configurado por uma variável de ambiente).
2.  Inscreve-se no tópico `"traffic_sign"`.
3.  Fica esperando por qualquer mensagem que seja publicada nesse tópico.

Isso desacopla a simulação de quem gera os eventos. Qualquer sistema externo (um painel de controle de tráfego, um script, etc.) pode publicar uma mensagem no RabbitMQ, e o `events_handler` irá recebê-la.

### 2. Processamento de Mensagens (`loop/1`)

Esta é a função que lida com as mensagens quando elas chegam.

```erlang
loop(Channel) ->
	receive
		{#'basic.deliver'{ exchange=_Exchange, routing_key=_RoutingKey}, Content} ->
			#amqp_msg{payload = Payload} = Content,

			// A mensagem esperada é no formato "NodeAtual.NodeOrigem.NodeDestino"
			[ CurrentNode, FromNode, ToNode ] = string:tokens( binary_to_list(Payload), "." ),
			io:format("Current Node: ~p~nFrom Node: ~p~nTo Node: ~p~n", [CurrentNode, FromNode, ToNode]),

			// Insere o evento na tabela ETS 'traffic_events'
			ets:insert(traffic_events, { CurrentNode, { FromNode, ToNode } }),
			loop(Channel)
	end.
```

**O que acontece aqui:**
1.  O processo fica bloqueado no `receive` até que uma mensagem chegue da fila do RabbitMQ.
2.  Ele extrai o `Payload` (conteúdo) da mensagem.
3.  Ele assume que o conteúdo é uma string no formato `"NóAtual.NóDeOndeVim.NóParaOndeVou"`, e a divide usando `.` como delimitador. Por exemplo, uma mensagem `"n5.n1.n8"` indicaria um problema de trânsito no nó `n5` que afeta o trajeto vindo de `n1` para `n8`.
4.  **Este é o passo crucial:** Ele insere essa informação em uma tabela ETS chamada `traffic_events`.

### Qual o Impacto na Simulação?

O `events_handler` por si só não recalcula nenhuma rota. Sua única responsabilidade é receber eventos externos e armazená-los na tabela ETS `traffic_events`.

A "mágica" acontece no `class_Car`. Como vimos anteriormente, a função `get_next_vertex` de cada carro verifica esta tabela `traffic_events` a cada movimento. Se o carro está em um `CurrentNode` que tem um evento de trânsito registrado, e a próxima parte do seu caminho (`FromNode` -> `ToNode`) corresponde ao que está no evento, ele aciona o recálculo da rota (`best_path`).

Portanto, o `events_handler` é o gatilho que permite que a simulação reaja a condições de tráfego imprevistas e dinâmicas, informadas por sistemas externos.

```erlang
listen_for_events() ->
	setup(),
	timer:sleep(1000), // Espera um pouco para garantir que o resto da simulação subiu

	// Conecta-se ao RabbitMQ
	Hostname = os:getenv( "RABBITMQ_HOST", "localhost" ),
	{ok, Connection} = amqp_connection:start(#amqp_params_network{host=Hostname}),
	{ok, Channel} = amqp_connection:open_channel(Connection),

	// Define um "exchange" do tipo 'topic' chamado "traffic_sign"
	// Um exchange recebe mensagens de produtores e as empurra para as filas
	X = <<"traffic_sign">>,
	BindKey = <<"#">>, // O '#' é um curinga que significa "receba todas as mensagens"

	// Cria uma fila anônima e exclusiva para este handler
	#'queue.declare_ok'{queue = Q} = amqp_channel:call(Channel, #'queue.declare'{}),

	// Vincula a fila ao exchange
	QueueBind = #'queue.bind'{queue = Q,
				  exchange = X,
				  routing_key = BindKey},
	#'queue.bind_ok'{} = amqp_channel:call(Channel, QueueBind),

	// Começa a consumir mensagens da fila
	Sub = #'basic.consume'{queue = Q, no_ack = true},
	#'basic.consume_ok'{} = amqp_channel:call(Channel, Sub),

	loop(Channel). // Entra no loop para processar as mensagens recebidas
```

```erlang
loop(Channel) ->
	receive
		{#'basic.deliver'{ exchange=_Exchange, routing_key=_RoutingKey}, Content} ->
			#amqp_msg{payload = Payload} = Content,

			// A mensagem esperada é no formato "NodeAtual.NodeOrigem.NodeDestino"
			[ CurrentNode, FromNode, ToNode ] = string:tokens( binary_to_list(Payload), "." ),
			io:format("Current Node: ~p~nFrom Node: ~p~nTo Node: ~p~n", [CurrentNode, FromNode, ToNode]),

			// Insere o evento na tabela ETS 'traffic_events'
			ets:insert(traffic_events, { CurrentNode, { FromNode, ToNode } }),
			loop(Channel)
	end.
```

---

**User**

me explique a street

---

**Cursor**

Com certeza. O ator `class_Street` na versão do Kanashiro, apesar do nome, tem uma responsabilidade muito mais ampla do que apenas representar "ruas". Ele atua como o **inicializador e configurador principal do ambiente da simulação**.

Analisando o arquivo `kanashiro/class_Street.erl`, sua função mais importante é a `construct/6`. Vamos detalhar o que ela faz passo a passo:

### Função `construct/6` - O Grande Inicializador

Quando o `class_Street` é criado no início da simulação, ele executa as seguintes tarefas:

1.  **Cria a Tabela `list_streets`**:
    ```erlang
    case ets:info(list_streets) of
        undefined -> ets:new(list_streets, [public, set, named_table]);
        _ -> ok
    end,
    ```
    Esta é talvez a tabela ETS mais importante. Ela armazena os dados estáticos de cada via (aresta do grafo), como seu comprimento, capacidade, velocidade livre, etc. É consultada constantemente pelos veículos para calcular o tempo de viagem.

2.  **Cria a Tabela `waiting_bus`**:
    ```erlang
    case ets:info(waiting_bus) of
        undefined -> ets:new(waiting_bus, [public, bag, named_table]);
        _ -> ok
    end,
    ```
    Esta tabela é usada para gerenciar passageiros que estão esperando por um ônibus em um ponto específico.

3.  **Cria a Tabela `traffic_events` e Inicia o `events_handler`**:
    ```erlang
    case ets:info(traffic_events) of
        undefined ->
            ets:new(traffic_events, [public, set, named_table]),
            spawn(events_handler, listen_for_events, []);
        _ -> ok
    end,
    ```
    Este é um passo crucial para a funcionalidade de eventos dinâmicos.
    *   Ele cria a tabela `traffic_events`, que será preenchida pelo `events_handler`.
    *   Ele então **inicia o ator `events_handler`** (`spawn(events_handler, listen_for_events, [])`), que, como vimos, se conectará ao RabbitMQ para ouvir por eventos de tráfego externos.

4.  **Popula a Tabela `list_streets`**:
    ```erlang
    iterate_list( ListEdges ),
    ```
    Ele chama `iterate_list/1`, que recebe a lista de todas as ruas do mapa (lida a partir de um arquivo de configuração no início da simulação) e insere cada uma delas como um registro na tabela `list_streets`.

    ```erlang
    iterate_list([ Element | List ]) ->
        Vertices = element( 1, Element),
        { Id , Length , Capacity , Freespeed , Count , From , To } = element(2, Element),
        ets:insert(list_streets, {Vertices,  Id , Length , Capacity , Freespeed , Count , From , To , 0 , 1 , 4 }),
        iterate_list( List ).
    ```

5.  **Configura Opções Globais**:
    ```erlang
    create_option_table( LogName , Paths ),
    ```
    Esta função `create_option_table/2` cria a tabela `options`, que armazena configurações e referências globais, como o arquivo de log e os caminhos (paths) para as dependências do projeto (como o cliente AMQP).

### Outras Funções

-   **`actSpontaneous/1`**: No `class_Street`, esta função está praticamente vazia. Isso significa que, após a sua configuração inicial, este ator se torna passivo, não realizando ações proativas a cada *tick* da simulação. Seu trabalho principal já foi feito.
-   **`destruct/1`**: É chamada no final da simulação e sua principal responsabilidade é fechar o arquivo de log para garantir que todos os dados sejam salvos corretamente.

### Resumo

O `class_Street` é o "maestro" que prepara o palco para a simulação. Ele não representa uma única rua, mas sim o **conjunto de toda a infraestrutura viária e de comunicação**. Suas responsabilidades são:

-   **Criar as estruturas de dados centrais** (tabelas ETS).
-   **Iniciar o listener de eventos externos** (`events_handler`).
-   **Carregar os dados do mapa** na memória (`list_streets`).
-   **Configurar parâmetros globais** da simulação.

Depois de executar essas tarefas na sua criação, ele permanece inativo, servindo como a fundação sobre a qual todos os outros atores (carros, ônibus, eventos) irão operar.

```erlang
    case ets:info(list_streets) of
        undefined -> ets:new(list_streets, [public, set, named_table]);
        _ -> ok
    end,
```

```erlang
    case ets:info(waiting_bus) of
        undefined -> ets:new(waiting_bus, [public, bag, named_table]);
        _ -> ok
    end,
```

```erlang
    case ets:info(traffic_events) of
        undefined ->
            ets:new(traffic_events, [public, set, named_table]),
            spawn(events_handler, listen_for_events, []);
        _ -> ok
    end,
```

```erlang
    iterate_list( ListEdges ),
```

```erlang
    iterate_list([ Element | List ]) ->
        Vertices = element( 1, Element),
        { Id , Length , Capacity , Freespeed , Count , From , To } = element(2, Element),
        ets:insert(list_streets, {Vertices,  Id , Length , Capacity , Freespeed , Count , From , To , 0 , 1 , 4 }),
        iterate_list( List ).
```

```erlang
    create_option_table( LogName , Paths ),
```

