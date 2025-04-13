# **Trabajando con JSON en C#: Serialización y Deserialización para la Nube**

## **Introducción: JSON en el Mundo del Desarrollo Cloud**

JSON se ha convertido en el lenguaje universal para intercambiar datos en arquitecturas cloud. Como desarrollador, dominar su manejo en C# es crucial para:

- **Integrar microservicios** (APIs REST, Azure Functions, AWS Lambda)  
- **Procesar datos** en DynamoDB/Cosmos DB  
- **Automatizar flujos** con Logic Apps o Power Automate  

En este capítulo práctico, aprenderemos a:

1. **Serializar objetos C# a JSON** (para enviar datos a APIs)  
2. **Deserializar JSON a objetos C#** (para procesar respuestas)  

---

## **Configuración Inicial en Visual Studio**

### **1. Crear Proyecto y Agregar Newtonsoft.Json**
```csharp
// 1. Crear proyecto Windows Forms (o cualquier tipo)
// 2. Instalar NuGet Package:
Install-Package Newtonsoft.Json
```
**¿Por qué Newtonsoft?**  
- Librería más popular para JSON en .NET  
- Soporta casos complejos (anidamientos, formatos personalizados)  

---

## **Ejemplo 1: Serialización (Objeto → JSON)**

### **Clase de Ejemplo**
```csharp
public class UsuarioCloud 
{
    public string Nombre { get; set; }
    public string Apellido { get; set; }
    public int Edad { get; set; }
    public List<string> Servicios { get; set; } // Ej: ["AWS", "Azure"]
}
```

### **Serializar a JSON**
```csharp
var usuarioAWS = new UsuarioCloud 
{
    Nombre = "Ana",
    Apellido = "López",
    Edad = 32,
    Servicios = new List<string> { "EC2", "S3", "Lambda" }
};

string json = JsonConvert.SerializeObject(usuarioAWS);
Console.WriteLine(json);
```

**Salida (Formateada)**:
```json
{
  "Nombre": "Ana",
  "Apellido": "López",
  "Edad": 32,
  "Servicios": ["EC2", "S3", "Lambda"]
}
```

**Caso de Uso Cloud**:  
- Enviar datos a una **API REST de AWS** desde una aplicación .NET.  
- Almacenar configuración en **AWS Parameter Store**.  

---

## **Ejemplo 2: Deserialización (JSON → Objeto)**

### **JSON de Entrada (Desde API Cloud)**
```json
{
  "Nombre": "Carlos",
  "Apellido": "Ruiz",
  "Edad": 45,
  "Servicios": ["Azure Functions", "Cosmos DB"]
}
```

### **Deserializar en C#**
```csharp
string jsonFromCloud = File.ReadAllText("usuarioAzure.json");
UsuarioCloud usuario = JsonConvert.DeserializeObject<UsuarioCloud>(jsonFromCloud);

Console.WriteLine($"Bienvenido {usuario.Nombre}, dominas: {string.Join(", ", usuario.Servicios)}");
```

**Caso de Uso Cloud**:  
- Procesar eventos de **AWS S3 Notifications** en Lambda.  
- Leer datos de **Azure Cosmos DB Change Feed**.  

---

## **Truco Pro: Generar Clases Automáticamente desde JSON**

### **En Visual Studio:**
1. Copiar JSON de ejemplo:  
   ```json
   {
     "DeploymentId": "dpl-123",
     "Resources": ["VM-1", "Storage-ACCT"],
     "CostEstimate": 1500.50
   }
   ```
2. Pegar especial como clase:  
   **Edit → Paste Special → Paste JSON as Classes**  

**Clase Generada**:
```csharp
public class DeploymentCloud 
{
    public string DeploymentId { get; set; }
    public string[] Resources { get; set; }
    public float CostEstimate { get; set; }
}
```

**Caso de Uso**:  
- Rápido prototipado al integrar con **APIs externas** (ej: AWS Cost Explorer API).  

---

## **Ejemplo Avanzado: Anidamiento para Cloud**

### **JSON Complejo (AWS CloudFormation Template)**
```json
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Resources": {
    "MyEC2": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "InstanceType": "t2.micro"
      }
    }
  }
}
```

### **Clases C# Equivalentes**
```csharp
public class CloudFormationTemplate 
{
    public string AWSTemplateFormatVersion { get; set; }
    public Dictionary<string, Resource> Resources { get; set; }
}

public class Resource 
{
    public string Type { get; set; }
    public Properties Properties { get; set; }
}

public class Properties 
{
    public string InstanceType { get; set; }
}
```

**Uso en la Nube**:  
- Programáticamente generar/modificar plantillas IaC.  

---

## **Conclusión: JSON + C# en Arquitecturas Cloud**

1. **Serialización**:  
   - Convierte objetos C# en JSON para APIs cloud.  
   - Usa `JsonConvert.SerializeObject()`.  

2. **Deserialización**:  
   - Transforma respuestas JSON (ej: AWS SDK) en objetos manejables.  
   - Usa `JsonConvert.DeserializeObject<T>()`.  

3. **Generación Automática**:  
   - Ahorra tiempo al crear clases desde JSON de APIs cloud.  

**¿Qué sigue?**  
- Prueba serializar objetos con **Azure Blob Storage metadata**.  
- Explora **JSON Schema validation** para APIs críticas.  

¡Ahora estás listo para integrar C# con cualquier servicio cloud mediante JSON! 🌩️
