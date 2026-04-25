
# Roteiro de Testes Funcionais — Micromouse

<!--
  SOBRE ESTE ROTEIRO
  ==================
  Os testes estão divididos em três partes:

  1. Unitários  — Testa cada peça sozinha (sensor, motor, encoder).
  2. Integrados — Testa as peças funcionando juntas e a lógica do Flood Fill.
  3. De Sistema — Testa o robô resolvendo o labirinto de verdade, em duas fases:
       Fase 1 (Exploração): O robô anda devagar, descobre as paredes e monta um mapa.
       Fase 2 (Corrida):    Com o mapa pronto, o robô refaz o caminho mais curto em velocidade máxima.

  Seguir sempre a parede direita NÃO funciona em labirintos de competição
  porque existem "ilhas" de paredes que fazem o robô andar em círculos.
  Por isso usamos o Flood Fill.
-->

---

## Testes Unitários

<!--
  Cada teste aqui verifica uma peça por vez, separada do resto.
  Se o sensor ou o motor não funcionar sozinho, não adianta testar o robô inteiro.
-->

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-01** | **Detecção Frontal** | Diferenciar parede próxima (< 5cm) de ausência. | Robô ligado; Serial Monitor aberto. | 1. Aproximar mão (3cm).<br>2. Anotar Serial.<br>3. Afastar mão (10cm). | Mudança clara de valor entre os cenários (Alto vs Baixo). | Ajustar valor de referência de detecção no código. | Confirmar diferença de valores nos dois cenários. |
| **CT-02** | **Detecção Lateral** | Verificar sensores laterais sem interferência mútua. | CT-01 aprovado; sensores laterais ativos. | 1. Aproximar mão lateral esq.<br>2. Verificar se dir. não muda.<br>3. Repetir para dir. | Detecção independente por sensor, sem interferência. | Verificar pinagem no código ou alinhar sensores físicos. | Confirmar independência das leituras laterais. |
| **CT-03** | **Sentido dos Motores** | Validar comandos de Frente e Ré isolados. | Rodas suspensas; bateria carregada. | 1. Testar Frente (Esq/Dir).<br>2. Testar Ré (Esq/Dir).<br>3. Observar rotação. | Rotação correta conforme comando, sem travamentos. | Inverter lógica no código ou fiação do motor. | Validar todos os sentidos de rotação novamente. |
| **CT-04** | **Níveis de Potência** | Validar diferenciação (30%, 60%, 100%). | CT-03 aprovado; rodas suspensas. | 1. Testar Potência Baixa.<br>2. Testar Média.<br>3. Testar Máxima. | Velocidade aumenta visivelmente em cada nível. | Verificar tensão da fonte ou PWM no código. | Confirmar ganho de velocidade perceptível. |
| **CT-05** | **Leitura de Encoder** | Validar contagem de pulsos (volta completa). | Codigo ativo; Serial Monitor aberto. | 1. Zerar contador.<br>2. Girar roda manualmente (1 volta).<br>3. Ler Serial. | Registro igual ao especificado (variação +/- 1 pulso). | Validar pino de leitura ou lógica de interrupção. | Garantir contagem consistente em 3 voltas. |
| **CT-06** | **Inicialização Geral** | Garantir boot limpo de todos componentes. | Robô montado; Serial Monitor aberto. | 1. Ligar robô.<br>2. Monitorar inicialização.<br>3. Aguardar ociosidade. | Boot sem erros; robô imóvel e pronto para uso. | Identificar erro no Serial; checar pinagem/conexões. | Confirmar sequência de boot estável. |

---

## Testes de Integração

<!--
  Aqui as peças são testadas funcionando juntas.
  CT-07: Parada de Segurança — interação entre sensor frontal e motor.
  CT-08 a CT-12: O robô anda, faz curvas e para — motores e sensores cooperando.
  CT-13: O robô guarda no mapa onde tem parede e onde não tem.
  CT-14: O Flood Fill funciona assim:
    - O centro do labirinto recebe o valor 0 (é o destino).
    - As células ao redor recebem 1, depois 2, 3... como uma enchente se espalhando.
    - O robô sempre anda para o vizinho com o número menor.
    - Se descobre uma parede nova, refaz a enchente e continua.
    - Com isso, ele sempre encontra o caminho mais curto que conhece até ali.
