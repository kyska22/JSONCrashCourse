# **Ejemplos Avanzados de JSON en Servicios Cloud: AWS S3 y Azure Blob Storage**

## **1. Configuración de Buckets/Contenedores en JSON**

### **AWS S3: Configuración de Bucket con Políticas**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E123456789ABCD"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-cloud-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": ["192.0.2.0/24"]
        }
      }
    }
  ]
}
```

**Estructura y Uso**:
- **`Version`**: Versión de política IAM (siempre "2012-10-17").
- **`Statement`**: Array de reglas (puede tener múltiples).
  - **`Sid`**: ID descriptivo.
  - **`Effect`**: `Allow`/`Deny`.
  - **`Principal`**: Quién accede (ARN de IAM, OAI para CloudFront).
  - **`Action`**: Operaciones permitidas (`s3:GetObject`, `s3:PutObject`).
  - **`Resource`**: ARN del bucket/objeto (`arn:aws:s3:::bucket-name/*`).
  - **`Condition`**: Restricciones (IPs, cifrado, etc.).

**Caso de Uso**:  
- Vincular CloudFront a S3 con Origin Access Identity (OAI).
- Restringir acceso por IP.

---

### **Azure Blob Storage: Metadata de Blob**
```json
{
  "blobProperties": {
    "contentType": "application/json",
    "contentEncoding": "gzip",
    "metadata": {
      "uploadedBy": "data-pipeline-1",
      "environment": "production"
    },
    "tags": {
      "project": "blueprint-x",
      "retention": "90-days"
    }
  }
}
```

**Estructura y Uso**:
- **`contentType`**: Tipo MIME (ej: `text/csv`).
- **`metadata`**: Pares clave-valor personalizados (máx. 8 KB).
- **`tags`**: Etiquetas para gestión de costos/búsqueda.

**Caso de Uso**:  
- Automatizar uploads desde Azure Data Factory.
- Clasificar blobs para lifecycle management.

---

## **2. Notificaciones de Eventos**

### **AWS S3 Event Notification**
```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "us-east-1",
      "eventTime": "2023-10-05T12:00:00Z",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "my-cloud-bucket",
          "arn": "arn:aws:s3:::my-cloud-bucket"
        },
        "object": {
          "key": "incoming/data.json",
          "size": 1024,
          "eTag": "d41d8cd98f00b204e9800998ecf8427e"
        }
      }
    }
  ]
}
```

**Estructura y Uso**:
- **`Records`**: Array de eventos (puede contener múltiples).
  - **`eventName`**: Tipo de evento (`ObjectCreated:Put`, `ObjectRemoved:Delete`).
  - **`s3.bucket`**: Nombre y ARN del bucket.
  - **`s3.object`**: Detalles del objeto (key, tamaño, eTag).

**Caso de Uso**:  
- Trigger Lambda cuando se sube un archivo a S3.
- Integración con EventBridge para workflows complejos.

---

### **Azure Blob Storage Event Grid**
```json
{
  "topic": "/subscriptions/123/resourceGroups/my-rg/providers/Microsoft.Storage/storageAccounts/mystorage",
  "subject": "/blobServices/default/containers/mycontainer/blobs/data.csv",
  "eventType": "Microsoft.Storage.BlobCreated",
  "data": {
    "api": "PutBlob",
    "clientRequestId": "a1b2c3d4",
    "contentType": "text/csv",
    "url": "https://mystorage.blob.core.windows.net/mycontainer/data.csv"
  }
}
```

**Estructura y Uso**:
- **`subject`**: Ruta al blob.
- **`eventType`**: `BlobCreated`/`BlobDeleted`.
- **`data.url`**: URL directa al blob.

**Caso de Uso**:  
- Activar Azure Functions cuando se modifica un blob.
- Notificaciones a Logic Apps.

---

## **3. Operaciones Batch con JSON**

### **AWS S3 Batch Operations**
```json
{
  "invocationSchemaVersion": "1.0",
  "tasks": [
    {
      "taskId": "task-1",
      "s3Bucket": "my-cloud-bucket",
      "s3Key": "inventory/report.csv",
      "operation": {
        "type": "LambdaInvoke",
        "functionArn": "arn:aws:lambda:us-east-1:123:function:process-csv"
      }
    },
    {
      "taskId": "task-2",
      "s3Bucket": "my-cloud-bucket",
      "s3Key": "archives/old-data.zip",
      "operation": {
        "type": "S3PutObjectTagging",
        "tagSet": [
          {"key": "retention", "value": "365-days"}
        ]
      }
    }
  ]
}
```

**Estructura y Uso**:
- **`tasks`**: Array de operaciones.
  - **`type`**: `LambdaInvoke`, `S3PutObjectTagging`, etc.
  - **`tagSet`**: Etiquetas para lifecycle policies.

**Caso de Uso**:  
- Procesamiento masivo de objetos con Lambda.
- Aplicar tags a millones de blobs.

---

## **Buenas Prácticas Cloud**
1. **Validación**:  
   - AWS: `aws iam validate-policy --policy-document file://policy.json`  
   - Azure: `az storage blob upload --validate-content`  

2. **Seguridad**:  
   - Encriptar JSON con AWS KMS o Azure Key Vault.  
   - Usar `Condition` en políticas S3 para exigir cifrado (`s3:x-amz-server-side-encryption`).  

3. **Optimización**:  
   - Para blobs >1MB en Azure, usar `"contentType": "application/octet-stream"`.  
   - En S3, activar `"RequestPayer": "requester"` para acceso cross-account.  
