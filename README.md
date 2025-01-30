# Documentación de la API de Betta

## Autenticación y Seguridad

Para realizar cualquier operación en el entorno de **Betta**, es obligatorio generar un **BettaAuth Token** desde el panel administrativo. A considerar:

- Cada token es **específico para una funcionalidad** y debe ser aprobado por **Betta** antes de su activación por razones de seguridad y privacidad, basadas en los estándares de **ISO/IEC 27001 y su extensión ISO/IEC 27701**.
- **No se permiten consultas masivas**, ni por rangos de fecha, ni descargas masivas.
- Todas las consultas realizadas a la API son **registradas y monitoreadas**.
- Cualquier intento de descarga masiva o consulta automatizada en bucle será detectado por el sistema, resultando en la **desactivación del API y la inclusión de la IP en una lista negra**.
- Para solicitar el desbloqueo de un token, se debe presentar una explicación técnica detallada, firmada por el responsable, justificando el intento de acceso.

---

## 1. Consultar Folio

### **Endpoint**

```http
GET https://gql.portal.betta.com.mx/api/folio/{folio}
```

### **Parámetros**

- `{folio}`: Identificador del folio a consultar (tipo **string**).
- El token de autenticación debe contar con permisos específicos para acceder al folio solicitado.

### **Respuesta (JSON)**

```json
{
  "dependencia": "string",
  "generadoPor": "string",
  "nombreGestion": "string",
  "nombreGestionExterna": "string",
  "folio": "string",
  "estatus": "string",
  "ciudadano": {
    "nombre": "string",
    "apellido_paterno": "string",
    "apellido_materno": "string",
    "email": "string",
    "telefono": "string",
    "esciudadano": "boolean"
  },
  "origenFolio": "string",
  "colonia": "string",
  "direccionFolio": "string",
  "coordinates": {
    "lat": "float",
    "lng": "float"
  },
  "fechaCreacion": "string",
  "formulario": [
    {
      "key": "string",
      "pregunta": "string",
      "respuesta": "string | Object | Array",
      "respuesta_array": "string"
    }
  ],
  "cuantificacion": [
    {
      "key": "string",
      "pregunta": "string",
      "respuesta": "string | Object | Array",
      "peso": "number"
    }
  ],
  "formulas": [
    {
      "key": "string",
      "pregunta": "string",
      "respuesta": "string | Object | Array",
      "peso": "number"
    }
  ]
}
```

---

## 2. Cambiar Estatus de un Folio

### **Endpoint**

```http
POST https://gql.portal.betta.com.mx/api/folio/cambiar/status
```

### **Body (JSON)**

```json
{
  "folio": "string",
  "statusId": "string"
}
```

- `folio`: Identificador del folio.
- `statusId`: ID del nuevo estado del folio (debe coincidir con el tipo de folio: **Solicitud/Reporte/Trámite**).
  - Las **ID de estatus** se encuentran en el portal administrativo de Betta, en la configuración de estatus API.

![Estatus](https://storage.googleapis.com/betta-prod/BETTA/estatuskey.png)

### **Respuesta (JSON)**

```json
{
  "success": true,
  "msg": "Estatus cambiado con éxito"
}
```

---

## 3. Senders (Envió de Folios a Sistemas Externos)

### **Descripción**

Los **Senders** permiten la integración de Betta con sistemas externos en tiempo real, notificando sobre folios generados.

### **Configuración**


![Configuración](https://storage.googleapis.com/betta-prod/BETTA/senders.png)


Desde la configuración de seguridad en el portal Betta, se deben definir:

- **Método:** `POST` o `PUT`.
- **URL:** Dirección del endpoint receptor.
- **Headers:** Deben enviarse en formato JSON:

```json
{
   "KEY": "VALUE"
}
```

- **Body Extra:** Se enviará en formato JSON con la siguiente estructura:

```json
{
   "folio": { },
   "extra": { }
}
```

- El campo `folio` contiene la información detallada del folio.
- El campo `extra` permite enviar parámetros personalizados para identificadores adicionales en la petición.

---

## 4. Errores y Códigos de Estado

En caso de error, la API retornará una respuesta JSON con un mensaje descriptivo y un código de error **codBetta**. Consultar la documentación en el panel administrativo para más detalles sobre los posibles errores y sus soluciones.

---

## Contacto y Soporte

Para consultas técnicas, contactar con el equipo de soporte de **Betta** a través del portal administrativo.

---

**Última actualización:** 30/01/2025

**Jonatan dos Santos**  
_<small>jonatan@betta.com.mx</small>_  
_<small>CTO / Founder<small>_
