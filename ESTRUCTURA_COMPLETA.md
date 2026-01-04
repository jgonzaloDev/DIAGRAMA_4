# 📁 ESTRUCTURA COMPLETA DEL PROYECTO

## 🎯 Estructura Actualizada con Base de Datos

```
📁 tu-repositorio/
│
├── 📁 .github/
│   └── 📁 workflows/
│       ├── 📄 apply.yaml              # Deploy infraestructura Terraform
│       ├── 📄 plan.yaml               # Validación infraestructura
│       └── 📄 deploy-database.yaml    # ⭐ Deploy base de datos SQL
│
├── 📁 infra/                          # Terraform - Infraestructura
│   ├── 📄 main.tf                     # Recursos de Azure
│   ├── 📄 variables.tf                # Variables
│   ├── 📄 outputs.tf                  # Outputs
│   ├── 📄 cloud-init-docker.yaml      # Script VM (XFCE + Docker + OTel)
│   ├── 📄 terraform.tfvars            # ⚠️ Valores reales (NO subir)
│   └── 📄 terraform.tfvars.example    # Ejemplo
│
├── 📁 database/                       # ⭐ Scripts SQL
│   ├── 📁 migrations/                 # Scripts de esquema (siempre)
│   │   └── 📄 001_create_tables.sql   # Crea: customers, orders, order_items
│   ├── 📁 seeds/                      # Datos de prueba (solo dev)
│   │   └── 📄 001_seed_data.sql       # 5 customers, 5 orders, items
│   └── 📄 README.md                   # Documentación BD
│
├── 📄 .gitignore                      # Protección de archivos sensibles
├── 📄 GITHUB_SECRETS_NEW.md           # Guía de secretos
└── 📄 README.md                       # Documentación principal
```

## 📊 Flujo de Trabajo

### **1. Deploy de Infraestructura** (apply.yaml)
```bash
# Se ejecuta cuando:
- Push a main
- Manualmente desde GitHub Actions

# Despliega:
✅ Resource Group
✅ Virtual Network + Subnets
✅ VM con Docker + XFCE + OTel Collector + Elasticsearch + Kibana
✅ SQL Server + Database
✅ 2 App Services (Frontend React + Backend Spring Boot)
```

### **2. Deploy de Base de Datos** (deploy-database.yaml)
```bash
# Se ejecuta cuando:
- Push a main con cambios en database/**
- Manualmente desde GitHub Actions

# Ejecuta:
1️⃣ Valida scripts SQL
2️⃣ Se conecta a Azure SQL Server
3️⃣ Ejecuta migraciones (migrations/*.sql)
4️⃣ Ejecuta seeds (seeds/*.sql) - solo en dev
5️⃣ Verifica tablas y datos
```

## 🗄️ Tablas en SQL Server

### **customers**
```sql
- id (BIGINT, PK)
- customer_code (NVARCHAR(20), UNIQUE)
- name, email (UNIQUE), phone
- address, city, country
- status (ACTIVE/INACTIVE)
- created_at, updated_at
```

### **orders**
```sql
- id (BIGINT, PK)
- order_number (NVARCHAR(30), UNIQUE)
- customer_id (FK → customers)
- order_date, total_amount
- status (PENDING/PROCESSING/SHIPPED/COMPLETED)
- notes
- created_at, updated_at
```

### **order_items**
```sql
- id (BIGINT, PK)
- order_id (FK → orders)
- product_name, product_code
- quantity, unit_price, total_price
- created_at
```

### **migrations**
```sql
- id (INT, PK)
- migration (NVARCHAR(255))
- executed_at (tracking de migraciones)
```

## 🔐 Secretos de GitHub Necesarios

Ya los tienes configurados, pero ahora se usan también para la BD:

```
✅ AZURE_CLIENT_ID
✅ AZURE_TENANT_ID
✅ AZURE_SUBSCRIPTION_ID
✅ TF_VAR_RESOURCE_GROUP_NAME
✅ TF_VAR_SQL_SERVER_NAME          # ⭐ Usado por deploy-database
✅ TF_VAR_SQL_DATABASE_NAME        # ⭐ Usado por deploy-database
✅ TF_VAR_SQL_ADMIN_LOGIN          # ⭐ Usado por deploy-database
✅ TF_VAR_SQL_ADMIN_PASSWORD       # ⭐ Usado por deploy-database
... y los demás
```

## 🚀 Comandos para Organizar

