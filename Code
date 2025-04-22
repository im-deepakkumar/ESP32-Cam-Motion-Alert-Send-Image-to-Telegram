#include "esp_camera.h"
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include "soc/soc.h"
#include "soc/rtc_cntl_reg.h"
#include "driver/ledc.h"

// Replace with your WiFi credentials
const char* ssid = "ESP32TEST";
const char* password = "12345678";

// Telegram bot details
String token = "7687894779:AAHtEtWpHdOOIrHENPjNPbRYOKdLMC7CQq4";
String chat_id = "1128667301";

// PIR motion sensor pin
int gpioPIR = 13;

// Camera pin configuration (AI Thinker module)
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27

#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5

#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

// Web stream server
#include <WebServer.h>
WebServer server(80);

void startCameraServer() {
  server.on("/", HTTP_GET, []() {
    server.sendHeader("Location", "/stream");
    server.send(302, "text/plain", "");
  });

  server.on("/stream", HTTP_GET, []() {
    WiFiClient client = server.client();
    String response = "HTTP/1.1 200 OK\r\n";
    response += "Content-Type: multipart/x-mixed-replace; boundary=frame\r\n\r\n";
    server.sendContent(response);

    while (1) {
      camera_fb_t *fb = esp_camera_fb_get();
      if (!fb) continue;

      response = "--frame\r\nContent-Type: image/jpeg\r\n\r\n";
      server.sendContent(response);
      client.write(fb->buf, fb->len);
      server.sendContent("\r\n");
      esp_camera_fb_return(fb);

      if (!client.connected()) break;
    }
  });

  server.begin();
}

void sendPhotoToTelegram() {
  const char* server = "api.telegram.org";

  camera_fb_t *fb = esp_camera_fb_get();
  if (!fb) {
    Serial.println("Camera capture failed");
    return;
  }

  WiFiClientSecure client;
  client.setInsecure(); // Accept all certs

  if (!client.connect(server, 443)) {
    Serial.println("Connection to Telegram failed");
    esp_camera_fb_return(fb);
    return;
  }

  String head = "--boundary\r\n"
                "Content-Disposition: form-data; name=\"chat_id\"\r\n\r\n" + chat_id +
                "\r\n--boundary\r\n"
                "Content-Disposition: form-data; name=\"photo\"; filename=\"image.jpg\"\r\n"
                "Content-Type: image/jpeg\r\n\r\n";
  String tail = "\r\n--boundary--\r\n";

  uint32_t totalLen = head.length() + fb->len + tail.length();

  client.println("POST /bot" + token + "/sendPhoto HTTP/1.1");
  client.println("Host: " + String(server));
  client.println("Content-Type: multipart/form-data; boundary=boundary");
  client.println("Content-Length: " + String(totalLen));
  client.println();
  client.print(head);

  for (size_t n = 0; n < fb->len; n += 1024) {
    size_t size = (n + 1024 < fb->len) ? 1024 : fb->len - n;
    client.write(fb->buf + n, size);
  }
  client.print(tail);
  esp_camera_fb_return(fb);

  while (client.connected()) {
    if (client.available()) {
      String line = client.readStringUntil('\n');
      Serial.println(line);
    } else {
      break;
    }
  }

  client.stop();
}

void setup() {
  WRITE_PERI_REG(RTC_CNTL_BROWN_OUT_REG, 0); // Disable brownout detector
  Serial.begin(115200);
  pinMode(gpioPIR, INPUT_PULLUP);

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500); Serial.print(".");
  }

  Serial.println("\n✅ WiFi connected");
  Serial.println("IP Address: " + WiFi.localIP().toString());

  // Camera config
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  if (psramFound()) {
    config.frame_size = FRAMESIZE_VGA;
    config.jpeg_quality = 10;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_QVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }

  // Init camera
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed 0x%x", err);
    ESP.restart();
  }

  startCameraServer();
  Serial.println("📷 Camera ready! Stream: http://" + WiFi.localIP().toString());
}

void loop() {
  if (digitalRead(gpioPIR) == HIGH) {
    Serial.println("🚨 Motion Detected!");
    sendPhotoToTelegram();
    delay(10000);  // Avoid spamming Telegram
  }
  delay(1000);
}
