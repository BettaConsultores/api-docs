# API de Integración Externa

Esta documentación está dirigida a integradores externos que consumirán la API para consultar ciudadanos, consultar catálogos, administrar contactos y registrar gestiones. 

Para solicitar acceso envia un correo a [jonatan@betta.com.mx](mailto:jonatan@betta.com.mx).

Jonatan dos Santos

## Consideraciones generales

- Todos los endpoints descritos a continuación forman parte de la API de integración externa.
- Los endpoints que reciben datos utilizan formato JSON.
- La autenticación se realiza mediante el mecanismo configurado para su integración.
- En las respuestas se incluye un objeto JSON con el resultado de la operación.
- El flujo recomendado para identificar al solicitante es:
  1. Buscar primero al ciudadano en la base de datos.
  2. Si no existe, consultar o crear un contacto.
  3. Registrar la gestión indicando el `solicitante_tipo` correspondiente (`db` o `contacto`).

---

# 1. Consultar ciudadano

## Ruta
`POST /api/ciudadano/consultar`

## Descripción
Permite consultar un ciudadano existente en la base de datos utilizando uno o más filtros de búsqueda.

## Campos de envío
Puede enviar uno o varios de los siguientes campos:

| Campo | Tipo | Descripción |
|---|---|---|
| `telefono` | string | Teléfono del ciudadano. |
| `curp` | string | CURP del ciudadano. |
| `nombre` | string | Nombre del ciudadano. |
| `apellidoPaterno` | string | Apellido paterno del ciudadano. |
| `apellidoMaterno` | string | Apellido materno del ciudadano. |

## Ejemplo de envío
```json
{
  "curp": "ROSC601010HMSCLR09"
}
```

## Respuesta exitosa
```json
{
  "error": false,
  "msg": "Ciudadano encontrado",
  "data": {
    "id": "65f0a1b2c3d4e5f678901234",
    "nombre": "JUAN",
    "ap_paterno": "PEREZ",
    "ap_materno": "LOPEZ",
    "telefono": "8123456789",
    "email": "juan@email.com",
    "direccion": "Monterrey, Nuevo León",
    "folios": [
      {
        "tipo": "Reporte ciudadano",
        "folio": "XA-2026-000001",
        "status": "Abierto",
        "categoria": 1,
        "finalizado": false,
        "direccion": "Calle Principal 100, Monterrey",
        "createdAt": "2026-03-10T15:20:00.000Z"
      }
    ]
  }
}
```

## Respuesta cuando no existe coincidencia
```json
{
  "error": true,
  "msg": "No se encontró ningún ciudadano con los datos proporcionados.",
  "bettaCod": 2003,
  "statusCod": 404
}
```

## Notas de integración
- La búsqueda regresa un solo ciudadano.
- La respuesta incluye también el historial de folios asociados al ciudadano.
- Si no encuentra un ciudadano, el flujo recomendado es consultar o crear un contacto y continuar con la recepción de la gestión usando `solicitante_tipo: "contacto"`.

---

# 2. Listar gestiones

## Ruta
`POST /api/gestiones/list`

## Descripción
Obtiene el catálogo de gestiones disponibles para integración, incluyendo su estructura de formulario.

## Campos de envío
Todos los campos son opcionales.

| Campo | Tipo | Descripción |
|---|---|---|
| `enable_api` | boolean | Filtra únicamente las gestiones habilitadas para uso por API. |
| `tags` | string \| array | Filtra por etiqueta o conjunto de etiquetas. |
| `name` | string | Busca por nombre de la gestión. |

## Ejemplo de envío
```json
{
  "enable_api": true
}
```

## Respuesta exitosa
```json
{
  "error": false,
  "msg": "Lista de Gestiones",
  "data": [
    {
      "id": "61dca1bf130badfdeb50c6ea",
      "tituloInterno": "Bacheo",
      "tituloExterno": "Reporte de bache",
      "categoria": 1,
      "tags": ["bache", "calle", "vialidad"],
      "formulario": [
        {
          "key": "pregunta-condicionar",
          "pregunta": "¿Desea continuar?",
          "type": "seleccionunica",
          "nota": "Seleccione una opción",
          "obligatorio": true,
          "opcionesmult": [
            { "label": "SI", "value": "SI" },
            { "label": "NO", "value": "NO" }
          ],
          "condicional": null,
          "columns": 1
        },
        {
          "key": "pregunta-21726875206292",
          "pregunta": "Nombre del reporte",
          "type": "texto",
          "nota": "",
          "obligatorio": true,
          "opcionesmult": [],
          "condicional": null,
          "columns": 1
        }
      ]
    }
  ]
}
```

