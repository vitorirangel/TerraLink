#include <WiFi.h>
#include <HTTPClient.h>

// ===== CONFIGURAÇÕES =====
const char* ssid = "sua internet";
const char* password = "sua senha";

String SUPABASE_URL = "sua URL";
String SUPABASE_KEY = "Sua chave";

// O login, senha, KEY e URL foram alterados por segurança

// ===== PINOS =====
#define SENSOR_SOLO     34
#define LED_VERDE       26
#define LED_VERMELHO    27
#define BOTAO           25

volatile bool sistemaParado = false;

// ==========================
void IRAM_ATTR interrupcaoBotao()
{
  sistemaParado = !sistemaParado;
}

// ==========================
void conectarWiFi()
{
  Serial.println("Conectando WiFi...");

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi conectado!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());
}

// ==========================
void enviarSupabase(
  int umidade,
  String classificacao,
  String wifi_status,
  String sistema_status)
{
  if (WiFi.status() != WL_CONNECTED)
    return;

  HTTPClient http;

  String endpoint =
      SUPABASE_URL +
      "/rest/v1/leituras_solo";

  http.begin(endpoint);

  http.addHeader("Content-Type", "application/json");
  http.addHeader("apikey", SUPABASE_KEY);
  http.addHeader("Authorization", "Bearer " + SUPABASE_KEY);
  http.addHeader("Prefer", "return=minimal");

  String json =
      "{"
      "\"umidade\":" + String(umidade) + ","
      "\"classificacao\":\"" + classificacao + "\","
      "\"wifi_status\":\"" + wifi_status + "\","
      "\"sistema_status\":\"" + sistema_status + "\""
      "}";

  int resposta = http.POST(json);

  Serial.print("HTTP Response: ");
  Serial.println(resposta);

  http.end();
}

// ==========================
void setup()
{
  Serial.begin(115200);

  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);

  pinMode(BOTAO, INPUT_PULLUP);

  attachInterrupt(
      digitalPinToInterrupt(BOTAO),
      interrupcaoBotao,
      FALLING);

  conectarWiFi();
}

// ==========================
void loop()
{
  String wifi_status;
  String sistema_status;
  String classificacao;

  // ===== WIFI =====
  if (WiFi.status() != WL_CONNECTED)
  {
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_VERMELHO, HIGH);

    wifi_status = "OFFLINE";
    sistema_status = "SEM WIFI";

    Serial.println("Sem conexão WiFi");

    delay(2000);
    return;
  }

  wifi_status = "ONLINE";

  // ===== BOTÃO =====
  if (sistemaParado)
  {
    digitalWrite(LED_VERDE, LOW);
    digitalWrite(LED_VERMELHO, HIGH);

    sistema_status = "PARADO";

    Serial.println("Sistema PARADO");

    enviarSupabase(
      0,
      "PARADO",
      wifi_status,
      sistema_status);

    delay(500);
    return;
  }

  // ===== SISTEMA ATIVO =====
  sistema_status = "ATIVO";

  digitalWrite(LED_VERDE, HIGH);
  digitalWrite(LED_VERMELHO, LOW);

  // ===== SENSOR =====
  int leitura = analogRead(SENSOR_SOLO);

  int umidade = map(
      leitura,
      4095,
      0,
      0,
      100);

  if (umidade < 0) umidade = 0;
  if (umidade > 100) umidade = 100;

  // ===== CLASSIFICAÇÃO =====
  if (umidade <= 30)
  {
    classificacao = "SOLO SECO";
  }
  else if (umidade <= 60)
  {
    classificacao = "SOLO IDEAL";
  }
  else
  {
    classificacao = "SOLO MUITO UMIDO";
  }

  // ===== SERIAL =====
  Serial.println("--------------------------------");

  Serial.print("Leitura ADC: ");
  Serial.println(leitura);

  Serial.print("Umidade: ");
  Serial.print(umidade);
  Serial.println("%");

  Serial.print("Condicao: ");
  Serial.println(classificacao);

  Serial.print("WiFi: ");
  Serial.println(wifi_status);

  Serial.print("Sistema: ");
  Serial.println(sistema_status);

  // ===== SUPABASE =====
  enviarSupabase(
      umidade,
      classificacao,
      wifi_status,
      sistema_status);

  delay(10000);
}
