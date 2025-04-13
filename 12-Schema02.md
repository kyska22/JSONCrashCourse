# **Validación de Esquemas JSON: El Contrato de Datos en la Nube**

## **De Objetos JSON a Esquemas Validados**

Al crear un objeto JSON como:
```json
{
  "action": "save_to_db",
  "first_name": "Ana",
  "last_name": "López",
  "email": "ana@cloud.com",
  "course": "JSON Avanzado",
  "enrollment_fee": 299,
  "paid": true
}
```
Necesitamos un **esquema** que defina reglas para validar su estructura, tipos de datos y campos obligatorios.

---

## **Generación Automática de Esquemas con Herramientas**

### **1. Extracción del Esquema (json-schema.net)**
- **Proceso**:
  1. Pegar el JSON de ejemplo en el panel izquierdo.
  2. La herramienta genera automáticamente el esquema en el panel derecho.
  3. Ajustar campos requeridos y patrones (ej: validar emails con regex).

**Esquema Generado**:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["action", "first_name", "last_name", "email", "course", "enrollment_fee", "paid"],
  "properties": {
    "action": {"type": "string"},
    "first_name": {"type": "string"},
    "last_name": {"type": "string"},
    "email": {"type": "string", "format": "email"},
    "course": {"type": "string"},
    "enrollment_fee": {"type": "integer"},
    "paid": {"type": "boolean"}
  }
}
```

**Ajustes Clave**:
- Eliminar `enrollment_fee` de `required` si es opcional.
- Agregar `"format": "email"` para validación estricta.
- Definir `enum` en `action` para limitar valores permitidos:
  ```json
  "action": {"type": "string", "enum": ["save_to_db", "update_record"]}
  ```

---

## **Validación en Tiempo Real**

### **2. Uso de JSON Schema Validator**
1. **Copiar el esquema generado**.
2. **Pegarlo en una herramienta como [JSON Schema Validator](https://www.jsonschemavalidator.net/)**.
3. **Probar con datos reales**:

**Caso Válido**:
```json
{
  "action": "save_to_db",
  "first_name": "Carlos",
  "paid": false
}
```
- **Error**: Faltan campos requeridos (`last_name`, `email`, etc.).

**Caso Inválido**:
```json
{
  "action": "save_to_db",
  "first_name": "Ana",
  "paid": "yes"  // ¡Error! Debe ser booleano.
}
```
- **Mensaje**: `Invalid type: expected boolean but got string`.

---

## **Buenas Prácticas en Entornos Cloud**

### **Para AWS (API Gateway + Lambda)**
1. **Definir Modelos en API Gateway**:
   ```yaml
   # CloudFormation
   MyApi:
     Type: AWS::ApiGateway::Model
     Properties:
       RestApiId: !Ref MyApi
       ContentType: "application/json"
       Schema: !Include file://schema.json
   ```
   - Rechaza peticiones malformadas antes de invocar Lambda.

### **Para Azure (Logic Apps)**
- **Usar el conector `JSON Validation`**:
  ```json
  {
    "actions": {
      "Validate_Input": {
        "type": "JsonValidation",
        "inputs": {
          "json": "@triggerBody()",
          "schema": {
            "type": "object",
            "required": ["action"],
            "properties": {
              "action": {"type": "string"}
            }
          }
        }
      }
    }
  }
  ```

---

## **Conclusión: Por qué los Esquemas son Esenciales**
1. **Seguridad**: Evita inyección de datos malformados.
2. **Eficiencia**: Reduce errores en pipelines ETL.
3. **Documentación**: El esquema sirve como contrato técnico.

**Ejemplo Avanzado (AWS S3 Event)**:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["Records"],
  "properties": {
    "Records": {
      "type": "array",
      "items": {
        "required": ["s3"],
        "properties": {
          "s3": {
            "type": "object",
            "required": ["bucket"],
            "properties": {
              "bucket": {
                "type": "object",
                "required": ["name"],
                "properties": {
                  "name": {"type": "string", "pattern": "^[a-z0-9.-]{3,63}$"}
                }
              }
            }
          }
        }
      }
    }
  }
}
```

**¿Necesitas esquemas para otros servicios como DynamoDB Streams o Azure Event Grid?** ¡Aquí tienes!

### **Esquema para DynamoDB Streams**
```json
{
  "type": "object",
  "required": ["Records"],
  "properties": {
    "Records": {
      "type": "array",
      "items": {
        "required": ["dynamodb"],
        "properties": {
          "dynamodb": {
            "type": "object",
            "required": ["NewImage"],
            "properties": {
              "NewImage": {
                "type": "object",
                "required": ["id"],
                "properties": {
                  "id": {"type": "string"}
                }
              }
            }
          }
        }
      }
    }
  }
}
```

**¡Domina los esquemas JSON y lleva tus integraciones cloud al siguiente nivel!** 🚀
