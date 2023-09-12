# Reto 2 Sistemas IoT- Capa de Comunicación - Red

## Comparación entre tecnologías

Para la comparación se ajustó el script del nodo sensor para enviar un máximo de 1000 mensajes, ya sea de alarma o de repetición; se configuró el script para que muestre el número de mensajes y el consumo de batería, realizando un muestro cada 10 milisegundos.

```
set ant 999
atget id id
getpos2 lonSen latSen
set ite 0
battery set 100

loop
wait 10
read mens
rdata mens tipo valor
if((tipo=="hola") && (ant == 999))
   set ant valor
   data mens tipo id
   send mens * valor
end

if(tipo=="alerta")
   send mens ant
   inc ite
end

areadsensor tempSen
rdata tempSen SensTipo idSens temp
if(temp>30)
   data mens "alerta" lonSen latSen
   send mens ant
   inc ite
end

print ite
if(ite>=1000)
   stop
end

if(tipo=="stop")
   data mens "stop"
   send mens * valor
   cprint "Para sensor:"id
   wait 1000
   stop
end

battery bat 
if(bat<5)
   data mens "critico" lonSen latSen
   send mens ant
end

delay 10
```

Además de las configuraciones dadas, se establece el tiempo de flecha en 100ms y se define que la red de sensores no va a tener cambios de ubicación. 

Dados los parámetros del sistema se observa el siguiente comportamiento con cada tecnología:

### ZigBee  

Número de mensajes enviados: 1000 / Batería de nodo inferior: 63  

<image src="images/red-zigbee-01.png" width="90%" height="90%" alt="zigbee-1">

<image src="images/red-zigbee-02.png" width="90%" height="90%" alt="zigbee-2">


### WiFi

Número de mensajes enviados: 695 / Batería de nodo inferior: 0 

<image src="images/red-wifi-01.png" width="90%" height="90%" alt="wifi-1">

<image src="images/red-wifi-02.png" width="90%" height="90%" alt="wifi-2">

### LoRa

Número de mensajes enviados: 1000 / Batería de nodo inferior: 84

<image src="images/red-lora-01.png" width="90%" height="90%" alt="lora-1">

<image src="images/red-lora-02.png" width="90%" height="90%" alt="lora-2">


## Análisis de los datos obtenidos

| Tecnología | Mensajes enviados  | Consumo |
|------------|--------------------|---------|
| LoRa       | 1000               | 16%     |
| ZigBee     | 1000               | 37%     |
| WiFi       | 695                | 100%    |

* La tecnología que tiene más consumo de energía es el WiFi por lo que no alcanzó a enviar los 1000 mensajes antes de acabar completamente con la batería. El consumo fue de 0.14/mensaje. 

* La tecnología ZigBee mostró un buen desempeño logrando el envío de todos los mensajes utilizando el 37% de la batería. El consumo fue de 0,037/mensaje 

* La tecnología más eficiente en consumo de energía fue LoRa logrando enviar los 1000 mensajes utilizando tan solo el 16% de la batería. El consumo fue de 0,016/mensaje. 

## Conclusiones 

Las tecnologías usadas según la literatura tienen las siguientes características en consumo, costos, rango de alcance y tasa de transmisión: 

<image src="images/lora-and-wireless-technologies.png" width="90%" height="90%" alt="lora-and-wireless-technologies">

Efectivamente la simulación confirma lo presentado en la gráfica, siendo la tecnología LoRa, una red inalámbrica de área amplia y baja potencia (LPWAN), la que evidenció mejor eficiencia.  

Para los objetivos del proyecto, ésta es la tecnología de comunicación que más se adapta debido a que los cerros orientales son de gran extensión, por tanto, el alcance de LoRa brinda una gran ventaja en el terreno que se puede abarcar. Adicionalmente, su bajo consumo de energía y su bajo costo definitivamente lo confirman como el candidato favorito para la solución. 

Por el contrario, la tecnología WiFi tiene un alto consumo de energía y su alcance es limitado, por tanto, es más recomendado para soluciones domésticas. 