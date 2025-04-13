# **Ejemplos Avanzados de JSON en DynamoDB y Cosmos DB**

## **1. DynamoDB (AWS): Modelado de Datos NoSQL**

### **Ejemplo 1: Item Básico en DynamoDB**
```json
{
  "UserId": {"S": "USR-12345"},
  "Name": {"S": "María García"},
  "AccountStatus": {"S": "ACTIVE"},
  "LastLogin": {"S": "2023-11-20T08:30:00Z"},
  "Credits": {"N": "1500"},
  "Preferences": {
    "M": {
      "Theme": {"S": "dark"},
      "Notifications": {"BOOL": true}
    }
  },
  "Roles": {"SS": ["admin", "editor"]}
}
```

**Estructura y Tipos de Datos**:
- **`S`**: String (requerido para claves de partición)
- **`N`**: Number (para cálculos matemáticos)
- **`M`**: Map (objeto anidado)
- **`BOOL`**: Boolean
- **`SS`**: String Set (array de strings únicos)

**Caso de Uso**:  
Perfil de usuario con:
- Atributos escalares (`UserId`, `Name`)
- Objeto anidado (`Preferences`)
- Conjunto de roles (`Roles`)

---

### **Ejemplo 2: Índice Secundario Global (GSI)**
```json
{
  "PK": {"S": "CUST#ACME"},       // Clave de partición
  "SK": {"S": "ORDER#2023-1001"}, // Clave de ordenación
  "GSI1PK": {"S": "STATUS#SHIPPED"}, // Clave GSI
  "OrderDate": {"S": "2023-11-15"},
  "Items": {
    "L": [
      {"M": {"ProductId": "P-100", "Qty": 2}},
      {"M": {"ProductId": "P-200", "Qty": 1}}
    ]
  },
  "Total": {"N": "59.98"}
}
```

**Patrón de Diseño**:  
- **PK/SK**: Modelo de tabla única (Single-Table Design)
- **GSI1PK**: Consultar órdenes por estado
- **`L`**: Lista de items (array de objetos)

**Query Ejemplo**:  
```bash
aws dynamodb query \
  --table-name Orders \
  --index-name GSI1 \
  --key-condition-expression "GSI1PK = :status" \
  --expression-attribute-values '{":status":{"S":"STATUS#SHIPPED"}}'
```

---

## **2. Azure Cosmos DB (NoSQL API)**

### **Ejemplo 1: Documento con Particionamiento**
```json
{
  "id": "prod-987",
  "categoryId": "electronics",
  "type": "product",
  "name": "Monitor 4K",
  "price": 299.99,
  "tags": ["4K", "27-inch", "gaming"],
  "metadata": {
    "weight_kg": 5.8,
    "dimensions": {"width": 61.2, "height": 35.6}
  },
  "_partitionKey": "electronics"
}
```

**Características Clave**:
- **`id`**: Identificador único (requerido)
- **`_partitionKey`**: Clave de partición (optimiza rendimiento)
- **Tipos nativos de JSON**: Arrays (`tags`), objetos anidados (`dimensions`)

**Consulta SQL en Cosmos DB**:  
```sql
SELECT * FROM c 
WHERE c.categoryId = "electronics" AND ARRAY_CONTAINS(c.tags, "gaming")
```

---

### **Ejemplo 2: Datos Temporales con TTL**
```json
{
  "id": "sess-xyz789",
  "userId": "user-123",
  "sessionStart": "2023-11-20T10:00:00Z",
  "lastActivity": "2023-11-20T10:45:00Z",
  "ttl": 86400, // TTL en segundos (24 horas)
  "data": {
    "cartItems": 3,
    "preferences": {"currency": "USD"}
  }
}
```

**Configuración de TTL**:  
1. Habilitar TTL en el contenedor:  
   ```bash
   az cosmosdb sql container update \
     --resource-group my-rg \
     --account-name my-cosmosdb \
     --database-name ecommerce \
     --name sessions \
     --ttl 86400
   ```
2. El campo `ttl` automáticamente elimina documentos pasadas 24h.

---

## **Comparativa DynamoDB vs Cosmos DB**

| **Característica**       | **DynamoDB**                          | **Cosmos DB**                        |
|--------------------------|---------------------------------------|---------------------------------------|
| **Sintaxis JSON**        | Requiere especificar tipos (`{"S": "value"}`) | Usa JSON estándar (sin wrappers) |
| **Particionamiento**     | Clave de partición (`PK`) + ordenación (`SK`) | Cualquier propiedad como `partitionKey` |
| **Consultas**            | Limited a operaciones con claves (a menos que uses GSIs) | SQL completo + MongoDB API |
| **TTL**                  | Configurable por item                 | Configuración a nivel de contenedor + item |
| **Precio**               | Basado en RCU/WCU                     | Basado en RU (Unidades de Request) |

---

## **Buenas Prácticas Comunes**

### **Para DynamoDB**
1. **Single-Table Design**:  
   - Almacenar múltiples entidades en una tabla (ej: `PK=USER#123`, `SK=PROFILE#123`)
2. **Índices Globales (GSI)**:  
   - Crear GSIs para patrones de acceso alternativos
3. **Expresiones condicionales**:  
   ```bash
   aws dynamodb put-item \
     --table-name Users \
     --item file://user.json \
     --condition-expression "attribute_not_exists(UserId)"
   ```

### **Para Cosmos DB**
1. **Selección de partitionKey**:  
   - Elegir propiedades con alta cardinalidad (ej: `userId` vs `country`)
2. **Optimizar RU/s**:  
   - Usar consultas con `DISTINCT` y `TOP` para reducir RU consumidas
3. **Change Feed**:  
   - Procesar cambios en tiempo real:
     ```csharp
     var changes = container.GetChangeFeedProcessorBuilder("processorName", HandleChanges)
         .WithInstanceName("instance1")
         .WithLeaseContainer(leaseContainer)
         .Build();
     ```
