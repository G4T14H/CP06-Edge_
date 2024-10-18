# Documentação do Projeto FIWARE com Postman e Wokwi

## Integrantes do Grupo
- Gustavo Alves - RM 557876
- Gabriel Galerani - RM 557421
- Gabriel Dias - RM 556830
- Pedro Paulo - RM 554880
- Pedro Pawlik - RM 556968

---

## Índice
1. [Pré-requisitos](#pré-requisitos)
2. [Passo a Passo](#passo-a-passo)
   - [Passo 1: Download do Postman](#passo-1-download-do-postman)
   - [Passo 2: Provisionamento do Dispositivo](#passo-2-provisionamento-do-dispositivo)
   - [Passo 3: Registro do Dispositivo](#passo-3-registro-do-dispositivo)
   - [Passo 4: Verificação e Deleção](#passo-4-verificação-e-deleção)
3. [Configuração do Código no Wokwi](#configuração-do-código-no-wokwi)
4. [Exibição dos Dados no MyMQTT](#exibição-dos-dados-no-mymqtt)
5. [Considerações Finais](#considerações-finais)

---

## Pré-requisitos
- Conta no GitHub
- Postman instalado
- Acesso à internet
- Ambiente de desenvolvimento Wokwi configurado
- Aplicativo MyMQTT instalado em um dispositivo móvel

---

## Passo a Passo

### Passo 1: Download do Postman
1. Acesse o repositório do GitHub: FIWARE no GitHub.
2. Baixe a pasta do Postman.
3. Abra o Postman e verifique os "health checks" para garantir que o sistema está respondendo ao servidor.
4. Certifique-se de verificar o IP correto do servidor.

### Passo 2: Provisionamento do Dispositivo
1. No Postman, selecione a opção de **POST**.
2. Envie as informações para o servidor nas portas 40 e 41.
3. Configure o nome do dispositivo e o respectivo ID, trocando a entidade conforme definido.
4. Verifique se o protocolo e o método de transporte estão corretos.
5. Mantenha a lista de comandos e atributos inalterada.
6. Clique em **Send**. Se a resposta for `{}`, o provisionamento está correto.

### Passo 3: Registro do Dispositivo
1. Acesse a opção 4 chamada **registrar**.
2. Insira o ID configurado anteriormente.
3. Clique em **Send** novamente. A resposta correta será `1`.

### Passo 4: Verificação e Deleção
1. Para verificar a configuração, vá para a opção 5 e clique em **Send** novamente.
2. Você verá os 4 dispositivos configurados anteriormente. Verifique os atributos e outros parâmetros.
3. Para deletar tudo, vá para a opção 9, altere a aba de **HTTP** para hosp200 e envie o comando.

---

### Montagem ESP32 - DHT11/LDR
Siga a montagem do modelo abaixo:

![image](https://github.com/user-attachments/assets/ea1a8591-a8dd-4dbf-a49f-6e4bf011a795)

---

## Configuração do Código no Wokwi
Aqui está o código padrão para a configuração do dispositivo no Wokwi:

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"

// Defines para MQTT e DHT
#define TOPICO_SUBSCRIBE "/TEF/hosp200/cmd"
#define TOPICO_PUBLISH "/TEF/hosp200/attrs"
#define TOPICO_PUBLISH_2 "/TEF/hosp200/attrs/l"
#define TOPICO_PUBLISH_3 "/TEF/hosp200/attrs/h"
#define TOPICO_PUBLISH_4 "/TEF/hosp200/attrs/t"
#define ID_MQTT "fiware_200"
#define DHTTYPE DHT22
#define DHTPIN 4

DHT dht(DHTPIN, DHTTYPE);

// WiFi e MQTT
const char* SSID = "Wokwi-GUEST";
const char* PASSWORD = "";
const char* BROKER_MQTT = "46.17.108.113";
int BROKER_PORT = 1883;

// Saídas
int D4 = 2;
WiFiClient espClient;
PubSubClient MQTT(espClient);
char EstadoSaida = '0';

// Prototypes
void initSerial();
void initWiFi();
void initMQTT();
void reconnectWiFi();
void mqtt_callback(char* topic, byte* payload, unsigned int length);
void VerificaConexoesWiFIEMQTT();
void InitOutput();

void setup() {
    InitOutput();
    initSerial();
    initWiFi();
    initMQTT();
    dht.begin();
    MQTT.publish(TOPICO_PUBLISH, "s|off");
}

// (Demais funções omitidas para brevidade)

// Loop principal
void loop() {
    const int potPin = 34;
    char msgBuffer[4];
    VerificaConexoesWiFIEMQTT();

    int sensorValue = analogRead(potPin);
    float voltage = sensorValue * (3.3 / 4096.0);
    float luminosity = map(sensorValue, 0, 4095, 0, 100);

    Serial.print("Voltage: ");
    Serial.print(voltage);
    Serial.print("V - ");
    Serial.print("Luminosidade: ");
    Serial.print(luminosity);
    Serial.println("%");

    dtostrf(luminosity, 4, 1, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_2, msgBuffer);

    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature();
    if (isnan(humidity) || isnan(temperature)) {
        Serial.println(F("Falha leitura do sensor DHT-22!"));
    }

    Serial.print(F("Umidade: "));
    Serial.print(humidity);
    Serial.print(F("% Temperatura: "));
    Serial.print(temperature);
    Serial.println(F("°C "));

    dtostrf(humidity, 4, 1, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_3, msgBuffer);

    dtostrf(temperature, 4, 1, msgBuffer);
    MQTT.publish(TOPICO_PUBLISH_4, msgBuffer);

    MQTT.loop();
}
```

### Observações
- Verifique os tópicos relacionados ao objeto criado no IoT do Postman.
- Assegure-se de que o `ID_MQTT` esteja configurado corretamente.
- Conecte-se ao IP `46.17.108.113`.
- Dependendo da mensagem recebida nos tópicos, ações correspondentes serão acionadas.
- Faça leituras dos parâmetros do DHT22 e publique nos tópicos correspondentes.

---

## Exibição dos Dados no MyMQTT
Para visualizar os dados coletados pelo seu dispositivo no aplicativo MyMQTT, siga estas etapas:

1. Abra o MyMQTT em seu dispositivo móvel.
2. Adicione um novo broker com as seguintes configurações:
   - **Endereço do Broker**: 46.17.108.113
   - **Porta**: 1883
3. Conecte-se ao broker.
4. Navegue até a seção de **Subscriptions (Inscrições)**.
5. Inscreva-se nos tópicos relevantes para receber atualizações:
   - `/TEF/hosp200/cmd` (para comandos)
   - `/TEF/hosp200/attrs` (para atributos)
   - `/TEF/hosp200/attrs/l` (luminosidade)
   - `/TEF/hosp200/attrs/h` (umidade)
   - `/TEF/hosp200/attrs/t` (temperatura)

Agora, você poderá ver os dados de luminosidade, umidade e temperatura em tempo real no MyMQTT, conforme seu dispositivo publica as informações.

---

# IoT Dashboard com Streamlit e MQTT

Este projeto implementa um dashboard IoT em tempo real utilizando *Streamlit* para visualização dos dados e *MQTT* para receber informações de sensores de luminosidade, umidade e temperatura.

---

## Instalação

### Pré-requisitos
- Python 3.x
- Broker MQTT configurado (exemplo usado: 18.208.160.16)

### Dependências
Instale as bibliotecas necessárias executando o comando abaixo:

```bash
pip install streamlit paho-mqtt
```

---

## Configuração

Crie um arquivo Python chamado `dashboard.py` e cole o código abaixo:

```python
import streamlit as st
import paho.mqtt.client as mqtt

# Configurações do MQTT
MQTT_BROKER = "18.208.160.16"
MQTT_PORT = 1883
MQTT_TOPIC_LUM = "/TEF/hosp7771/attrs/l"
MQTT_TOPIC_UMID = "/TEF/hosp7771/attrs/h"
MQTT_TOPIC_TEMP = "/TEF/hosp7771/attrs/t"

# Variáveis globais para armazenar os dados
luminosidade = 0
umidade = 0
temperatura = 0

# Função de callback para o MQTT
def on_message(client, userdata, message):
    global luminosidade, umidade, temperatura
    payload = message.payload.decode()
    if message.topic == MQTT_TOPIC_LUM:
        luminosidade = float(payload)
    elif message.topic == MQTT_TOPIC_UMID:
        umidade = float(payload)
    elif message.topic == MQTT_TOPIC_TEMP:
        temperatura = float(payload)

# Configuração do cliente MQTT
client = mqtt.Client()
client.on_message = on_message
client.connect(MQTT_BROKER, MQTT_PORT, 60)
client.subscribe([(MQTT_TOPIC_LUM, 0), (MQTT_TOPIC_UMID, 0), (MQTT_TOPIC_TEMP, 0)])
client.loop_start()

# Configuração do Streamlit
st.title("Dashboard IoT")
st.header("Monitoramento de Sensores")

# Loop principal do Streamlit
while True:
    st.write(f"Luminosidade: {luminosidade}")
    st.write(f"Umidade: {umidade}")
    st.write(f"Temperatura: {temperatura}")
    st.experimental_rerun()
```

---

## Execução

Para rodar o dashboard, execute o seguinte comando no terminal:

```bash
streamlit run dashboard.py
```

O dashboard será aberto no navegador, exibindo os dados dos sensores de luminosidade, umidade e temperatura em tempo real.

---

## Configuração do Broker MQTT

O código está configurado para conectar ao broker MQTT no endereço `18.208.160.16` e assinar os seguintes tópicos:

- **Luminosidade**: `/TEF/hosp7771/attrs/l`
- **Umidade**: `/TEF/hosp7771/attrs/h`
- **Temperatura**: `/TEF/hosp7771/attrs/t`

Ajuste o broker e os tópicos conforme necessário para seu ambiente de IoT.

---

## Funcionalidades
- **MQTT**: Recebe dados de sensores por meio dos tópicos configurados.
- **Streamlit**: Exibe uma interface web com os valores dos sensores atualizados em tempo real.

---

![image](https://github.com/user-attachments/assets/988e61a1-4483-4dbf-8dcb-b326bc5cde75)             ![image](https://github.com/user-attachments/assets/1bbf0200-408e-4bfa-980b-80deda2cd46e)

---
