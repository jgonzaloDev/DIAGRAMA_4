# 📊 Database Scripts

Scripts SQL para Azure SQL Server del proyecto.

## 📁 Estructura

```
database/
├── migrations/          # Scripts de creación/modificación de esquema
│   └── 001_create_tables.sql
└── seeds/              # Datos iniciales (solo en desarrollo)
    └── 001_seed_data.sql
```

## 🔄 Cómo funciona

### **1. Migraciones (migrations/)**
- Se ejecutan **siempre** (desarrollo y producción)
- Crean/modifican el esquema de la base de datos
- Se ejecutan en orden alfabético
- Son **idempotentes** (se pueden ejecutar múltiples veces sin error)

### **2. Seeds (seeds/)**
- Se ejecutan **solo en desarrollo** (NO en main/producción)
- Insertan datos de prueba
- También son **idempotentes**

## 🚀 Ejecución

### **Automática:**
El workflow se ejecuta automáticamente cuando:
- Haces push a `main` con cambios en `database/**`
- Ejecutas manualmente desde GitHub Actions

### **Manual:**
1. Ve a GitHub → **Actions**
2. Selecciona **"Deploy Database to Azure SQL Server"**
3. Click en **"Run workflow"**
4. Selecciona la rama (main o develop)
5. Click en **"Run workflow"**

## 📋 Tablas creadas

### **customers**
- `id` (BIGINT, PK, IDENTITY)
- `customer_code` (NVARCHAR(20), UNIQUE)
- `name` (NVARCHAR(150))
- `email` (NVARCHAR(100), UNIQUE)
- `phone` (NVARCHAR(20))
- `address`, `city`, `country`
- `status` (NVARCHAR(20), default: 'ACTIVE')
- `created_at`, `updated_at` (DATETIME2)

### **orders**
- `id` (BIGINT, PK, IDENTITY)
- `order_number` (NVARCHAR(30), UNIQUE)
- `customer_id` (BIGINT, FK → customers)
- `order_date` (DATETIME2)
- `total_amount` (DECIMAL(18,2))
- `status` (NVARCHAR(20), default: 'PENDING')
- `notes` (NVARCHAR(500))
- `created_at`, `updated_at` (DATETIME2)

### **order_items**
- `id` (BIGINT, PK, IDENTITY)
- `order_id` (BIGINT, FK → orders)
- `product_name` (NVARCHAR(200))
- `product_code` (NVARCHAR(50))
- `quantity` (INT)
- `unit_price` (DECIMAL(18,2))
- `total_price` (DECIMAL(18,2))
- `created_at` (DATETIME2)

### **migrations**
- `id` (INT, PK, IDENTITY)
- `migration` (NVARCHAR(255))
- `executed_at` (DATETIME2)

## 📊 Datos de ejemplo (seeds)

- **5 Customers**: Acme Corp, Tech Solutions, Global Traders, etc.
- **5 Orders**: Con diferentes estados (COMPLETED, SHIPPED, PROCESSING, PENDING)
- **10+ Order Items**: Productos variados

## ✅ Verificación

Después de ejecutar el workflow, verás:

```sql
-- Tablas creadas
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_TYPE = 'BASE TABLE';

-- Conteo de registros
SELECT 'customers' as tabla, COUNT(*) FROM customers
UNION ALL
SELECT 'orders', COUNT(*) FROM orders
UNION ALL
SELECT 'order_items', COUNT(*) FROM order_items;
```

## 🔐 Seguridad

- El workflow usa **OIDC** para autenticarse en Azure
- Las credenciales de SQL se obtienen desde **GitHub Secrets**
- Se crea una regla de firewall **temporal** para el runner
- La regla se **elimina automáticamente** al finalizar

## 📝 Agregar nuevas migraciones

1. Crea un archivo en `database/migrations/`:
   ```
   002_add_products_table.sql
   003_add_shipping_info.sql
   ```

2. Asegúrate de que sea **idempotente**:
   ```sql
   IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'products')
   BEGIN
       CREATE TABLE [dbo].[products]...
   END
   ```

3. Commit y push:
   ```bash
   git add database/
   git commit -m "feat: add products table migration"
   git push origin main
   ```

## ⚠️ Consideraciones

- Las migraciones NO se deshacen automáticamente
- Siempre revisa los scripts antes de ejecutarlos en producción
- Los seeds NO se ejecutan en producción (rama main)
- Usa transacciones para cambios complejos

## 🔗 Relacionado

- **Workflow**: `.github/workflows/deploy-database.yaml`
- **SQL Server**: Configurado en Terraform (`infra/main.tf`)
- **Secretos necesarios**: Ver `GITHUB_SECRETS_NEW.md`