-->

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-07** | **Parada de Segurança** | Reação ao sensor frontal durante movimento. | CT-01 e CT-03 aprovados; Parede a 30cm. | 1. Avançar rumo à parede.<br>2. Observar ponto de frenagem. | Parada completa sem colisão física. | Aumentar distância de segurança (threshold) no código. | Validar parada segura em múltiplas distâncias. |
| **CT-08** | **Avanço de Célula** | Validar deslocamento motor + encoder. | CT-03, 05, 07 aprovados; Corredor reto. | 1. Marcar posição inicial.<br>2. Comandar avanço (1 célula).<br>3. Medir parada. | Parada precisa dentro da célula seguinte. | Ajustar conversão pulsos -> cm no código. | Validar 5 avanços consecutivos com precisão. |
| **CT-09** | **Trajetória Contínua** | Validar linearidade em múltiplas células. | CT-08 aprovado; Corredor reto (4 células). | 1. Avançar 3 células seguidas.<br>2. Observar centralização final. | Robô centralizado na 3ª célula, sem colisões laterais. | Calibrar velocidades dos motores (trimpot ou software). | Confirmar trajetória reta em percurso longo. |
| **CT-10** | **Correção de Rumo** | Validar alinhamento dinâmico via sensores. | CT-02, 09 aprovados; Corredor padrão. | 1. Iniciar robô torto (10°).<br>2. Observar autocorreção. | Robô se alinha e percorre o eixo central sem tocar paredes. | Revisar lógica de controle Proporcional (PID) lateral. | Confirmar alinhamento partindo de diversos ângulos. |
| **CT-11** | **Curvas de 90°** | Validar rotação angular precisa. | CT-08 aprovado; Trecho com curva. | 1. Executar curva Esq.<br>2. Executar curva Dir.<br>3. Checar alinhamento. | Robô finaliza rotação alinhado ao novo corredor. | Ajustar constantes de tempo/delay de rotação. | Confirmar alinhamento consistente em 3 ciclos. |
| **CT-12** | **Meia-Volta (180°)** | Validar detecção de Dead End e retorno. | CT-07, 11 aprovados; Célula sem saída. | 1. Entrar no corredor cego.<br>2. Observar manobra de saída. | Identificação do bloqueio + rotação 180° e saída limpa. | Corrigir lógica condicional de sensores (F+L+R). | Validar manobra sem colisões em beco sem saída. |
| **CT-13** | **Escrita de Mapa** | Validar se o robô salva corretamente as paredes na matriz de memória. | CT-01, CT-02 aprovados; robô em uma célula de teste; Serial aberto. | 1. Colocar o robô em célula com paredes conhecidas (ex: Esq e Frente).<br>2. Ler os sensores.<br>3. Imprimir matriz no Serial. | A matriz reflete exatamente as paredes lidas na célula atual (ex: salva 1 onde tem parede e 0 onde não tem). | Revisar uso de índices (X, Y) ou manipulação de bits na gravação do array. | Validar o mapeamento movendo o robô para 3 células diferentes. |
| **CT-14** | **Lógica Flood Fill** | Validar a atualização dos pesos numéricos para encontrar o centro. | CT-13 aprovado; mapa inicializado com paredes (simulação via código). | 1. Injetar um labirinto simulado na memória.<br>2. Rodar a função Flood Fill.<br>3. Verificar no Serial o próximo movimento. | O algoritmo atribui os "pesos" corretamente e escolhe mover para a célula vizinha com o menor valor numérico. | Corrigir lógica da fila ou cálculo de células adjacentes no código. | O algoritmo deve traçar o caminho perfeito em um mapa estático conhecido no Serial. |

---

## Testes de Sistema

<!--
  O robô resolve o labirinto em duas etapas:

  Fase 1 — Exploração (CT-15)
    O robô não sabe nada sobre o labirinto.
    Ele anda devagar, célula por célula, anotando onde tem parede.
    Usa o Flood Fill para escolher para onde ir.
    Cada vez que encontra uma parede nova, refaz a "enchente" de números.
    O importante aqui é chegar ao centro sem bater — pode ser lento.

  Fase 2 — Corrida (CT-16)
    Agora o robô já tem o mapa na memória.
    Ele calcula o caminho mais curto e percorre em velocidade máxima.
    É essa fase que vale na competição.

  Para o nosso projeto (PI1), o objetivo é:
    Fazer o Flood Fill básico funcionar, explorar o labirinto inteiro
    e depois tentar a corrida. A primeira vez vai ser devagar, e tá tudo bem.
-->

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-15** | **Fase 1 — Exploração** | Validar a navegação autônoma do início ao centro construindo o mapa. | CT-08 a CT-14 aprovados; labirinto físico montado; robô na largada. | 1. Ligar o robô no Modo Exploração.<br>2. Soltar o robô.<br>3. Acompanhar mapeamento e recalculos até o centro. | O robô anda célula a célula, não bate nas paredes, refaz o Flood Fill ao achar becos e para na célula central. | Ajustar máquina de estados principal ou recalibrar constantes do PID. | O robô deve conseguir chegar ao centro sozinho pelo menos uma vez, mesmo que lento. |
| **CT-16** | **Fase 2 — Corrida** | Validar a execução do trajeto ótimo (caminho mais curto) utilizando o mapa salvo. | CT-15 concluído com sucesso (mapa completo na memória). | 1. Posicionar robô de volta na largada.<br>2. Acionar Modo Corrida.<br>3. Observar o trajeto escolhido pelo robô. | O robô percorre o caminho mais curto de forma contínua e direta, ignorando os becos mapeados anteriormente. | Ajustar leitura da matriz de caminhos na memória para evitar re-exploração ou loop. | Completar o percurso de forma direta do início ao centro, sem entrar em becos já conhecidos. |

