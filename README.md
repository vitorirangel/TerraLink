#TerraLink: Monitoramento Inteligente de Solo

O TerraLink é uma solução de Internet das Coisas (IoT) desenvolvida para auxiliar pequenos e médios agricultores na gestão da agropecuária e otimização do plantio. O dispositivo realiza a leitura da umidade do solo em tempo real, transformando dados brutos em informações claras e precisas para apoiar a tomada de decisões no campo, evitando o desperdício de água e protegendo a saúde da plantação.


#Funcionalidades Principais

* **Monitoramento Contínuo:** Leitura automatizada do nível de umidade do solo.
* **Classificação Inteligente:** Análise dos dados com categorização imediata do estado do solo (`SOLO SECO`, `SOLO IDEAL` ou `SOLO MUITO ÚMIDO`).
* **Sincronização em Nuvem:** Envio dos dados coletados via requisições HTTP para uma base de dados remota.
* **Controle Operacional:** Botão físico com interrupção por hardware que permite pausar ou reativar o sistema instantaneamente.
* **Feedback Visual local:** LEDs indicadores que mostram o status operacional e de conexão do dispositivo.

---

# Arquitetura de Hardware

O projeto foi estruturado utilizando os seguintes componentes:

* **Microcontrolador:** ESP32 (com suporte nativo a Wi-Fi).
* **Sensor:** Sensor de Umidade do Solo (Higrômetro Analógico).
* **Atuadores e Indicadores:** * LED Verde (Indicador de Sistema Ativo/Online).
  * LED Vermelho (Indicador de Alerta/Sistema Parado/Sem Wi-Fi).
  * Botão de Pressão (Push Button) configurado com resistor interno de *Pull-up*.

# Mapeamento de Pinos (GPIOs)

| Componente | Pino ESP32 | Função |
| :--- | :--- | :--- |
| **Sensor de Solo** | GPIO 34 | Entrada Analógica (ADC) |
| **LED Verde** | GPIO 26 | Saída Digital (Status Ativo) |
| **LED Vermelho** | GPIO 27 | Saída Digital (Status Alerta/Pausa) |
| **Botão Físico** | GPIO 25 | Entrada Digital com Interrupção (Fallback) |


#Tecnologias e Softwares Utilizados

* **Linguagem:** C++ 
* **Conectividade:** Protocolo Wi-Fi (Biblioteca `WiFi.h`)
* **Comunicação Web:** Cliente HTTP integrado (Biblioteca `HTTPClient.h`)
* **Banco de Dados / Backend:** Supabase (Integração via API RESTful)


# Regras de Classificação

O sistema realiza a leitura analógica (0 a 4095) e mapeia os valores para uma escala percentual de **0% a 100%**. A tomada de decisão local segue os seguintes parâmetros:

* **💧 Até 30% de umidade:** `SOLO SECO` -> Alerta de necessidade de irrigação.
* **✅ Entre 31% e 60% de umidade:** `SOLO IDEAL` -> Condições ótimas para o cultivo.
* **🌊 Acima de 60% de umidade:** `SOLO MUITO ÚMIDO` -> Alerta para evitar superidratação.

