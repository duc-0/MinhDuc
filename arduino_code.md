#include <WiFi.h>
#include <HTTPClient.h>

// WiFi credentials
const char* ssid = "Tao Tên Đức";
const char* password = "09042003";

// IFTTT Webhook URL (KHÔNG được xuống dòng!)
const char* webhookURL = "https://maker.ifttt.com/trigger/e_stop/with/key/hGQEcG0CMNCbG-aze6lu4NxAoEUEn87EgQTFfhQnDvT";

// Chân phần cứng
#define BUTTON_PIN 18
#define LED_PIN    19

bool hasSent = false;

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);  // Nút kéo lên
  pinMode(LED_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);

  // Kết nối WiFi
  Serial.println("🔌 Đang kết nối WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\n✅ Kết nối WiFi thành công!");
  Serial.print("📡 IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);

  if (buttonState == LOW && !hasSent) {
    Serial.println("⚠️ Nút E-Stop được nhấn!");
    digitalWrite(LED_PIN, HIGH);

    if (WiFi.status() == WL_CONNECTED) {
      HTTPClient http;
      http.begin(webhookURL);
      int httpCode = http.GET();  // Gửi yêu cầu GET
      Serial.print("📤 HTTP Response Code: ");
      Serial.println(httpCode);
      http.end();
    } else {
      Serial.println("❌ Mất kết nối WiFi!");
    }

    hasSent = true;  // Đánh dấu đã gửi
  }

  if (buttonState == HIGH) {
    hasSent = false;             // Cho phép gửi lại lần sau
    digitalWrite(LED_PIN, LOW);  // Tắt LED
  }

  delay(100);  // Chống rung và tránh spam
}