## Notas de integración
- Este endpoint debe utilizarse para obtener la estructura del formulario antes de enviar una gestión.
- El campo `key` de cada pregunta debe enviarse posteriormente en `/api/gestion/recept`.
- El `formulario` puede contener preguntas condicionales y catálogos de opciones.

---

# 3. Recepción de gestión

## Ruta
`POST /api/gestion/recept`

## Descripción
Registra una nueva gestión a partir de una gestión del catálogo, un solicitante y un conjunto de respuestas del formulario.

## Campos de envío

| Campo | Tipo | Requerido | Descripción |
|---|---|---:|---|
| `gestion_id` | string | Sí | Identificador de la gestión a registrar. |
| `forms` | array | Sí | Respuestas del formulario. Cada elemento debe contener `key` y `respuesta`. |
| `solicitante_id` | string | Sí | Identificador del solicitante. |
| `solicitante_tipo` | string | Sí | Tipo de solicitante. Valores esperados: `db` o `contacto`. |
| `localizacion` | object | No | Datos de ubicación asociados a la gestión. |

### Objeto `forms[]`

| Campo | Tipo | Descripción |
|---|---|---|
| `key` | string | Clave de la pregunta obtenida desde el catálogo de gestiones. |
| `respuesta` | string \| number \| boolean \| array | Respuesta capturada para la pregunta. |

### Objeto `localizacion`

| Campo | Tipo | Descripción |
|---|---|---|
| `lng` | number | Longitud. |
| `lat` | number | Latitud. |
| `calle` | string | Calle. |
| `ciudad` | string | Ciudad. |
| `colonia` | string | Colonia. |
| `cp` | string | Código postal. |
| `estado` | string | Estado. |
| `numero` | string | Número exterior o interior. |

## Ejemplo de envío
```json
{
  "gestion_id": "61dca1bf130badfdeb50c6ea",
  "forms": [
    {
      "key": "pregunta-condicionar",
      "respuesta": "SI"
    },
    {
      "key": "pregunta-21726875206292",
      "respuesta": "Jhon"
    },
    {
      "key": "pregunta-41726875716806",
      "respuesta": ""
    },
    {
      "key": "pregunta-51726876246243",
      "respuesta": 1
    },
    {
      "key": "pregunta-61726876372302",
      "respuesta": "Nada"
    },
    {
      "key": "pregunta-71736830144969",
      "respuesta": "TestQR"
    }
  ],
  "solicitante_id": "69b0bc707ec2502aa2475102",
  "solicitante_tipo": "contacto",
  "localizacion": {
    "lng": -99.167449,
    "lat": 19.427326,
    "calle": "Av. P.º de la Reforma",
    "ciudad": "Ciudad de México",
    "colonia": "Cuauhtémoc",
    "cp": "06600",
    "estado": "CDMX",
    "numero": "27"
  }
}
```

## Respuesta exitosa
```json
{
  "error": false,
  "msg": "Gestión receptada con éxito",
  "data": {
    "folio": "XA-2026-000145"
  }
}
```

## Notas de integración
- `gestion_id` debe obtenerse previamente desde `/api/gestiones/list`.
- `forms` debe construirse usando las claves `key` definidas en el formulario de la gestión.
- `solicitante_tipo` define el origen del solicitante:
  - `db`: cuando el solicitante existe como ciudadano.
  - `contacto`: cuando el solicitante existe como contacto.
- El flujo recomendado es:
  1. Buscar ciudadano.
  2. Si no existe, consultar o crear contacto.
  3. Registrar gestión indicando el tipo correcto de solicitante.
- La respuesta devuelve el folio generado de la gestión registrada.

---

# 4. Crear contacto

## Ruta
`POST /api/ciudadano/contacto/recept`

## Descripción
Crea un contacto para ser utilizado posteriormente como solicitante en la recepción de gestiones.

## Campos de envío

| Campo | Tipo | Requerido | Descripción |
|---|---|---:|---|
| `numero` | string | Sí | Número de contacto. |
| `nombre` | string | Sí | Nombre del contacto. |
| `email` | string | No | Correo electrónico del contacto. |

