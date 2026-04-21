
# Roteiro de Testes Funcionais — Micromouse

---

## Testes Unitários

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

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-08** | **Avanço de Célula** | Validar deslocamento motor + encoder. | CT-03, 05, 07 aprovados; Corredor reto. | 1. Marcar posição inicial.<br>2. Comandar avanço (1 célula).<br>3. Medir parada. | Parada precisa dentro da célula seguinte. | Ajustar conversão pulsos -> cm no código. | Validar 5 avanços consecutivos com precisão. |
| **CT-09** | **Trajetória Contínua** | Validar linearidade em múltiplas células. | CT-08 aprovado; Corredor reto (4 células). | 1. Avançar 3 células seguidas.<br>2. Observar centralização final. | Robô centralizado na 3ª célula, sem colisões laterais. | Calibrar velocidades dos motores (trimpot ou software). | Confirmar trajetória reta em percurso longo. |
| **CT-10** | **Correção de Rumo** | Validar alinhamento dinâmico via sensores. | CT-02, 09 aprovados; Corredor padrão. | 1. Iniciar robô torto (10°).<br>2. Observar autocorreção. | Robô se alinha e percorre o eixo central sem tocar paredes. | Revisar lógica de controle Proporcional (PID) lateral. | Confirmar alinhamento partindo de diversos ângulos. |
| **CT-11** | **Curvas de 90°** | Validar rotação angular precisa. | CT-08 aprovado; Trecho com curva. | 1. Executar curva Esq.<br>2. Executar curva Dir.<br>3. Checar alinhamento. | Robô finaliza rotação alinhado ao novo corredor. | Ajustar constantes de tempo/delay de rotação. | Confirmar alinhamento consistente em 3 ciclos. |
| **CT-12** | **Meia-Volta (180°)** | Validar detecção de Dead End e retorno. | CT-07, 11 aprovados; Célula sem saída. | 1. Entrar no corredor cego.<br>2. Observar manobra de saída. | Identificação do bloqueio + rotação 180° e saída limpa. | Corrigir lógica condicional de sensores (F+L+R). | Validar manobra sem colisões em beco sem saída. |

---

## Testes de Sistema

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-13** | **Navegação Completa** | Validar resolução do labirinto. | CT-08 a 12 aprovados; Labirinto 4x4; Bateria OK. | 1. Posicionar na partida.<br>2. Iniciar modo autônomo.<br>3. Monitorar percurso. | Chegada ao destino em < 5min e sem colisões. | Identificar falha local e re-testar integração específica. | Validar percurso total sem intervenção humana. |

## Template para Preenchimento

| Código | Caso de Teste | Objetivo | Pré-condições | Procedimentos | Resultado Esperado | Reparo | Pós-Reparo |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CT-0X** | **Nome do Teste** | Finalidade técnica... | Estado do sistema... | 1. Passo A<br>2. Passo B | Pós-condição final... | Ação corretiva... | Re-teste final... |

---
