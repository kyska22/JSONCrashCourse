### **En JSON, existen **6 tipos de datos**:**  

"Hasta ahora, creamos nuestro objeto `Persona` con propiedades como `first_name`, `last_name`, `date_of_birth`, `spouse` y `children`, todas tratadas como *strings* (entre comillas dobles).  

Pero en JSON, existen **6 tipos de datos**:  
1. **String**: Texto entre comillas dobles (`"nombre": "Sasha"`).  
2. **Number**: Números *sin* comillas (`"children": 3`).  
3. **Boolean**: `true` o `false` (palabras reservadas, sin comillas).  
4. **Null**: Para propiedades sin valor (`"middle_name": null`).  
5. **Object**: Propiedades que son objetos anidados (ej: `"spouse": { "first_name": "Isla" }`).  
6. **Array**: Listas entre corchetes (`"hobbies": ["acting", "comedy"]`).  

---

### **Explicación desde la perspectiva de la nube:**  

#### **1. Tipos de datos en JSON y su impacto en la nube**  
- **Strings**:  
  - Usados en nombres de recursos (ej: `"bucket_name": "my-cloud-storage"` en AWS S3).  
  - **Cuidado**: No confundir con valores que deben ser numéricos (ej: `"memory_size": "1024"` es incorrecto; debe ser `1024` sin comillas).  

- **Numbers**:  
  - Clave para configuraciones de escalado:  
    ```json
    {
      "autoscale": {
        "min_instances": 1,
        "max_instances": 5
      }
    }
    ```  

- **Booleans**:  
  - Habilitan/deshabilitan features en políticas cloud:  
    ```json
    {
      "encryption_enabled": true,
      "public_access": false
    }
    ```  
  - **¡Case-sensitive!**: `True` o `False` (con mayúscula) invalida el JSON. Solo `true`/`false`.  

- **Objects**:  
  - Anidar objetos es común en plantillas IaC (ej: definir una subnet dentro de una VPC en AWS):  
    ```json
    {
      "VPC": {
        "subnet": {
          "cidr": "10.0.0.0/24"
        }
      }
    }
    ```  

- **Arrays**:  
  - Usados en reglas de seguridad (ej: puertos permitidos en un Security Group):  
    ```json
    {
      "allowed_ports": [80, 443, 8080]
    }
    ```  

#### **2. Validación en entornos cloud**  
- **Errores comunes**:  
  - Usar comillas en números o booleanos rompe plantillas de Terraform/CloudFormation.  
  - Mayúsculas en `true`/`false` causa fallos en APIs de Azure/AWS.  

- **Herramientas**:  
  - **CLI**:  
    ```bash
    # Valida JSON con jq
    echo '{"is_active": true}' | jq empty
    ```  
  - **Azure**:  
    ```bash
    az deployment group validate --template-file template.json
    ```  

#### **3. Ejemplo práctico: Configuración de un Lambda (AWS)**  
```json
{
  "function_name": "data-processor",
  "timeout": 30,
  "memory_size": 256,
  "environment_vars": {
    "ENV": "production"
  },
  "layers": ["arn:aws:lambda:us-east-1:123456789012:layer:my-layer:1"],
  "tracing_enabled": true
}
```  
- **Strings**: `function_name`, `ENV`.  
- **Numbers**: `timeout`, `memory_size`.  
- **Boolean**: `tracing_enabled`.  
- **Array**: `layers`.  

#### **4. Buenas prácticas**  
- **Siempre validar JSON** antes de desplegar en la nube.  
- Usar **VS Code** con extensión JSON Schema para autocompletado.  
- En **CI/CD**, agregar un paso de validación con `jq`.  

--- 

**¿Por qué es crítico?**  
Un error de sintaxis en JSON puede:  
- Fallar un despliegue en CloudFormation.  
- Romper una pipeline DevOps.  
- Generar costos inesperados (ej: si `"auto_delete": false` se escribe como `"false"`).  
