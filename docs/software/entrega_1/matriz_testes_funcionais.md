
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
| **CT-07** | **Parada de Segurança** | Reação ao sensor frontal durante movimento. | CT-01 e CT-03 aprovados; Parede a 30cm. | 1. Avançar rumo à parede.<br>2. Observar ponto de frenagem. | Parada completa sem colisão física. | Aumentar distância de segurança (threshold) no código. | Validar parada segura em múltiplas distâncias. |

---

## Testes Integrados

<!--
  Aqui as peças são testadas funcionando juntas.
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

<!--
## Template para Preenchimento

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-0X** | **Nome do Teste** | Finalidade técnica... | Estado do sistema... | 1. Passo A<br>2. Passo B | Pós-condição final... | Ação corretiva... | Re-teste final... |

-->
