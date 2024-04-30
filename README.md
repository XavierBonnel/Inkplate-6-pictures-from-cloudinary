# Inkplate-6-pictures-from-cloudinary

Arduino code for using on Soldered Inkplate 6 to show pictures from cloudinary folder.

```c++
#include "HTTPClient.h"
#include "Inkplate.h"
#include "WiFi.h"
#include "ArduinoJson.h" 
#include <WString.h>  // For String
#include <base64.h>
 // Include Arduino JSON library

Inkplate display(INKPLATE_3BIT); 

const char* ssid = "YOUR_WIFI_SSID";    // Your WiFi SSID
const char* password = "YOUR_WIFI_PASSWORD"; // Your WiFi password
const char* cloudName = "YOUR_CLOUDINARY_CLOUD_NAME";
const char* apiKey = "YOUR_CLOUDINARY_API_KEY";
const char* apiSecret = "YOUR_CLOUDINARY_API_SECRET"; // Be careful with your API secret!
const char* folderName = "YOUR_FOLDER_NAME";  // Name of the folder in Cloudinary

void setup() {
    Serial.begin(115200);
    display.begin();
    display.clearDisplay();
    display.display();

    // Connect to WiFi
    WiFi.begin(ssid, password);
    Serial.println("Connecting to WiFi...");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("Connected to WiFi!");

    // Configure the API request to Cloudinary
    String url = String("https://api.cloudinary.com/v1_1/") + cloudName + "/resources/image";
    String queryParams = "?type=upload&prefix=" + String(folderName) + "/"; // Correct concatenation

    HTTPClient http;
    http.begin(url + queryParams);
    http.setAuthorization(apiKey, apiSecret);  // Basic Auth

    int httpCode = http.GET();

    if (httpCode == 200) {
        String payload = http.getString();
        DynamicJsonDocument doc(8192);
        deserializeJson(doc, payload);

        JsonArray resources = doc["resources"];

        while (true) {  // Loop to continuously cycle through images
            for (JsonObject res : resources) {
                String imageUrl = res["secure_url"].as<String>();  // Use secure_url for HTTPS
                if (!display.drawImage(imageUrl.c_str(), 0, 0, true)) {
                    Serial.println("Failed to load image: " + imageUrl);
                }
                display.display();
                delay(5000);  // Display each image for 5 seconds
            }
        }
    } else {
        Serial.println("Failed to fetch images, HTTP code: " + String(httpCode));
    }
    http.end();
    WiFi.disconnect(true);
    WiFi.mode(WIFI_OFF);
}

void loop() {
    // Nothing needed here since the slideshow runs in setup
}
```
