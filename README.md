# 🪣 Terraform — VAPI S3 Setup

Template de Terraform para automatizar el aprovisionamiento de infraestructura AWS necesaria en cada despliegue de un agente de voz con VAPI.

## ¿Qué crea?

Por cada proyecto se generan automáticamente los siguientes recursos en AWS:

- **Bucket S3** con bloqueo total de acceso público, cifrado SSE-S3 y etiquetas de proyecto.
- **Política IAM** con permisos restringidos al bucket (`PutObject`, `GetObject`, `DeleteObject`, `ListBucket`).
- **Usuario IAM** con la política adjunta y credenciales de acceso programático listas para configurar en VAPI.

### Nomenclatura generada

| Recurso | Nombre |
|---|---|
| Bucket S3 | `{proyecto}-s3-vapi-grabaciones` |
| Política IAM | `{proyecto}-vapi-s3-policy` |
| Usuario IAM | `{proyecto}-vapi-s3-user` |

---

## Requisitos

### 1. Terraform CLI

1. Descarga el ZIP para tu sistema operativo en [developer.hashicorp.com/terraform/install](https://developer.hashicorp.com/terraform/install)
2. Extrae el archivo `terraform.exe` (Windows) o `terraform` (Mac/Linux)
3. Muévelo a una carpeta, por ejemplo `C:\terraform\`
4. Agrega esa carpeta al PATH del sistema:
   - Windows: Busca "Variables de entorno" → Variables del sistema → `Path` → Editar → Agregar `C:\terraform`
   - Mac/Linux: Agrega `export PATH=$PATH:/usr/local/terraform` a tu `.bashrc` o `.zshrc`
5. Verifica la instalación:
```bash
terraform -version
```

---

### 2. AWS CLI

1. Descarga e instala desde [aws.amazon.com/cli](https://aws.amazon.com/cli)
2. Verifica la instalación:
```bash
aws --version
```

---

### 3. Usuario IAM para Terraform

Crea un usuario IAM en AWS con los siguientes permisos mínimos y genera sus credenciales de acceso programático (Access Key + Secret Key).

**Política IAM requerida:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3Permissions",
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:PutBucketPublicAccessBlock",
        "s3:PutBucketVersioning",
        "s3:PutBucketTagging",
        "s3:PutEncryptionConfiguration",
        "s3:GetBucketPolicy",
        "s3:GetBucketPublicAccessBlock",
        "s3:GetBucketVersioning",
        "s3:GetBucketTagging",
        "s3:GetEncryptionConfiguration",
        "s3:ListBucket",
        "s3:GetBucketAcl",
        "s3:GetBucketCORS",
        "s3:GetBucketWebsite",
        "s3:GetBucketLogging",
        "s3:GetLifecycleConfiguration",
        "s3:GetReplicationConfiguration",
        "s3:GetAccelerateConfiguration",
        "s3:GetBucketRequestPayment",
        "s3:GetBucketObjectLockConfiguration"
      ],
      "Resource": "*"
    },
    {
      "Sid": "IAMPermissions",
      "Effect": "Allow",
      "Action": [
        "iam:CreatePolicy",
        "iam:GetPolicy",
        "iam:GetPolicyVersion",
        "iam:ListPolicyVersions",
        "iam:CreateUser",
        "iam:GetUser",
        "iam:TagUser",
        "iam:AttachUserPolicy",
        "iam:ListAttachedUserPolicies",
        "iam:CreateAccessKey",
        "iam:ListAccessKeys"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### 4. Configurar credenciales AWS

Con las credenciales del usuario IAM creado, ejecuta:

```bash
aws configure
```

Llena los datos solicitados:

```
AWS Access Key ID: TU_ACCESS_KEY
AWS Secret Access Key: TU_SECRET_KEY
Default region name: us-east-1
Default output format: json
```

---

## Uso

### 1. Clonar el repositorio

```bash
git clone <url-del-repo>
cd terraform-vapi
```

### 2. Inicializar Terraform (solo la primera vez)

```bash
terraform init
```

### 3. Previsualizar los recursos a crear

```bash
terraform plan -var="proyecto=nombre-del-proyecto"
```

### 4. Crear la infraestructura

```bash
terraform apply -var="proyecto=nombre-del-proyecto"
```

Confirma con `yes` cuando se solicite.

### 5. Obtener las credenciales generadas

```bash
# Ver Access Key ID (visible en el output)
terraform output access_key_id

# Ver Secret Access Key
terraform output secret_access_key
```

---

## Ejemplo

```bash
terraform apply -var="proyecto=proyecto"
```

Genera:
- `proyecto-s3-vapi-grabaciones`
- `proyecto-vapi-s3-policy`
- `proyecto-vapi-s3-user`

---

## Estructura del proyecto

```
terraform-vapi/
├── main.tf          # Recursos: bucket S3, política IAM, usuario IAM
├── variables.tf     # Definición de variables
├── outputs.tf       # Outputs: bucket, usuario y credenciales
├── .gitignore       # Archivos excluidos del repositorio
└── README.md        # Este archivo
```

---

## ⚠️ Seguridad

- Las credenciales de AWS **nunca** se almacenan en los archivos `.tf`.
- El directorio `.terraform/` y el archivo `terraform.tfstate` están excluidos del repositorio via `.gitignore`.
- El `terraform.tfstate` puede contener valores sensibles como el `secret_access_key` — **nunca lo compartas ni lo subas a un repo público**.