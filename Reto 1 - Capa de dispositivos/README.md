# Reto 1 Sistemas IoT- Capa de Dispositivos

## Extensión de la capa de dispositivo  

### Adicionar el sensor a la placa de desarrollo para capturar la nueva variable 
El circuito que se optó por usar fue el siguiente 

<image src="https://www.geekering.com/wp-content/uploads/2021/06/image-2.png" width="35%" height="35%" alt="circuito fotoresistencia" caption="Ilustración 1- ESP8266 NodeMCU – Simple LDR Lux Meter tomado de https://www.geekering.com/categories/embedded-sytems/esp8266/ricardocarreira/esp8266-nodemcu-simple-ldr-luximeter/">

<image src="../images/circuito.png" width="35%" height="35%" alt="circuito fotoresistencia-2">

### Extender el programa para adquirir los valores del nuevo sensor:

Las modificaciones al código que permitieron la obtención, transformación y transporte de la información de la nueva variable fueron: 

```
//Caracterización de los datos medidos del sensor
const long A = 500;     //Resistencia en oscuridad en KΩ
const int B = 1;        //Resistencia a la luz (10 Lux) en KΩ
const int Rc = 12;       //Resistencia calibracion en KΩ
const int LDRPin = A0;   //Pin del LDR
```
...
```
  now = time(nullptr);
  //Lee los datos del sensor
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  int V = analogRead(LDRPin);
  //Transforma la información a la notación JSON para poder enviar los datos 
  //El mensaje que se envía es de la forma {"value": x}, donde x es el valor de temperatura o humedad

  //Obtención de el valor de iluminación en lux
  int ilum = ((long)(1024-V)*A*10)/((long)B*Rc*V);

  //JSON para humedad
  String json = "{\"value\": "+ String(h) + "}";
  char payload1[json.length()+1];
  json.toCharArray(payload1,json.length()+1);
  //JSON para temperatura
  json = "{\"value\": "+ String(t) + "}";
  char payload2[json.length()+1];
  json.toCharArray(payload2,json.length()+1);
  //JSON para luminosidad
  json = "{\"value\": "+ String(ilum) + "}";
  char payload3[json.length()+1];
  json.toCharArray(payload3,json.length()+1);
```
...
```
  //Imprime en el monitor serial la información recolectada
  Serial.print(MQTT_PUB_TOPIC1);
  Serial.print(" -> ");
  Serial.println(payload1);
  Serial.print(MQTT_PUB_TOPIC2);
  Serial.print(" -> ");
  Serial.println(payload2);
  Serial.print(MQTT_PUB_TOPIC3);
  Serial.print(" -> ");
  Serial.println(payload3);
  /*Espera 5 segundos antes de volver a ejecutar la función loop*/
  delay(5000);
```

En este punto se logra evidenciar la intensidad de luz en Lux impresa en el log, por tanto, se procedió a agregar el código requerido para el envío de la variable al bróker MQTT.

### Extender el programa para transmitir a la plataforma web los valores adquiridos del nuevo sensor:

```
  //Si los valores recolectados no son indefinidos, se envían a los tópicos correspondientes
  if ( !isnan(h) && !isnan(t) ) {
    //Publica en el tópico de la humedad
    client.publish(MQTT_PUB_TOPIC1, payload1, false);
    //Publica en el tópico de la temperatura
    client.publish(MQTT_PUB_TOPIC2, payload2, false);
    //Publica en el tópico de la luminosidad
    client.publish(MQTT_PUB_TOPIC3, payload3, false);
  }
```
Previamente se logró establecer comunicación con el bróker y la transmisión de las variables humedad y temperatura. Ya preparado el dispositivo se agregó un nuevo tópico con la variable a transmitir, la ciudad origen y el usuario: ```luminosidad/bogota/c.solanor```

### Realizar la transmisión de datos incluyendo la nueva variable de monitoreo: 

Evidencia en Dashboard de las variables medidas:

* Datos en tiempo real - temperatura

<image src="../images/datos-realtime-temperatura.png" width="90%" height="90%" alt="temperatura">

* Datos en tiempo real - humedad

<image src="../images/datos-realtime-humedad.png" width="90%" height="90%" alt="temperatura">

* Datos en tiempo real - luminosidad

<image src="../images/datos-realtime-luminosidad.png" width="90%" height="90%" alt="temperatura">

Las gráficas presentan el comportamiento de las variables en el tiempo que se ejecutó el experimento. 

En el desarrollo del laboratorio el equipo se enfrentó a las siguientes dificultades: 
 
* El fotoresistor al ser genérico no está caracterizado, en consecuencia, el equipo se vio en la necesidad de hacer una caracterización por medida de variables, en la cual se trató de hallar la resistencia en luz y en oscuridad. 

* Para obtener las constantes de la resistencia en luz y oscuridad se necesitaban de equipos especializados y calibrados con los que no se contaban (luxómetro y óhmetro), por tanto, se usó una app móvil y un multímetro de bajo costo. La escogencia de estos equipos influye de manera radical en la baja precisión de los datos. 

* El desconocimiento del equipo en circuitos dificulta la obtención de una ecuación que permita a partir de la resistencia de la fotocelda obtener la variable en lux, por esto, el equipo decide utilizar la aproximación sugerida en el blog [Medir nivel de luz con Arduino y fotoresistencia LDR (GL55)](https://www.luisllamas.es/medir-nivel-luz-con-arduino-y-fotoresistencia-ldr/).
 