# **Dominando JSON y sus Esquemas para la Nube**

## **1. Fundamentos de JSON**
- **Qué es**: Formato ligero de intercambio de datos (JavaScript Object Notation).
- **Estructura básica**:
  ```json
  {
    "clave": "valor",
    "numérico": 42,
    "booleano": true,
    "array": ["item1", "item2"],
    "objetoAnidado": {"prop": "valor"}
  }
  ```
- **Ventajas**: Legible para humanos y máquinas, universal en APIs cloud.

## **2. Tipos de Datos en JSON**
| **Tipo**       | **Ejemplo**          | **Uso en Cloud**                          |
|----------------|----------------------|-------------------------------------------|
| String         | `"nombre": "AWS"`    | Nombres de recursos, IDs                  |
| Number         | `"CPU": 4`           | Configuración de instancias               |
| Boolean        | `"activo": true`     | Flags de estado                           |
| Null           | `"middleName": null` | Campos opcionales no definidos            |
| Array          | `"tags": ["prod"]`   | Listas de seguridad, múltiples valores    |
| Object         | `"metadata": {...}`  | Estructuras complejas (ej: configuraciones)|

## **3. Objetos Anidados y Arrays**
- **Jerarquías complejas**:
  ```json
  {
    "usuario": {
      "nombre": "Ana",
      "direcciones": [
        {"tipo": "casa", "ciudad": "Madrid"}
      ]
    }
  }
  ```
- **Caso de uso**: Modelar relaciones en DynamoDB/Cosmos DB sin tablas relacionales.

## **4. JSON Schema: El Contrato de Datos**
- **Para qué sirve**:
  - Validar estructura de datos.
  - Documentar APIs.
  - Garantizar integridad en integraciones cloud.

- **Ejemplo clave**:
  ```json
  {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "required": ["email"],
    "properties": {
      "email": {"type": "string", "format": "email"},
      "age": {"type": "number", "minimum": 18}
    }
  }
  ```

## **5. Aplicación en Servicios Cloud**
### **AWS**
- **S3 Notifications**: Validar eventos de buckets.
- **DynamoDB**: Esquemas para items y streams.
- **API Gateway**: Rechazar peticiones malformadas.

### **Azure**
- **Cosmos DB**: Validar documentos y change feeds.
- **Logic Apps**: Conector de validación JSON.
- **Event Grid**: Esquemas para eventos.

## **6. Herramientas Imprescindibles**
| **Herramienta**         | **Función**                              | **Enlace**                     |
|--------------------------|------------------------------------------|--------------------------------|
| JSON Schema Generator    | Crear esquemas desde ejemplos            | [jsonschema.net](https://www.jsonschema.net/) |
| JSON Validator           | Validar contra esquemas                  | [www.jsonschemavalidator.net](https://www.jsonschemavalidator.net/) |
| VS Code JSON Extension   | Soporte para esquemas con autocompletado | Extensión oficial              |

## **Conclusión Final**
JSON y sus esquemas son **el lenguaje universal de los datos en la nube**. Al dominarlos:
- **Evitas errores costosos** en pipelines de datos.
- **Automatizas validaciones** en serverless y microservicios.
- **Documentas APIs** de forma precisa y ejecutable.

**Ahora tienes las bases para:**
1. Modelar datos complejos en DynamoDB/Cosmos DB.
2. Diseñar APIs confiables en AWS/Azure.
3. Garantizar calidad en integraciones entre servicios.

**¡Convierte JSON Schema en tu aliado para construir sistemas cloud resilientes!** 🚀
