# Especificação de Requisitos Não-Funcionais

## 1. Introdução

O sistema é composto por dois blocos:
- O firmware embarcado no micromouse, que é responsável pelo controle autônomo do robô, e a aplicação web, que é responsável por exibir telemetria em tempo real e armazenar dados de cada corrida em um banco de dados.

## 2. Desempenho: Métricas de Tempo

### RNF-01 — Latência de Transmissão de Telemetria

| Atributo | Valor |
|---|---|
| **Descrição** | O micromouse deve transmitir dados de telemetria ao sistema web com latência máxima aceitável |
| **Métrica** | Tempo entre geração do dado no robô e exibição na interface web |
| **Limite máximo** | ≤ 500 ms (em condições normais de operação na rede local) |
| **Condição de medição** | Rede Wi-Fi local, distância de até 10 metros entre robô e roteador |
| **Justificativa** | Telemetria "em tempo real" percebida pelo usuário exige atualização sub-segundo |

### RNF-02 — Taxa de Atualização da Interface Web

| Atributo | Valor |
|---|---|
| **Descrição** | A interface web deve atualizar os dados exibidos (velocidade, trajeto, bateria) com frequência mínima |
| **Métrica** | Intervalo entre atualizações consecutivas dos dados na tela |
| **Limite máximo** | ≤ 1 segundo por ciclo de atualização |
| **Condição de medição** | Navegador moderno (Chrome/Firefox), conexão local ativa |
| **Justificativa** | Garantir que o observador acompanhe o progresso do micromouse em tempo real |

A interface deve se atualizar a cada 1 segundo. Para que isso seja possível, a transmissão dos dados do robô até o servidor precisa ser concluída em menos tempo do que esse intervalo, deixando margem para o processamento no servidor e a renderização no navegador. Adotando uma divisão conservadora: metade do tempo para transmissão, metade para processamento e renderização, chegamos a 500 ms como limite para a etapa de transmissão. Em redes locais Wi-Fi comuns, latências na faixa de 10 a 50 ms são típicas, de modo que 500 ms é um limite bastante seguro e alcançável.

### RNF-03 — Tempo de Carregamento da Página Web

| Atributo | Valor |
|---|---|
| **Descrição** | A aplicação web deve carregar completamente dentro de um tempo aceitável |
| **Métrica** | Tempo desde a requisição HTTP até o estado "interativo" da página (TTI) |
| **Limite máximo** | ≤ 3 segundos (em rede local) |
| **Condição de medição** | Conexão local, hardware padrão de laboratório |
| **Justificativa** | Evitar espera excessiva antes do início da corrida |

Este requisito se aplica ao carregamento inicial da aplicação, não à telemetria em si. O limite de 3 segundos é amplamente adotado como referência na indústria (Google Core Web Vitals define esse intervalo como limite aceitável de carregamento)

### RNF-04 — Tempo de Ciclo de Controle do Firmware

| Atributo | Valor |
|---|---|
| **Descrição** | Tempo máximo de execução do loop principal de controle no microcontrolador |
| **Limite máximo** | ≤ 10 ms por ciclo (equivalente a ≥ 100 Hz) |
| **Condição de medição** | Execução no microcontrolador alvo, labirinto 16×16 em operação |

Este valor vem das características físicas da competição. O robô precisa detectar paredes, corrigir sua trajetória e tomar decisões de navegação enquanto se move. As células têm 18 cm de lado e robôs em competições Micromouse tipicamente atingem velocidades entre 0,5 e 2 m/s. O tempo disponível para atravessar metade de uma célula é de aproximadamente 45 a 180 ms. Para ter pelo menos 4 a 5 ciclos de controle nesse intervalo (garantindo estabilidade), a frequência mínima deve ser próxima de 100 Hz, ou seja, 10 ms por ciclo. Este requisito deve ser revisado após a definição do microcontrolador.

### RNF-05 — Tempo de Armazenamento no Banco de Dados

| Atributo | Valor |
|---|---|
| **Descrição** | Tempo máximo entre a detecção do objetivo pelo micromouse e a confirmação de escrita dos dados finais no banco |
| **Limite máximo** | ≤ 2 segundos |
| **Condição de medição** | Banco de dados local ou servidor na mesma rede |

Ao final de uma corrida, o sistema precisa garantir que todos os dados sejam armazenados antes que o operador desligue o robô ou encerre a aplicação. Dois segundos é tempo suficiente para que a interface detecte o evento de conclusão, monte o objeto de dados e execute uma operação de escrita em banco de dados local.

## 3. Capacidade e Carga

### RNF-06 — Volume de Registros por Corrida

| Atributo | Valor |
|---|---|
| **Descrição** | Quantidade mínima de registros de telemetria que o sistema deve ser capaz de armazenar por corrida |
| **Limite mínimo** | ≥ 10.000 registros por corrida |
| **Estimativa de tamanho** | ~1 MB por corrida completa |

