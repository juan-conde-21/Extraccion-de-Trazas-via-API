# Extraccion-de-Trazas-via-API

# Prequisito generar el Api token en Instana.

1. Dirigirse a la seccion de Settings y ubicar la opcion de Api Token
 
   ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/af39a48d-f1aa-4bce-94f2-7052029f23e8)

2. Colocar el nombre al api token y seleccionar Save para guardar. (No es necesario agregar permisos adicionales)

   ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/a369d65f-e957-4398-a6f4-b032ad6ccd13)


## Extraer todas las trazas con error para un servicio especifico

  URL endpoint Instana
  
  https://{tenant-id}.instana.io/api/application-monitoring/analyze/traces

  **Timeframe**
  
  **to**: Indicar la fecha hasta cuando se realiza la consulta en formato epoch timestamp (ejemplo la fecha 15-05-2024 16:00:00 se traduce como 1715806800000)

  **windowsSize**: Indicar el tiempo de consulta en milisegundos (ejemplo el rango de una hora es 3600000 milisegundos, este valor se restara del campo **"to"** definido en el punto anterior, obteniendo como resultado las trazas desde 15-05-2024 15:00:00 hasta 15-05-2024 16:00:00)

  **tagFilterExpression**
  Se utiliza para colocar el filtrado de entidades, para este ejemplo usamos el nombre del servicio "bstrfinmediatas" y las llamadas con error "call.erroneous"

  **Request Body de ejemplo**

    {
        "timeFrame": {
            "to": 1715806800000,
            "windowSize": 3600000,
            "autoRefresh": false
        },
        "tagFilterExpression": {
            "type": "EXPRESSION",
            "logicalOperator": "AND",
            "elements": [
                {
                    "type": "TAG_FILTER",
                    "name": "service.name",
                    "operator": "EQUALS",
                    "entity": "DESTINATION",
                    "value": "bstrfinmediatas"
                },
                {
                    "type": "TAG_FILTER",
                    "name": "call.erroneous",
                    "operator": "EQUALS",
                    "entity": "NOT_APPLICABLE",
                    "value": true
                }
            ]
        },
        "metrics": [
            {
                "metric": "traces",
                "aggregation": "SUM"
            },
            {
                "metric": "errors",
                "aggregation": "MEAN"
            },
            {
                "metric": "latency",
                "aggregation": "MEAN"
            }
        ],
        "order": {
            "by": "timestamp",
            "direction": "DESC"
        },
        "group": {},
        "includeInternal": false,
        "includeSynthetic": false
    }


  **Ejemplo de ejecucion en postman**

  Ubicar el api de analisis de trazas

  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/70b047f9-50a0-4640-9be9-09baef036844)

  Ejecutar la consulta
  
  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/5f4c99db-6acd-442b-be1a-96edcd419052)

  **Ejemplo de ejecucion comando curl**

  Reemplazar los valores de {tenant-id} y {apiToken} con los valores correspondientes a su tenant Instana, modificar el nombre del servicio de acuerdo con su criterio de busqueda.

    curl -XPOST https://{tenant-id}.instana.io/api/application-monitoring/analyze/traces -H "Content-Type: application/json" -H "authorization: apiToken {apiToken}" -d '{"timeFrame":{"to":1715806800000,"windowSize":3600000,"focusedMoment":1715806800000,"autoRefresh":false},"tagFilterExpression":{"type":"EXPRESSION","logicalOperator":"AND","elements":[{"type":"TAG_FILTER","name":"service.name","operator":"EQUALS","entity":"DESTINATION","value":"bstransferencias"},{"type":"TAG_FILTER","name":"call.erroneous","operator":"EQUALS","entity":"NOT_APPLICABLE","value":true}]},"metrics":[{"metric":"traces","aggregation":"SUM"},{"metric":"errors","aggregation":"MEAN"},{"metric":"latency","aggregation":"MEAN"}],"order":{"by":"timestamp","direction":"DESC"},"group":{},"includeInternal":false,"includeSynthetic":false}'

  Con los resultados obtenidos se pueden identificar los campos a nivel de los indicadores clave de la traza.

  - **startTime**: Fecha de incio en milisegundos formato epoch timestamp.  
  - **duration**: Tiempo en que se procesa la traza completa.
  - **erroneous**: Indica el estado de error de la primera llamada de la traza.

  Para calcular el tiempo de finalizacion se debe sumar startTime + duration.