---

## Testes de Servidor (Backend)

<!--
  Estes testes verificam o pool Servidor do BPMN: recepção de telemetria,
  retransmissão via WebSocket ao Dashboard e persistência no banco de dados.
  Nenhuma interação com o firmware real é necessária — os pacotes podem ser
  simulados via script ou ferramenta como Postman / curl.
-->

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-17** | **Recepção de Telemetria** | Verificar se o servidor recebe corretamente um pacote de telemetria enviado pelo Micromouse. | Servidor em execução; endpoint de recepção ativo. | 1. Enviar pacote de telemetria simulado (posição, paredes, bateria, velocidade, timestamp).<br>2. Verificar log do servidor. | Servidor registra o recebimento sem erro; todos os campos do pacote são identificados. | Revisar parsing do pacote ou configuração do endpoint. | Reenviar pacote e confirmar recepção completa. |
| **CT-18** | **Rejeição de Pacote Malformado** | Verificar se o servidor rejeita pacotes inválidos sem travar ou lançar exceção não tratada. | Servidor em execução. | 1. Enviar pacote com campos ausentes.<br>2. Enviar pacote com tipo de dado errado.<br>3. Enviar payload vazio. | Servidor retorna erro apropriado (ex.: 400 Bad Request) e continua operando normalmente. | Adicionar validação de schema na camada de recepção. | Reenviar pacote malformado e confirmar que o servidor permanece estável. |
| **CT-19** | **Ordem: Retransmitir antes de Persistir** | Verificar se o servidor retransmite os dados ao Dashboard via WebSocket antes de gravá-los no banco. | Servidor, Dashboard e banco de dados em execução; cliente WebSocket conectado. | 1. Enviar pacote de telemetria com flag de conclusão.<br>2. Registrar o timestamp de recebimento do dado no Dashboard.<br>3. Registrar o timestamp de inserção no banco. | O Dashboard recebe o dado antes (ou ao mesmo tempo) que o banco registra a inserção — nunca depois. | Revisar ordem de chamadas no handler do servidor (WebSocket primeiro, banco depois). | Repetir o teste e comparar timestamps em 3 execuções. |
| **CT-20** | **Persistência ao Concluir Desafio** | Verificar se os dados são gravados no banco somente após o recebimento da flag de conclusão. | Servidor e banco em execução. | 1. Enviar sequência de pacotes de telemetria sem flag de conclusão.<br>2. Consultar banco após cada envio.<br>3. Enviar pacote final com flag de conclusão.<br>4. Consultar banco novamente. | Nenhum registro no banco durante os passos 1–2; registro inserido somente após o passo 3. | Verificar lógica do gateway "Desafio concluído?" no servidor. | Repetir sequência e confirmar ausência de registro prematuro. |
| **CT-21** | **Corrida Interrompida Não Persiste** | Verificar se uma corrida encerrada sem flag de conclusão não gera registro no banco. | Servidor e banco em execução. | 1. Enviar pacotes de telemetria normalmente.<br>2. Encerrar conexão abruptamente (sem enviar flag de conclusão).<br>3. Consultar banco. | Nenhum registro inserido para essa corrida. | Adicionar tratamento de desconexão abrupta no servidor. | Repetir interrupção e confirmar banco inalterado. |
| **CT-22** | **Consulta Filtrada por Labirinto** | Verificar se a API retorna corretamente os registros de um labirinto específico. | Banco com registros de corridas nos três tipos de labirinto (4×4, 8×8, 16×16). | 1. Chamar endpoint com filtro "4×4".<br>2. Chamar com filtro "8×8".<br>3. Chamar com filtro "16×16".<br>4. Verificar registros retornados em cada chamada. | Cada chamada retorna apenas registros do tipo de labirinto solicitado, sem misturar outros tipos. | Revisar cláusula WHERE da query de consulta. | Repetir consultas e validar segregação dos dados. |
| **CT-23** | **Consulta com Filtro "Todos"** | Verificar se a API retorna todos os registros quando o filtro selecionado é "todos os labirintos". | Banco com registros de ao menos dois tipos de labirinto diferentes. | 1. Chamar endpoint sem filtro de tipo (ou com parâmetro "todos").<br>2. Comparar quantidade de registros retornados com total no banco. | Todos os registros de todas as corridas são retornados, sem omissão. | Verificar ausência de filtro indevido na query padrão. | Inserir novo registro e confirmar que aparece na consulta geral. |
| **CT-24** | **Consulta com Banco Vazio** | Verificar o comportamento da API quando não há corridas cadastradas. | Banco vazio (sem nenhum registro de corrida). | 1. Chamar endpoint de consulta geral.<br>2. Chamar endpoint com filtro de labirinto específico. | API retorna lista vazia (ex.: `[]`) com status 200, sem erro 500 ou exceção. | Adicionar tratamento explícito para resultado vazio na camada de consulta. | Repetir consultas e confirmar retorno vazio sem erro. |

