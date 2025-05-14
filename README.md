# Projeto Blinky STM32 com Ativação do Sistema

Este projeto demonstra uma aplicação fundamental de piscar LEDs para um microcontrolador STM32, aprimorada com um recurso de ativação/desativação do sistema controlado por um botão de pressão.

## Visão Geral do Projeto

A funcionalidade principal envolve alternar dois LEDs (Verde e Vermelho) em uma taxa consistente quando o sistema está ativo. Um terceiro LED (Amarelo) permanece apagado durante esta sequência de piscadas. O estado ativo do sistema é alternado pressionando um botão azul conectado ao microcontrolador.

- **Estado Ativo:** Quando o sistema está ativo, os LEDs Verde e Vermelho piscam alternadamente com um período de 1 segundo (500ms LIGADO, 500ms DESLIGADO). O LED Amarelo permanece apagado.
- **Estado Inativo:** Quando o sistema está inativo, todos os três LEDs (Verde, Vermelho e Amarelo) são desligados.

O piscar é precisamente temporizado usando um timer de hardware (TIM2) configurado para gerar interrupções periódicas. A entrada do botão de pressão é monitorada dentro do loop principal, incorporando um mecanismo básico de debounce para garantir transições de estado confiáveis.

## Pré-requisitos de Hardware

Para executar este projeto, você precisará dos seguintes componentes de hardware:

- Uma placa de desenvolvimento de microcontrolador STM32 (A série STM32 específica usada neste código é implícita pelo uso da biblioteca HAL, mas não explicitamente declarada).
- Três LEDs: Verde, Vermelho e Amarelo.
- Três resistores limitadores de corrente adequados para os LEDs.
- Um botão de pressão momentâneo (Azul neste caso).
- Fios de conexão para interfacear os componentes com a placa STM32.

## Pré-requisitos de Software

O ambiente de desenvolvimento e as ferramentas de software necessárias são:

- STM32CubeIDE ou outro Ambiente de Desenvolvimento Integrado (IDE) compatível para desenvolvimento STM32.
- A biblioteca STM32 Hardware Abstraction Layer (HAL), que geralmente está incluída no STM32CubeIDE.

## Estrutura dos Arquivos do Projeto

O código-fonte do projeto está organizado nos seguintes arquivos principais:

- **`main.c`:** Este arquivo contém a lógica principal da aplicação, incluindo:
    - Inicialização do sistema do microcontrolador e periféricos (clock, GPIO, timer).
    - O loop principal do programa que monitora o estado do botão.
    - Rotinas de serviço de interrupção (ISRs) para o timer.
    - A função `HandleButton()` para detectar e processar as pressões do botão.
- **`stm32xxxx_hal_conf.h`:** O arquivo de configuração da biblioteca HAL (substitua `xxxx` pela sua série STM32 específica).
- **`stm32xxxx_it.c`:** Contém os handlers de interrupção, incluindo o do timer TIM2 (substitua `xxxx` pela sua série STM32 específica).
- **`system_stm32xxxx.c`:** Arquivo de código-fonte da configuração do clock do sistema (substitua `xxxx` pela sua série STM32 específica).
- **`stm32xxxx_hal_msp.c`:** Funções de inicialização e desinicialização de periféricos específicos do microcontrolador para a HAL (substitua `xxxx` pela sua série STM32 específica).

## Guia de Conexão de Hardware

Siga estes passos para conectar os componentes de hardware à sua placa de desenvolvimento STM32:

1. **Conexões dos LEDs:**
   - Conecte a perna positiva (ânodo) do LED Verde a um pino GPIO (definido como `LED_GREEN_Pin` em `LED_GREEN_GPIO_Port`) através de um resistor limitador de corrente. Conecte a perna negativa (cátodo) ao terra (GND).
   - Da mesma forma, conecte o LED Vermelho ao seu pino GPIO designado (`LED_RED_Pin` em `LED_RED_GPIO_Port`) com um resistor limitador de corrente ao GND.
   - Conecte o LED Amarelo ao seu pino GPIO designado (`LED_YELLOW_Pin` em `LED_YELLOW_GPIO_Port`) com um resistor limitador de corrente ao GND.

