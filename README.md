# Reto 4 Sistemas IoT- Capa de Datos

## Modificaciones de código

Para medir las diferencias entre las implementaciones de la capa de datos con Postgres y TimeScale, el equipo decide ajustar la función get_map_json para incluir el parámetro ciudad en la consulta y validar el comportamiento en cada motor. 

Para ello, inicialmente se adiciona la nueva ruta en el archivo urls.py, donde se incluye el nuevo parámetro city:  

```
urlpatterns = [
    path('', DashboardView.as_view(), name='dashboard'),
    path('historical/', HistoricalView.as_view(), name='historical'),
    path('rema/', RemaView.as_view(), name='rema'),
    path('rema/<str:measure>', RemaView.as_view(), name='rema'),
    path("mapJson/", get_map_json, name="mapJson"),
    path("mapJson/<str:measure>", get_map_json, name="mapJson"),
    path("mapJson/<str:measure>/<str:city>", get_map_json, name="mapJson"),
    path('login/', LoginView.as_view(), name='login'),
    path('logout/', LogoutView.as_view(), name='logout'),
    path('historical/data',
         download_csv_data, name='historical-data'),
]
```

Y posteriormente, se ajusta la lógica del nuevo parámetro city en la función get_map_json en el archivo views.py, filtrando las locations por la ciudad requerida en la consulta: 

```
def get_map_json(request, **kwargs):
    data_result = {}

    measureParam = kwargs.get("measure", None)
    selectedMeasure = None
    measurements = Measurement.objects.all()

    if measureParam != None:
        selectedMeasure = Measurement.objects.filter(name=measureParam)[0]
    elif measurements.count() > 0:
        selectedMeasure = measurements[0]

    # 2023-09-11 add city param
    cityParam = kwargs.get("city", None)
    selectedCity = None
    cities = City.objects.all()

    if cityParam != None:
        selectedCity = City.objects.filter(name=cityParam)[0]
    elif cities.count() > 0:
        selectedCity = cities[0]

    #locations = Location.objects.all()
    locations = Location.objects.filter(city__name=selectedCity.name)
```

Es importante mencionar que este ajuste se aplicó en los archivos realtimeGraph/urls.py y realtimeGraph/views.py, tanto para en la máquina virtual con postgres como en la timescale.

## Comparación de resultados 

Para ejecutar las pruebas de los ajustes, inicialmente se configura en Jmeter la nueva variable consulta_url_reto, apuntando al endpoint mapJson, y pasando los parámetros measure y city con los valores Temperatura y Ciudad1:  

```
/mapJson/Temperatura/Ciudad1?from=1625115600000&to=1627793999999  
```
Posteriormente, se duplicaron los thread groups de pruebas, tanto para postgres como para timescale, apuntando a la variable recién creada ${consulta_url_reto}. 

Se procede entonces a ejecutar las pruebas hacia la instancia postgres: 

<image src="images/jmeter-postgres-query-reto4-01.jpg" width="100%" height="100%" alt="postgres-60">

Y luego hacia la instancia timescale: 

<image src="images/jmeter-timescale-query-reto4-01.jpg" width="100%" height="100%" alt="timescale-60">

La diferencia de los resultados es sorprendente, mientras que, para 60 usuarios por segundo, la respuesta promedio de la aplicación con postgres es de aproximadamente 36 segundos, en el caso de la aplicación com timescale no supera los 2 segundos. 

Adicionalmente, el equipo decide ejecutar nuevamente las pruebas, esta vez incrementando en 60 la cantidad de usuarios, con el objetivo de validar en qué momento se generan errores  

En la prueba de 120 usuarios por segundo para la aplicación postgres ya se evidencia un 20% de errores: 

<image src="images/jmeter-postgres-query-reto4-02.jpg" width="100%" height="100%" alt="postgres-120">

Y en la prueba de 180 usuarios por segundo, aunque no desmejora el tiempo promedio de respuesta, los errores se incrementan considerablemente a un 46,67%: 

<image src="images/jmeter-postgres-query-reto4-03.jpg" width="100%" height="100%" alt="postgres-180">

Ejecutando la prueba de 120 usuarios por segundo en la aplicación con timescale, se duplica la respuesta promedio, pero se mantiene el % de error en 0.

Para prueba de 180 usuarios por segundo en la aplicación con timescale, se sigue manteniendo el % de error en 0.

Finalmente se ejecuta una prueba de 600 usuarios por segundo en la aplicación con timescale, donde ya se evidencia un incremento en el tiempo de respuesta promedio a 15.5 segundos aproximadamente y un % de error del 2.17:

<image src="images/jmeter-timescale-query-reto4-04.jpg" width="100%" height="100%" alt="timescale-600">

### Consolidado de iteraciones

|            | Iteración 1 | Iteración 2 | Iteración 3 | Iteración 4 |
| ---------- | ----------- | ----------- | ----------- | ----------- |
| Tecnología | Timescale   | Postgres    | Timescale   | Postgres    | Timescale | Postgres | Timescale |
| Samples    | 61          | 60          | 120         | 120         | 180 | 180 | 600 |
| Average    | 1867        | 36050       | 3767        | 53402       | 5298 | 55223 | 15591 |
| Min        | 0           | 0           | 0           | 0           | 0 | 0 | 0 |
| Max        | 2297        | 38212       | 5424        | 64919       | 8167 | 71354 | 27324 |
| %Error     | 0           | 0           | 0           | 20          | 0 | 46,67 | 2,17 |

## Conclusión 

Las bases de datos de series de tiempo sobresalen en términos de rendimiento en comparación con las bases de datos relacionales por varias razones claves.  

En una base de datos de series de tiempo, una hipertabla se convierte en el eje central de almacenamiento. Esta hipertabla simplifica la gestión de datos temporales al permitir la segmentación automática de la información en "chunks" o segmentos de tiempo más pequeños. Cada uno de estos chunks almacena datos correspondientes a un período específico, lo que facilita enormemente la organización y el acceso a los datos. Al fragmentar los datos de esta manera, se pueden realizar operaciones de lectura y escritura de manera paralela en cada segmento de tiempo, lo que mejora significativamente el rendimiento, especialmente cuando se trabaja con grandes volúmenes de datos temporales. 

El laboratorio previo al reto nos propone un código en el cual se ejecuta un procesamiento de datos antes de su almacenamiento en TimescaleDB, esto empieza con la ejecución de una consulta que recupera el registro asociado a un tiempo base y continua con el cálculo de los valores de "max_value" y "avg_value".  

Esta práctica tiene dos beneficios notables: en primer lugar, reduce la cantidad de datos que deben consultarse al agrupar varios valores en un solo objeto; en segundo lugar, disminuye la carga computacional al obtener valores previamente calculados para un conjunto de datos, lo que resulta en una reducción de las iteraciones requeridas para su obtención. 

En resumen, las bases de datos de series de tiempo superan a las bases de datos relacionales en términos de rendimiento gracias a su eficiente estructura de hipertabla y el procesamiento previo de datos. Esto simplifica la gestión de datos temporales, mejora la eficiencia de las consultas y reduce la carga computacional, lo que las hace ideales para aplicaciones con grandes volúmenes de datos temporales.