## Extraer detalle de las trazas

  URL endpoint Instana
  
  https://{tenant-id}.instana.io/api/application-monitoring/v2/analyze/traces/{id}

  1. En el punto anterior se mostro como extraer la lista de trazas con errores, para consultar el detalle de cada traza se requiere el identificador del traceId.

  2. Luego de obtener el traceId procedemos con la consulta y ubicamos los id de las llamadas con error para realizar su posterior consulta.

  **Ejemplo de ejecucion en postman**

  Ubicar el api version 2 de analisis de trazas

  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/20f013b2-89b3-420f-bf0b-35431f13d099)

  Ejecutar la consulta
  
  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/2e34b46e-22be-4e29-aebd-50e1ac1853ca)

  Como se muestra la traza "c6471e92ad483b93" tiene un error en la llamada con id "c70dee7d36029987".

  **Ejemplo de ejecucion comando curl**

  Reemplazar los valores de {tenant-id} y {apiToken} con los valores correspondientes a su tenant Instana, modificar el valor de {traceId} de acuerdo con la traza a consultar.

    curl https://{tenant-id}.instana.io/api/application-monitoring/v2/analyze/traces/{traceId} -H "Content-Type: application/json" -H "authorization: apiToken {apiToken}"

  Con los resultados obtenidos se pueden identificar los campos a nivel de los indicadores clave de la traza y llamada.

  - **timestamp**: Fecha de incio en milisegundos formato epoch timestamp.
  - **duration**: Tiempo en que se procesa la traza completa.\n
  - **errorCount**: Indica la cantidad de errores identificados en la llamada.

  Para calcular el tiempo de finalizacion se debe sumar timestamp + duration.


## Extraer detalle de las llamadas con error pertenecientes a una traza

  URL endpoint Instana
  
  https://{tenant-id}.instana.io/api/application-monitoring/v2/analyze/traces/{traceId}/calls/{callId}/details

  Ejecutar la consulta con los datos del traceId y el id de la llamada obtenidos en los pasos anteriores

  **Ejemplo de ejecucion en postman**

  Ubicar el api version 2 de analisis de llamadas
  
  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/b43fca53-507e-407f-9e92-5ca8a9d6b7dd)

  Ejecutar la consulta
  
  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/3bac49d2-2191-409d-ac23-9ad8f905dd38)

  **Ejemplo de ejecucion comando curl**

  Reemplazar los valores de {tenant-id} y {apiToken} con los valores correspondientes a su tenant Instana, modificar los valores de {traceId} y {callId} de acuerdo con la traza y llamada a consultar.

    curl https://{tenant-id}.instana.io/api/application-monitoring/v2/analyze/traces/{traceId}/calls/{callId}/details -H "Content-Type: application/json" -H "authorization: apiToken {apiToken}"

  Con los resultados obtenidos se pueden identificar los campos a nivel de los indicadores clave de la traza y llamada.

  - **start**: Fecha de incio en milisegundos formato epoch timestamp.
  - **duration**: Tiempo en que se procesa la traza completa.
  - **errorCount**: Indica la cantidad de errores identificados en la llamada.
  - **spans.data**: Contiene el detalle a nivel de status code para cada span (ejecución de linea de codigo).
  - **logs**: Detalle de los logs de errores identificados en la llamada.

  Para calcular el tiempo de finalizacion se debe sumar timestamp + duration.


