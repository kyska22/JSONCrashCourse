# **Esquemas JSON: El Contrato de Datos en la Nube**

## **De Bases de Datos Relacionales a JSON Schema**

Al crear una tabla en SQL Server como:
```sql
CREATE TABLE Test (
    TestId INT NOT NULL,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Age INT,
    Education VARCHAR(100)
)
```
Estamos definiendo un **esquema**: tipos de datos, campos obligatorios (`NOT NULL`) y longitudes. **JSON Schema cumple la misma función para APIs y servicios cloud**, pero con flexibilidad NoSQL.

---

## **Problema del Mundo Real: Comunicación entre Microservicios**

### **Escenario**  
- **Aplicación A**: Web que envía solicitudes (ej: inscripción a curso).  
- **Aplicación B**: Servicio backend que procesa datos.  

**Sin esquema**:  
```json
{
  "first_name": "Ana",
  "email": "ana@correo.com",
  "paid": "yes"  // ¡Error! Debería ser booleano.
}
```
El servicio falla silenciosamente o rechaza la petición sin claridad.

---

## **Solución: JSON Schema como Contrato**

### **Ejemplo de Solicitud Válida**
```json
{
  "action": "save_to_db",
  "first_name": "Carlos",
  "last_name": "Ruiz",
  "email": "carlos@cloud.com",
  "course": "JSON Crash Course",
  "enrollment_fee": 299,
  "paid": true
}
```

### **JSON Schema Correspondiente**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["action", "first_name", "last_name", "email", "course", "enrollment_fee", "paid"],
  "properties": {
    "action": {"type": "string", "enum": ["save_to_db", "update_record"]},
    "first_name": {"type": "string", "minLength": 1},
    "last_name": {"type": "string", "minLength": 1},
    "email": {"type": "string", "format": "email"},
    "course": {"type": "string"},
    "enrollment_fee": {"type": "number", "minimum": 0},
    "paid": {"type": "boolean"}
  }
}
```

**Cláusulas Clave**:  
- `required`: Campos obligatorios.  
- `type`: Define string/number/boolean.  
- `format`: Valida emails, URLs, etc.  
- `enum`: Lista de valores permitidos.  

---

## **Herramientas para Generar/Validar Esquemas**

### **1. Generación Automática**
- **Visual Studio Code**: Extensión `JSON Schema` infiere esquemas desde muestras de datos.  
- **Online Tools**: [jsonschema.net](https://www.jsonschema.net/) crea esquemas interactivamente.

### **2. Validación en Tiempo Real**
```bash
# Usando ajv (Node.js)
npx ajv validate -s schema.json -d request.json
```

### **3. Integración en Servicios Cloud**
- **Azure Logic Apps**:  
  ```json
  "actions": {
    "Validate_JSON": {
      "type": "JsonValidation",
      "inputs": {
        "json": "@triggerBody()",
        "schema": "@json(triggerOutputs()?['headers']?['schema'])"
      }
    }
  }
  ```
- **AWS API Gateway**:  
  Usa **Modelos** para validar solicitudes contra esquemas antes de llegar a Lambda.

---

## **Por qué JSON Schema es Crítico en la Nube**

1. **Seguridad**:  
   - Evita inyección de datos malformados (ej: `enrollment_fee: -100`).  

2. **Eficiencia**:  
   - Rechaza peticiones inválidas antes de procesarlas, ahorrando ciclos de cómputo.  

3. **Documentación Viva**:  
   - El esquema sirve como documentación técnica para equipos de desarrollo.  

4. **Compatibilidad**:  
   - Usado en **OpenAPI**, **Azure Event Grid**, y **AWS CloudFormation**.  

---

## **Ejemplo Avanzado: Esquema para DynamoDB**
```json
{
  "type": "object",
  "required": ["PK", "SK"],
  "properties": {
    "PK": {"type": "string", "pattern": "^USER#\\d+"},
    "SK": {"type": "string", "pattern": "^PROFILE#\\d+"},
    "credit": {"type": "number", "minimum": 0},
    "tags": {"type": "array", "items": {"type": "string"}}
  }
}
```
**Uso**: Validar items antes de `PutItem` en DynamoDB Streams.

---

**Conclusión**:  
JSON Schema es el **"CREATE TABLE" del mundo NoSQL** y la base para APIs confiables en arquitecturas serverless.