---

## Testes de Dashboard (Frontend)

<!--
  Estes testes verificam o pool Dashboard do BPMN: conexão WebSocket,
  exibição de telemetria em tempo real e consulta de histórico via API REST.
  Podem ser executados com o servidor real ou com um servidor mock que simule
  os eventos WebSocket e as respostas REST.
-->

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-25** | **Conexão WebSocket** | Verificar se o Dashboard estabelece conexão WebSocket automaticamente ao ser acessado. | Servidor em execução com WebSocket ativo. | 1. Abrir o Dashboard no navegador.<br>2. Verificar log de conexão no servidor.<br>3. Verificar indicador de status no Dashboard (se existir). | Conexão WebSocket estabelecida sem interação manual do usuário; servidor registra o cliente conectado. | Verificar URL do WebSocket no frontend e configuração de CORS no servidor. | Recarregar página e confirmar reconexão automática. |
| **CT-26** | **Exibição dos 6 Campos de Telemetria** | Verificar se todos os campos obrigatórios são exibidos em tempo real no painel. | Dashboard conectado ao WebSocket; servidor enviando stream de telemetria simulado. | 1. Iniciar stream de telemetria com todos os campos.<br>2. Verificar no Dashboard a presença e atualização de: Tipo do labirinto, Trajeto, Consumo de bateria, Velocidade média, Tempo decorrido, Status do desafio (S/N). | Todos os 6 campos aparecem e são atualizados a cada pacote recebido. | Identificar campo ausente e corrigir binding no frontend. | Reenviar stream e confirmar atualização de todos os campos. |
| **CT-27** | **Exibição de Dados Consolidados ao Fim da Corrida** | Verificar se o Dashboard detecta o encerramento do stream e exibe os dados finais consolidados. | Dashboard conectado; servidor prestes a enviar pacote de conclusão. | 1. Enviar sequência de pacotes de telemetria.<br>2. Enviar pacote final com flag de conclusão.<br>3. Observar mudança na interface. | Dashboard transita da exibição em tempo real para a tela de dados consolidados sem ação manual do usuário. | Revisar lógica do gateway "Corrida finalizada?" no frontend. | Repetir o ciclo completo e confirmar transição automática. |
| **CT-28** | **Filtro por Labirinto Específico** | Verificar se o usuário consegue consultar o histórico de um labirinto específico. | Banco com registros de corridas; API REST disponível. | 1. Selecionar filtro "4×4" no Dashboard.<br>2. Confirmar requisição enviada ao servidor.<br>3. Verificar dados exibidos. | Apenas registros do labirinto 4×4 são exibidos; dados de outros tipos não aparecem. | Verificar parâmetro de filtro enviado na requisição REST. | Repetir para 8×8 e 16×16 e confirmar segregação correta. |
| **CT-29** | **Filtro "Todos os Labirintos"** | Verificar se o usuário consegue visualizar o histórico completo de todas as corridas. | Banco com registros de ao menos dois tipos de labirinto. | 1. Selecionar opção "Todos" no Dashboard.<br>2. Confirmar requisição enviada.<br>3. Verificar dados exibidos. | Todos os registros de todas as corridas são exibidos, sem omissão de nenhum tipo de labirinto. | Verificar se o frontend envia a requisição sem filtro de tipo. | Inserir novo registro e confirmar que aparece na listagem. |
| **CT-30** | **Reconexão WebSocket** | Verificar o comportamento do Dashboard após perda e restabelecimento da conexão WebSocket. | Dashboard conectado; servidor em execução. | 1. Estabelecer conexão WebSocket.<br>2. Derrubar o servidor temporariamente (simular queda de rede).<br>3. Restaurar o servidor.<br>4. Observar comportamento do Dashboard. | Dashboard exibe indicação de conexão perdida e reconecta automaticamente ao servidor sem necessidade de recarregar a página. | Implementar lógica de reconexão automática (ex.: backoff exponencial) no cliente WebSocket. | Repetir queda de conexão e confirmar reconexão em até 30 segundos. |

---

<!--
## Template para Preenchimento

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-0X** | **Nome do Teste** | Finalidade técnica... | Estado do sistema... | 1. Passo A<br>2. Passo B | Pós-condição final... | Ação corretiva... | Re-teste final... |

-->