## Ejemplo de envío
```json
{
  "numero": "8129708788",
  "nombre": "Jonatan dos Santos",
  "email": ""
}
```

## Respuesta exitosa cuando se crea el contacto
```json
{
  "error": false,
  "msg": "Contacto creado con éxito",
  "data": {
    "id": "69b0bc707ec2502aa2475102",
    "numero": "8129708788",
    "name": "Jonatan dos Santos",
    "mail": ""
  }
}
```

## Respuesta exitosa cuando el contacto ya existe
```json
{
  "error": false,
  "msg": "El contacto ya existe",
  "data": {
    "id": "69b0bc707ec2502aa2475102",
    "numero": "8129708788",
    "name": "Jonatan dos Santos",
    "mail": ""
  }
}
```

## Notas de integración
- Este endpoint puede utilizarse antes de registrar una gestión cuando el solicitante no existe como ciudadano.
- Si el contacto ya existe, la API devuelve el mismo identificador para reutilizarlo.
- El identificador retornado en `data.id` puede enviarse posteriormente como `solicitante_id` en `/api/gestion/recept`.

---

# 5. Consultar contacto

## Ruta
`GET /api/ciudadano/contacto/:numero`

## Descripción
Consulta un contacto previamente registrado utilizando su número.

## Ejemplo de consumo
```http
GET /api/ciudadano/contacto/8129708788
```

## Respuesta exitosa
```json
{
  "error": false,
  "msg": "Contacto encontrado",
  "data": {
    "id": "69b0bc707ec2502aa2475102",
    "numero": "8129708788",
    "nombre": "Jonatan dos Santos",
    "mail": ""
  }
}
```

## Notas de integración
- Este endpoint permite validar si un número ya existe como contacto antes de intentar crearlo.
- Si el contacto existe, el valor `data.id` puede usarse como `solicitante_id` en la recepción de gestiones.

---

# 6. Listar categorías

## Ruta
`GET /api/categorias/lista`

## Descripción
Obtiene el catálogo de categorías disponibles para las gestiones.

## Ejemplo de consumo
```http
GET /api/categorias/lista
```

## Respuesta exitosa
```json
{
  "error": false,
  "msg": "Categorias obtenidas con éxito",
  "data": [
    {
      "id": 1,
      "name": "Reporte"
    },
    {
      "id": 2,
      "name": "Solicitud"
    }
  ]
}
```

## Notas de integración
- Este endpoint puede utilizarse para mostrar o clasificar las gestiones por categoría.
- Los valores reales del catálogo dependen de la configuración vigente del entorno.

---

# Flujo recomendado de integración

## Escenario 1: el solicitante existe como ciudadano
1. Consumir `POST /api/ciudadano/consultar` con alguno de los filtros disponibles.
2. Tomar el identificador del ciudadano encontrado.
3. Consultar `POST /api/gestiones/list` para obtener la gestión y su formulario.
4. Registrar la gestión en `POST /api/gestion/recept` enviando:
   - `solicitante_id`: identificador del ciudadano.
   - `solicitante_tipo`: `db`.

## Escenario 2: el solicitante no existe como ciudadano
1. Consumir `POST /api/ciudadano/consultar`.
2. Si no existe coincidencia, consultar `GET /api/ciudadano/contacto/:numero`.
3. Si el contacto no existe, crearlo mediante `POST /api/ciudadano/contacto/recept`.
4. Consultar `POST /api/gestiones/list` para obtener la gestión y su formulario.
5. Registrar la gestión en `POST /api/gestion/recept` enviando:
   - `solicitante_id`: identificador del contacto.
   - `solicitante_tipo`: `contacto`.

---

# Resumen rápido de endpoints

| Método | Ruta | Uso |
|---|---|---|
| POST | `/api/ciudadano/consultar` | Buscar ciudadano por CURP, teléfono o nombre. |
| POST | `/api/gestiones/list` | Obtener catálogo de gestiones y formulario. |
| POST | `/api/gestion/recept` | Registrar una nueva gestión. |
| POST | `/api/ciudadano/contacto/recept` | Crear contacto. |
| GET | `/api/ciudadano/contacto/:numero` | Consultar contacto por número. |
| GET | `/api/categorias/lista` | Obtener catálogo de categorías. |
