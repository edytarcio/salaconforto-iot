# Firmware da placa
 
Guia para utilizar o firmware do SalaConforto na sua placa ESP32 com sensor DHT22.
 
Antes de começar, tenha em mãos: ESP32, DHT22, protoboard, cabos jumper, cabo USB de dados e um Wi-Fi de **2,4 GHz**. O broker do projeto precisa estar sendo executado no seu computador (veja o [README](README.md#rodar)).
 
## 1. Copiar a pasta
 
1. Clone o repositório completo:
```bash
git clone https://github.com/edytarcio/salaconforto-iot.git
```
 
   Sem Git? Abra https://github.com/edytarcio/salaconforto-iot, clique em **Code → Download ZIP** e extraia o arquivo.
 
2. Entre na pasta do projeto:
```bash
cd salaconforto-iot
```
 
3. Copie a pasta `firmware` para outro local, como a pasta de sketches da Arduino IDE (normalmente `Documentos/Arduino`):
```bash
# Linux / macOS
cp -r firmware ~/Documents/Arduino/firmware
 
# Windows (PowerShell)
Copy-Item -Recurse firmware "$HOME\Documents\Arduino\firmware"
```
 
   Também funciona copiar e colar pelo explorador de arquivos.
 
4. Use sempre a **cópia** nos próximos passos. Assim o repositório continua intacto e as suas senhas ficam só na cópia.
> Não renomeie a pasta `firmware`: ela precisa ter o mesmo nome do arquivo `firmware.ino`.
 
## 2. Ligar o sensor
 
| DHT22 | ESP32 |
|---|---|
| VCC (+) | 3V3 |
| GND (-) | GND |
| DATA | GPIO 17 |
 
Se o seu DHT22 for avulso (4 pinos), coloque um resistor de 10 kΩ entre DATA e 3V3.
 
O GPIO 17 é o pino padrão do projeto. Se na sua placa o sensor estiver em outro pino, abra o `firmware.ino` e troque o número em `#define DHTPIN 17` pelo pino que você usou.
 
## 3. Configurar o Wi-Fi
 
Dentro da pasta `firmware/`, copie o modelo de credenciais:
 
```bash
cp credenciais.exemplo.h credenciais.h
```
 
No Windows (PowerShell): `Copy-Item credenciais.exemplo.h credenciais.h`
 
Abra o `credenciais.h` e preencha:
 
```cpp
#define WIFI_SSID "nome-da-rede"
#define WIFI_PASSWORD "senha-da-rede"
#define MQTT_HOST "192.168.0.10"   // IP do computador com o broker
#define SALA_ID "1"                // um número diferente por placa (o 2 é do simulador)
```
 
Para descobrir o IP do computador: `ipconfig` (Windows) ou `hostname -I` (Linux).
 
## 4. Instalar o necessário
 
O firmware usa as dependências abaixo. As três primeiras você precisa instalar; as demais já vêm com o pacote da placa.
 
| Dependência | Para que serve | Arquivo que usa | Instalação |
|---|---|---|---|
| Suporte ao ESP32 (`esp32:esp32`, da Espressif) | Compilar e gravar na placa; já inclui `WiFi.h` e o relógio NTP | todos | Boards Manager |
| DHT sensor library (Adafruit) | Ler temperatura e umidade do DHT22 (`DHT.h`) | `sensor.h`, `sensor.cpp` | Library Manager |
| Adafruit Unified Sensor | Exigida pela DHT sensor library | (indireta) | Library Manager |
| MQTT (arduino-mqtt, de Joel Gaehwiler) | Publicar no broker com QoS 1 (`MQTT.h`) | `mqttpublicador.cpp` | Library Manager |
| `WiFi.h`, `Arduino.h` | Conexão Wi-Fi e funções básicas | `wificonector.cpp`, `mqttpublicador.cpp` | Já vêm com o pacote ESP32 |
 
Na **Arduino IDE**:
 
1. Em *File → Preferences*, adicione em *Additional boards manager URLs*:
   `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
2. Em *Boards Manager*, instale **esp32** (Espressif).
3. Em *Manage Libraries*, instale **DHT sensor library**, **Adafruit Unified Sensor** e **MQTT** (de Joel Gaehwiler).
Ou pelo **arduino-cli**:
 
```bash
arduino-cli config add board_manager.additional_urls https://espressif.github.io/arduino-esp32/package_esp32_index.json
arduino-cli core update-index && arduino-cli core install esp32:esp32
arduino-cli lib install "DHT sensor library" "Adafruit Unified Sensor" "MQTT"
```
 
## 5. Gravar na placa
 
**Arduino IDE:** abra `firmware/firmware.ino`, escolha a placa **ESP32 Dev Module**, escolha a porta e clique em **Upload**. Se travar em `Connecting....`, segure o botão **BOOT** da placa.
 
**arduino-cli** (troque a porta pela sua; no Windows, algo como `COM3`):
 
```bash
arduino-cli compile --fqbn esp32:esp32:esp32 firmware
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32 firmware
```
 
## 6. Conferir
 
Abra o Monitor Serial em **115200**. Você deve ver a placa conectar ao Wi-Fi, ao broker e publicar cada leitura. No painel (http://localhost:8501), escolha a sala que você definiu em `SALA_ID`.
 
## Problemas comuns
 
| Problema | O que fazer |
|---|---|
| `credenciais.h` não encontrado | Crie o arquivo a partir do modelo (passo 3) |
| `DHT.h` ou `MQTT.h` não encontrado, ou erros na `MQTT` | Instale as bibliotecas do passo 4; use a **MQTT** de Joel Gaehwiler, não a `PubSubClient` |
| A IDE não encontra o nome da pasta | A pasta precisa se chamar `firmware`, igual ao `firmware.ino` |
| Placa ESP32 não aparece na IDE | Adicione a URL do passo 4 e instale o pacote **esp32** |
| Porta da placa não aparece | Troque o cabo (precisa ser de dados) ou instale o driver da placa (CP210x ou CH340) |
| Trava em `Connecting....` ao gravar | Segure o botão **BOOT** durante o envio |
| `Falha na leitura do sensor` | Confira a fiação e se o pino do sensor é o mesmo do `DHTPIN` no `firmware.ino` (padrão: 17) |
| Sensor ligado em outro pino | Altere `#define DHTPIN 17` no `firmware.ino` para o seu pino e grave de novo |
| `Falha ao conectar ao Wi-Fi` | Confira nome e senha; use rede de 2,4 GHz |
| `Falha ao conectar ao broker` | Confira o `MQTT_HOST` (o IP do computador), se o broker está ligado e se o firewall libera a porta 1883 |
| A placa publica, mas o painel está vazio | Confira se o gravador está rodando e se a sala do painel é a mesma do `SALA_ID` |
| Parou de funcionar depois de trocar de rede | O IP do computador mudou: atualize o `MQTT_HOST` e grave de novo |
 
Se ainda houver erro, rode `python verificar.py` na pasta do projeto: ele indica o que está faltando.
 