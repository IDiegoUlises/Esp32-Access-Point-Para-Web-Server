# Esp32 Access Point Para Web Server

### Funcionamiento
![](https://github.com/IDiegoUlises/Esp32-Access-Point-Para-Web-Server/blob/main/Images/Esp32-AP-Web-Server.gif)
* Se crea una red Wifi
* Se crea una web server dentro de la red Wifi 
* Se realiza una conexion mediante el navegador

### Codigo
```c++
//Libreria Wifi
#include <WiFi.h>

//La red Wifi que se creara
const char* ssid     = "ESP32-Wifi";
const char* password = "123456789";

//Establece la web server en el puerto 80
WiFiServer server(80);

//Variable para almacenar la solicitud HTTP
String header;

//Variable para almacenar el estado
String output26State = "off";
String output27State = "off";

//Asignar variables de salida
const int output26 = 26;
const int output27 = 27;

void setup()
{
  //Inicia el puerto serial
  Serial.begin(9600);

  //Se asignan las variables como salida
  pinMode(output26, OUTPUT);
  pinMode(output27, OUTPUT);

  //Imprime en el puerto serial
  Serial.print("Setting AP (Access Point)…");

  //Conexion de la red Wifi
  WiFi.softAP(ssid, password); //Asigne la contrasena como valor null si es una red abierta

  //Obtiene la direccion IP
  IPAddress IP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(IP);

  //Inicia el servidor
  server.begin();
}

void loop() {
  WiFiClient client = server.available();   //Escucha los clientes

  if (client) {                             // Si se conecta un nuevo cliente
    Serial.println("Nuevo Cliente.");       //Imprime un mensaje de salida en el puerto serial
    String currentLine = "";                //Crea una cadena que almacena los datos del cliente
    while (client.connected()) {            //Bucle while mientras el cliente esta conectado
      if (client.available()) {             //Si hay bytes para leer del cliente,
        char c = client.read();             //Lee un byte, despues
        Serial.write(c);                    //Se imprime en el puerto serial
        header += c;
        if (c == '\n') {                    // Si el byte es un caracter de salto de linea
          //Si la nueva linea está en blanco significa que es el fin del
          //HTTP request del cliente, entonces respondemos:
          if (currentLine.length() == 0) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();

            //Enciende y apaga los GPIO
            if (header.indexOf("GET /26/on") >= 0) {
              Serial.println("GPIO 26 on");
              output26State = "on";
              digitalWrite(output26, HIGH);
            } else if (header.indexOf("GET /26/off") >= 0) {
              Serial.println("GPIO 26 off");
              output26State = "off";
              digitalWrite(output26, LOW);
            } else if (header.indexOf("GET /27/on") >= 0) {
              Serial.println("GPIO 27 on");
              output27State = "on";
              digitalWrite(output27, HIGH);
            } else if (header.indexOf("GET /27/off") >= 0) {
              Serial.println("GPIO 27 off");
              output27State = "off";
              digitalWrite(output27, LOW);
            }

            //Mostrar la pagina HTML
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<link rel=\"icon\" href=\"data:,\">");
            // CSS para dar estilo a los botones de on/off
            //Cambiando los atributos del boton
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");

            //Encabezado de pagina web
            client.println("<body><h1>ESP32 Web Server</h1>");

            //Muestra el estado actual y los botones ON/OFF para gpio 26
            client.println("<p>GPIO 26 - State " + output26State + "</p>");

            //Si output26State esta apagado muestra el boton ON
            if (output26State == "off")
            {
              client.println("<p><a href=\"/26/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/26/off\"><button class=\"button button2\">OFF</button></a></p>");
            }

            //Muestra el estado actual y los botones ON/OFF para gpio 27
            client.println("<p>GPIO 27 - State " + output27State + "</p>");
            
            //Si el estado de salida 27 esta apagado muestra el boton encendido
            if (output27State == "off") {
              client.println("<p><a href=\"/27/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/27/off\"><button class=\"button button2\">OFF</button></a></p>");
            }
            client.println("</body></html>");

            //La respuesta HTTP termina con una linea en blanco
            client.println();
            break; //Para salir del bucle while del loop()
          } else { //Si tenemos una nueva linea limpiamos currentLine
            currentLine = "";
          }
        } else if (c != '\r') {  //Si la variable c es distinto al caracter de retorno de carro
          currentLine += c;      //Lo agrega al final de currentLine
        }
      }
    }
    //Limpia la variable header
    header = "";
    
    //Cierra la conexion
    client.stop();
    Serial.println("Cliente desconectado.");
    Serial.println("");
  }
}
```
### Debug
<img src="https://github.com/IDiegoUlises/Esp32-Access-Point-Para-Web-Server/blob/main/Images/Puerto-Serial.png" />