O maior labirinto tem 16×16 = 256 células. O micromouse pode precisar visitar cada célula mais de uma vez durante a fase de exploração (algoritmos como flood fill revisitam células até mapear o labirinto completamente). Estimando uma média de 3 visitas por célula e leituras de sensores a 100 Hz, com duração média de corrida de 2 a 3 minutos, chegamos a aproximadamente 12.000 a 18.000 leituras por corrida. O limite de 10.000 registros é um mínimo conservador que cobre também corridas mais rápidas nos labirintos menores (4×4 e 8×8).

### RNF-07 — Total de Corridas Armazenadas

| Atributo | Valor |
|---|---|
| **Descrição** | Quantidade mínima de corridas que o banco de dados deve suportar sem degradação de desempenho nas consultas |
| **Limite mínimo** | ≥ 100 corridas |
| **Tempo de consulta** | Resultado de consulta por labirinto específico em ≤ 1 segundo |

Estimando aproximadamente 10 sessões de teste distribuídas ao longo do semestre, com 3 labirintos por sessão e até 3 tentativas cada, chegamos a cerca de 90 corridas no total. O limite de 100 corridas cobre esse uso com uma pequena margem de segurança. Com ~1 MB por corrida, o banco precisaria suportar cerca de 100 MB de dados, volume sem qualquer impacto para bancos relacionais como PostgreSQL, SQLite ou MySQL.

### RNF-08 — Usuários Simultâneos no Sistema Web

| Atributo | Valor |
|---|---|
| **Descrição** | Número mínimo de usuários que podem acessar a interface web simultaneamente sem degradação |
| **Limite mínimo** | ≥ 10 usuários simultâneos |
| **Critério de degradação** | Latência de atualização mantida em ≤ 1 segundo com 10 clientes ativos |

O contexto mais exigente é a apresentação final, onde todos os professores da disciplina e os próprios integrantes da equipe podem estar acompanhando simultaneamente. São 5 professores avaliadores e aproximadamente 5 a 6 membros do grupo, totalizando cerca de 10 a 11 usuários simultâneos no pico de uso. O limite de ≥ 10 conexões simultâneas cobre esse cenário com folga mínima. Ainda assim, precisa ser explicitamente testado pois o sistema de telemetria mantém conexões abertas continuamente (via WebSocket ou polling), o que consome mais recursos do que requisições comuns.


### RNF-09 — Tamanho Máximo do Pacote de Telemetria

| Atributo | Valor |
|---|---|
| **Descrição** | Tamanho máximo de cada pacote de dados enviado pelo micromouse ao servidor por ciclo |
| **Limite máximo** | ≤ 512 bytes por pacote |

Microcontroladores tipicamente utilizados em projetos Micromouse (como STM32 ou ESP32) possuem buffers de comunicação limitados. Pacotes menores também reduzem a latência de transmissão e o risco de fragmentação na rede. Os dados necessários por ciclo: posição, velocidade, nível de bateria e timestamp, podem ser representados em menos de 100 bytes com estrutura binária eficiente. O limite de 512 bytes é generoso o suficiente para acomodar inclusive formatos texto como JSON, que são mais fáceis de depurar durante o desenvolvimento.

## 4. Segurança

### RNF-10 — Integridade dos Dados de Corrida

| Atributo | Valor |
|---|---|
| **Descrição** | Os dados gravados no banco após uma corrida não devem poder ser modificados pela interface web |
| **Requisito** | Interface de consulta somente leitura; nenhum endpoint de modificação ou exclusão exposto publicamente |

Como os dados serão utilizados para avaliação, é fundamental garantir que não haja possibilidade de alteração posterior dos resultados, seja acidental ou intencional. A forma mais simples de atender a isso é não implementar endpoints de atualização ou deleção no sistema web, deixando qualquer eventual correção como uma operação administrativa direta no banco de dados, com acesso restrito.

## 5. Usabilidade

### RNF-11 — Visibilidade dos Dados de Telemetria

| Atributo | Valor |
|---|---|
| **Descrição** | Todos os 6 campos de telemetria exigidos pelo projeto devem estar visíveis sem necessidade de rolagem |
| **Campos obrigatórios** | Tipo do labirinto, trajeto, consumo de bateria, velocidade média, tempo de conclusão, desafio cumprido (S/N) |
| **Requisito de layout** | Interface responsiva, adaptável a diferentes tamanhos de tela |

### RNF-12 — Compatibilidade de Navegadores

| Atributo | Valor |
|---|---|
| **Descrição** | A aplicação deve funcionar corretamente nos navegadores mais utilizados |
| **Requisito** | Chrome ≥ 110, Firefox ≥ 110, Edge ≥ 110 |

Para garantir que o sistema funcione nos equipamentos disponíveis durante as avaliações, independentemente do laboratório ou computador utilizado, é necessário definir um conjunto mínimo de navegadores suportados. As versões ≥ 110 foram escolhidas por oferecerem amplo suporte a APIs modernas como WebSocket, CSS Grid e Fetch API, recursos que provavelmente serão utilizados no desenvolvimento da interface de telemetria.





