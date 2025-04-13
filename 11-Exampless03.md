# **Esquemas JSON para Eventos en AWS S3 y Azure Cosmos DB**

## **1. AWS S3 Event Notifications Schema**

### **Esquema para Eventos de S3 (ej: PUT/DELETE)**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "AWS S3 Event Notification",
  "type": "object",
  "required": ["Records"],
  "properties": {
    "Records": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["eventVersion", "eventSource", "awsRegion", "eventTime", "eventName", "s3"],
        "properties": {
          "eventVersion": {"type": "string", "const": "2.1"},
          "eventSource": {"type": "string", "const": "aws:s3"},
          "awsRegion": {"type": "string", "pattern": "^[a-z]{2}-[a-z]+-\\d$"},
          "eventTime": {"type": "string", "format": "date-time"},
          "eventName": {
            "type": "string",
            "enum": [
              "ObjectCreated:Put",
              "ObjectCreated:Post",
              "ObjectCreated:Copy",
              "ObjectRemoved:Delete"
            ]
          },
          "s3": {
            "type": "object",
            "required": ["bucket", "object"],
            "properties": {
              "bucket": {
                "type": "object",
                "required": ["name", "arn"],
                "properties": {
                  "name": {"type": "string", "pattern": "^[a-z0-9.-]{3,63}$"},
                  "arn": {"type": "string", "pattern": "^arn:aws:s3:::"}
                }
              },
              "object": {
                "type": "object",
                "required": ["key"],
                "properties": {
                  "key": {"type": "string"},
                  "size": {"type": "number", "minimum": 0},
                  "eTag": {"type": "string"}
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

**Casos de Uso**:
- Validar eventos antes de procesarlos en Lambda.
- Filtrar notificaciones por tipo de evento (`eventName`) o bucket.

**Ejemplo de Evento Válido**:
```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "us-east-1",
      "eventTime": "2023-11-25T14:30:00Z",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "my-data-lake",
          "arn": "arn:aws:s3:::my-data-lake"
        },
        "object": {
          "key": "incoming/sales.csv",
          "size": 1024,
          "eTag": "d41d8cd98f00b204e9800998ecf8427e"
        }
      }
    }
  ]
}
```

---

## **2. Azure Cosmos DB Change Feed Schema**

### **Esquema para Change Feed (Documentos Modificados)**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Azure Cosmos DB Change Feed",
  "type": "array",
  "items": {
    "type": "object",
    "required": ["id", "_rid", "_ts", "_etag", "operationType", "resourceType"],
    "properties": {
      "id": {"type": "string"},
      "_rid": {"type": "string"},
      "_ts": {"type": "number"},
      "_etag": {"type": "string"},
      "operationType": {
        "type": "string",
        "enum": ["Create", "Update", "Delete", "Replace"]
      },
      "resourceType": {"type": "string", "const": "document"},
      "partitionKey": {"type": "string"},
      "previousImage": {"type": ["object", "null"]},
      "currentImage": {
        "type": "object",
        "required": ["id", "partitionKey"],
        "properties": {
          "id": {"type": "string"},
          "partitionKey": {"type": "string"},
          "timestamp": {"type": "string", "format": "date-time"}
        }
      }
    }
  }
}
```

**Casos de Uso**:
- Validar cambios antes de procesarlos en Azure Functions.
- Replicar datos a otros sistemas (ej: Synapse Analytics).

**Ejemplo de Cambio Válido**:
```json
[
  {
    "id": "user-123",
    "_rid": "ABC123",
    "_ts": 1637847000,
    "_etag": "\"0000abcd-0000-0000-0000-000000000000\"",
    "operationType": "Update",
    "resourceType": "document",
    "partitionKey": "users",
    "currentImage": {
      "id": "user-123",
      "partitionKey": "users",
      "name": "María",
      "lastLogin": "2023-11-25T12:00:00Z"
    },
    "previousImage": {
      "id": "user-123",
      "partitionKey": "users",
      "name": "Maria", // Antiguo valor
      "lastLogin": "2023-11-24T08:30:00Z"
    }
  }
]
```

---

## **Comparativa: AWS S3 vs Azure Cosmos DB**

| **Característica**       | **AWS S3 Notifications**                          | **Azure Cosmos DB Change Feed**          |
|--------------------------|------------------------------------------------|----------------------------------------|
| **Estructura**           | Array de eventos (`Records`)                   | Array de cambios                      |
| **Campos Clave**         | `eventName`, `s3.bucket.name`, `s3.object.key` | `operationType`, `currentImage`, `previousImage` |
| **Tipos de Operaciones** | PUT, DELETE, etc.                              | Create, Update, Delete, Replace       |
| **Uso Típico**           | Trigger Lambda al subir/eliminar archivos      | Sincronizar datos entre sistemas      |
| **Herramientas**         | AWS Lambda, EventBridge                       | Azure Functions, Change Feed Processor |

---

## **Buenas Prácticas para Esquemas en la Nube**

1. **Validación Temprana**:
   - En AWS, usa **API Gateway Models** para rechazar peticiones inválidas antes de Lambda.
   ```yaml
   # Ejemplo en SAM/CloudFormation
   MyApi:
     Type: AWS::ApiGateway::RestApi
     Properties:
       Models:
         S3EventModel:
           ContentType: "application/json"
           Schema: !Include file://s3-event-schema.json
   ```

2. **Versionado**:
   - Etiqueta tus esquemas para compatibilidad:
   ```json
   {
     "$schema": "http://json-schema.org/draft-07/schema#",
     "version": "1.0.0",
     "description": "Esquema para eventos S3 v1"
   }
   ```

3. **Pruebas**:
   - Valida con herramientas como [ajv-cli](https://github.com/ajv-validator/ajv-cli):
   ```bash
   ajv test -s s3-schema.json -d s3-event.json --valid
   ```

---

**¿Necesitas esquemas para otros servicios como DynamoDB Streams o Azure Event Grid?** ¡Aquí están! 

### **Esquema para DynamoDB Streams**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["Records"],
  "properties": {
    "Records": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["eventID", "eventName", "dynamodb"],
        "properties": {
          "eventID": {"type": "string"},
          "eventName": {"enum": ["INSERT", "MODIFY", "REMOVE"]},
          "dynamodb": {
            "type": "object",
            "required": ["Keys", "NewImage"],
            "properties": {
              "Keys": {"type": "object"},
              "NewImage": {"type": ["object", "null"]}
            }
          }
        }
      }
    }
  }
}
```