```bash
# 1. Crear estructura completa
mkdir -p .github/workflows
mkdir -p infra
mkdir -p database/migrations
mkdir -p database/seeds

# 2. Mover archivos Terraform
mv main.tf infra/
mv variables.tf infra/
mv outputs.tf infra/
mv cloud-init-docker.yaml infra/
mv terraform.tfvars.example infra/

# 3. Mover workflows
mv apply.yaml .github/workflows/
mv plan.yaml .github/workflows/
mv deploy-database.yaml .github/workflows/

# 4. Mover scripts SQL
mv 001_create_tables.sql database/migrations/
mv 001_seed_data.sql database/seeds/

# 5. Crear .gitignore
cat > .gitignore << 'EOF'
**/.terraform/
**/.terraform.lock.hcl
**/terraform.tfstate
**/terraform.tfstate.backup
**/*.tfvars
!**/*.tfvars.example
.vscode/
.DS_Store
*.pem
*.key
EOF

# 6. Verificar estructura
tree -a -I '.git'
```

## ✅ Orden de Ejecución (Primera vez)

### **Paso 1: Deploy Infraestructura**
```bash
# Ejecutar workflow: apply.yaml
# Esto crea TODA la infraestructura incluyendo SQL Server vacío
```

### **Paso 2: Deploy Base de Datos**
```bash
# Ejecutar workflow: deploy-database.yaml
# Esto crea las tablas y datos en el SQL Server
```

## 📋 Verificación Completa

### **Infraestructura:**
```bash
terraform output
# Verás:
# - frontend_url
# - backend_url
# - vm_public_ip
# - vm_kibana_url
# - sql_server_fqdn
```

### **Base de Datos:**
Conectarte a SQL Server y ejecutar:
```sql
-- Ver tablas
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_TYPE = 'BASE TABLE';

-- Ver datos
SELECT * FROM customers;
SELECT * FROM orders;
SELECT * FROM order_items;
```

## 🎨 Arquitectura Visual

```
┌─────────────────────────────────────────────────────────────┐
│                     AZURE CLOUD                             │
│                                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Virtual Network (10.0.0.0/16)                     │    │
│  │                                                     │    │
│  │  ┌──────────────────────────────────────────────┐  │    │
│  │  │ Subnet Integration (10.0.1.0/24)             │  │    │
│  │  │                                               │  │    │
│  │  │  ┌────────────┐      ┌──────────────┐        │  │    │
│  │  │  │ Frontend   │─────▶│ Backend      │───┐    │  │    │
│  │  │  │ (React)    │      │ (SpringBoot) │   │    │  │    │
│  │  │  └────────────┘      └──────────────┘   │    │  │    │
│  │  └─────────────────────────────────────────┼────┘  │    │
│  │                                             │       │    │
│  │  ┌──────────────────────────────────────┐  │       │    │
│  │  │ Subnet VM (10.0.2.0/24)              │  │       │    │
│  │  │                                       │  │       │    │
│  │  │  ┌────────────────────────────────┐  │  │       │    │
│  │  │  │ VM Observability               │◀─┘  │       │    │
│  │  │  │ • XFCE Desktop (RDP 3389)      │     │       │    │
│  │  │  │ • Docker                       │     │       │    │
│  │  │  │ • OTel Collector (4317)        │     │       │    │
│  │  │  │ • Elasticsearch (9200)         │     │       │    │
│  │  │  │ • Kibana (5601)                │     │       │    │
│  │  │  └────────────────────────────────┘     │       │    │
│  │  └──────────────────────────────────────────┘       │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │ SQL Server                                         │◀───┤
│  │ • customers (5 registros)                          │    │
│  │ • orders (5 registros)                             │    │
│  │ • order_items (10+ registros)                      │    │
│  │ • migrations (tracking)                            │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ GitHub Actions
                           ▼
                  ┌─────────────────────┐
                  │ Workflows:          │
                  │ • apply.yaml        │
                  │ • deploy-database   │
                  └─────────────────────┘
```

## 📝 Próximos Pasos

1. ✅ Organiza los archivos según la estructura
2. ✅ Actualiza `.gitignore`
3. ✅ Crea `terraform.tfvars` con tus valores
4. ✅ Commit y push:
   ```bash
   git add .
   git commit -m "feat: add database deployment workflow"
   git push origin main
   ```
5. ✅ Ejecuta workflow **apply.yaml** (infraestructura)
6. ✅ Ejecuta workflow **deploy-database.yaml** (base de datos)
7. ✅ Verifica todo funcionando

## 🔗 Archivos Descargados

Estos son los archivos que debes colocar en tu repositorio:

### **Workflows (.github/workflows/)**
- `apply.yaml`
- `plan.yaml`
- `deploy-database.yaml` ⭐

### **Infraestructura (infra/)**
- `main.tf`
- `variables.tf`
- `outputs.tf`
- `cloud-init-docker.yaml`
- `terraform.tfvars.example`

### **Base de Datos (database/)**
- `migrations/001_create_tables.sql` ⭐
- `seeds/001_seed_data.sql` ⭐
- `README.md` ⭐

### **Documentación (raíz)**
- `GITHUB_SECRETS_NEW.md`
- `README.md`
- `.gitignore`
