# PECWIZARDS
#include <Adafruit_BMP085.h>
#include <ESP8266WiFi.h>
#include "INDEX.h"
Adafruit_BMP085 bmp;
double T,P;
int smoke=0, flame=0,help=0;
boolean value=0;
String message="";
void lookup(){
  smoke = analogRead(A0);
  flame = !digitalRead(2);
  help = digitalRead(14);
  T=bmp.readTemperature();
  P=bmp.readPressure();
   if(flame){    //if flame
    digitalWrite(12,HIGH);
    digitalWrite(13,HIGH);
    digitalWrite(15,LOW);
    digitalWrite(16,LOW);
  }else if(smoke<350&&(!flame)){ //if only smoke
     digitalWrite(12,HIGH);
    digitalWrite(13,HIGH);
      digitalWrite(15,HIGH);
      digitalWrite(16,LOW);
  }else {
     digitalWrite(12,LOW);
    digitalWrite(13,LOW);
    digitalWrite(15,LOW);
   
    
  }
  if(help==HIGH||flame||smoke<350){
    digitalWrite(12,HIGH);
  }
    else{
      digitalWrite(12,LOW);
    }
}
const char* ssid = "Mangatayaru";
const char* password = "12345678";

// Create an instance of the server
// specify the port to listen on as an argument
WiFiServer server(80);

void setup() {
  Serial.begin(115200);
  delay(10);
  bmp.begin();


pinMode(2,INPUT);
pinMode(14,INPUT);
pinMode(12,OUTPUT);
pinMode(13,OUTPUT);
pinMode(15,OUTPUT);
pinMode(16,OUTPUT);
  
  // Connect to WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    lookup();
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  
  // Start the server
  server.begin();
  Serial.println("Server started");

  // Print the IP address
  Serial.println(WiFi.localIP());
}

void loop() {
  // Check if a client has connected
  WiFiClient client = server.available();
  if (!client) {
    lookup();
    return;
  }
  
  // Wait until the client sends some data
  Serial.println("new client");
  while(!client.available()){
    delay(1);
  }
  lookup();
  // Read the first line of the request
  String command1 = client.readStringUntil('/');
  String command = client.readStringUntil('/');
  Serial.println(command1);
  Serial.println(command);

if (command == "status") {
  client.print(F("status#"));
  message+="smoke=";
      message+=(String)smoke;
      message+="#";
      message+="flame=";
      message+=(String)flame;
      message+="#";
       message+="help=";
       message+=(String)help;
       message+="#";
       message+="alarm=";
       message+=(String)digitalRead(12);
       message+="#";
       message+="sprinkler=";
       message+=(String)digitalRead(13);
       message+="#";
       message+="fan=";
       message+=(String)digitalRead(15);
        message+="#";
       message+="power=";
       message+=(String)digitalRead(16);
       message+="#";
       message+="temperature=";
       message+=(String)T;
       message+="#";
       message+="pressure=";
       message+=(String)P;
       client.print(message);
       message="";
}else if(command=="power"){
  value= digitalRead(16);
  digitalWrite(16,!value);
  client.print(F("status#"));
  client.print(F("power="));
  client.print(!value);
  
}

else {  // Prepare the response
  String s = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n";
  s += file1;  
  client.flush();
  // Send the response to the client
while(s.length()>2000)
{
  String dummy = s.substring(0,2000);
  client.print(dummy);
  s.replace(dummy," ");
}
  
  client.print(s);
  delay(1);
  Serial.println("Client disconnected");

  // The client will actually be disconnected 
  // when the function returns and 'client' object is destroyed
}
}