## Extraer Llamadas Fallidas agrupado por endpoint a nivel de cada servicio

  URL endpoint Instana
  
  https://{tenant-id}.instana.io/api/application-monitoring/analyze/call-groups?fillTimeSeries=true

  **Timeframe**
  
  **to**: Indicar la fecha hasta cuando se realiza la consulta en formato epoch timestamp (ejemplo la fecha 15-05-2024 16:00:00 se traduce como 1715806800000)

  **windowsSize**: Indicar el tiempo de consulta en milisegundos (ejemplo el rango de una hora es 3600000 milisegundos, este valor se restara del campo **"to"** definido en el punto anterior, obteniendo como resultado las trazas desde 15-05-2024 15:00:00 hasta 15-05-2024 16:00:00)

  **tagFilterExpression**
  Se utiliza para colocar el filtrado de entidades, para este ejemplo usamos el nombre del servicio "bstrfinmediatas" y las llamadas con error "call.erroneous"

  **Request Body de ejemplo**

    {
        "timeFrame": {
            "to": 1720587600000,
            "windowSize": 86400000,
            "autoRefresh": false
        },
        "tagFilterExpression": {
            "type": "EXPRESSION",
            "logicalOperator": "AND",
            "elements": [
                {
                    "type": "TAG_FILTER",
                    "name": "service.name",
                    "operator": "EQUALS",
                    "entity": "DESTINATION",
                    "value": "bstrfinmediatas"
                },
                {
                    "type": "TAG_FILTER",
                    "name": "call.erroneous",
                    "operator": "EQUALS",
                    "entity": "NOT_APPLICABLE",
                    "value": true
                }
            ]
        },
        "metrics": [
            {
                "metric": "calls",
                "aggregation": "SUM"
            },
            {
                "metric": "errors",
                "aggregation": "MEAN"
            },
            {
                "metric": "latency",
                "aggregation": "MEAN"
            }
        ],
        "order": {
            "by": "calls",
            "direction": "DESC"
        },
        "group": {
            "groupbyTag": "endpoint.name",
            "groupbyTagEntity": "DESTINATION"
        },
        "includeInternal": false,
        "includeSynthetic": false
    }

    **Ejemplo de ejecucion en postman**

  Ubicar el api de analisis de trazas

  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/d8ebdccb-a19a-466c-9db2-6ab1a0aed402)

  Ejecutar la consulta
  
  ![image](https://github.com/juan-conde-21/Extraccion-de-Trazas-via-API/assets/13276404/ee51ac63-c9e7-42aa-8d7a-9cbc3ee403bb)


  **Ejemplo de ejecucion comando curl**

  Reemplazar los valores de {tenant-id} y {apiToken} con los valores correspondientes a su tenant Instana, modificar el nombre del servicio de acuerdo con su criterio de busqueda.

    curl -XPOST https://{tenant-id}.instana.io/api/application-monitoring/analyze/call-groups?fillTimeSeries=true -H "Content-Type: application/json" -H "authorization: apiToken {apiToken}" -d '{"timeFrame":{"to":1720587600000,"windowSize":86400000,"autoRefresh":false},"tagFilterExpression":{"type":"EXPRESSION","logicalOperator":"AND","elements":[{"type":"TAG_FILTER","name":"service.name","operator":"EQUALS","entity":"DESTINATION","value":"jocuentaahorros"},{"type":"TAG_FILTER","name":"call.erroneous","operator":"EQUALS","entity":"NOT_APPLICABLE","value":true}]},"metrics":[{"metric":"calls","aggregation":"SUM"},{"metric":"errors","aggregation":"MEAN"},{"metric":"latency","aggregation":"MEAN"}],"order":{"by":"calls","direction":"DESC"},"group":{"groupbyTag":"endpoint.name","groupbyTagEntity":"DESTINATION"},"includeInternal":false,"includeSynthetic":false}'

  Con los resultados obtenidos se pueden identificar los campos a nivel de los indicadores clave de la traza.

  - **name**: Nombre del endpoint donde se presentaron los errores.
  - **timestamp**: Fecha de incio en milisegundos formato epoch timestamp.
  - **errors.mean**: Indica el estado de error de la primera llamada de la traza.
  - **calls.sum**: Cantidad de errores para el endpoint.
  - **latency.mean**: Promedio de latencia para el endpoint en milisegundos