2. **Conexão do Botão de Pressão:**
   - Conecte um terminal do Botão Azul a um pino GPIO (definido como `BUTTON_BLUE_Pin` em `BUTTON_BLUE_GPIO_Port`).
   - Conecte o outro terminal do botão ao terra (GND).
   - **Importante:** O código configura o pino do botão com um resistor de pull-down interno. Se sua placa não tiver essa opção ou você preferir um resistor externo, conecte um resistor de pull-down de 10kΩ (ou valor similar) entre o pino GPIO do botão e o GND.

## Como Construir e Executar o Projeto

1. **Importar o Projeto:** Importe os arquivos do projeto para o seu STM32CubeIDE ou ambiente de desenvolvimento STM32 preferido.
2. **Configurar as Definições do Projeto:** Certifique-se de que as definições do projeto estejam corretamente configuradas para o seu microcontrolador STM32 específico. Isso inclui selecionar o dispositivo correto nas configurações do projeto.
3. **Construir o Projeto:** Compile o código-fonte. O IDE gerará os arquivos binários necessários.
4. **Fazer o Flash para o STM32:** Conecte sua placa de desenvolvimento STM32 ao seu computador usando uma interface adequada (por exemplo, USB). Use o utilitário de flash do IDE para carregar o binário compilado para a memória do microcontrolador.
5. **Observar a Saída:** Uma vez que o código esteja rodando na placa STM32:
   - Inicialmente, todos os LEDs devem estar apagados.
   - Pressione o Botão Azul. Os LEDs Verde e Vermelho devem começar a piscar alternadamente a uma taxa de 1 Hz. O LED Amarelo deve permanecer apagado.
   - Pressione o Botão Azul novamente. Todos os três LEDs devem se apagar, indicando que o sistema agora está inativo.
   - Pressionamentos subsequentes do Botão Azul alternarão o sistema entre os estados ativo (piscando) e inativo (todos os LEDs apagados).

## Destaques da Explicação do Código

- **Inicialização do Timer (`MX_TIM2_Init`)**: O timer TIM2 é configurado para gerar uma interrupção a cada 500 milissegundos. Isso é alcançado definindo os valores do prescaler e do período com base na frequência do clock do sistema.
- **Inicialização dos GPIOs (`MX_GPIO_Init`)**: Os pinos GPIO conectados aos LEDs Verde, Vermelho e Amarelo são configurados como pinos de saída push-pull. O pino GPIO conectado ao Botão Azul é configurado como um pino de entrada com um resistor de pull-down.
- **Callback de Interrupção do Timer (`HAL_TIM_PeriodElapsedCallback`)**: Esta função é chamada automaticamente quando o período do timer TIM2 expira. Ela verifica se a interrupção veio do TIM2 e se a flag `system_active` está definida. Se ambas forem verdadeiras, ela alterna a variável `led_state` e define a saída dos LEDs Verde e Vermelho de acordo, enquanto garante que o LED Amarelo permaneça apagado.
- **Tratamento do Botão (`HandleButton`)**: Esta função é chamada repetidamente no loop principal. Ela lê o estado do Botão Azul. Um pequeno atraso (`HAL_Delay(50)`) é introduzido para um debounce básico. Se o botão for pressionado, a flag `system_active` é alternada. Se o sistema for desativado, todos os LEDs são apagados. A função também inclui um loop para esperar que o botão seja liberado, evitando múltiplas alternâncias com uma única pressão.
- **Loop Principal (`main`)**: O loop principal chama continuamente a função `HandleButton()` com um pequeno atraso, permitindo que o estado do botão seja verificado e processado.

Este projeto fornece um exemplo básico, porém ilustrativo, de controle de LEDs com um botão e um timer em um microcontrolador STM32 usando a biblioteca HAL. Ele pode servir como base para aplicações embarcadas mais complexas.
