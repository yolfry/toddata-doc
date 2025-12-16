# 📄 Documentación del Esquema de Almacenamiento – `app.toddata`

## 🌐 Reglas Globales

1. **Formato de IDs:** Todos los identificadores deben ser **UUID** (*Universally Unique Identifier*).  
   **Propósito:** Garantiza unicidad global, seguridad y trazabilidad legal.
2. **Identidad del Negocio:** Todo registro debe incluir la propiedad **`businessId`** para identificar quién emitió los datos.

---

## 📂 Espacio de Nombres Raíz (Root Namespace)

El sistema utiliza aislamiento de datos por negocio (*Multitenancy*).

**Ruta Base:**
```
app.toddata.<businessId>
```

---

## 🏢 Business (Empresa)

Almacena la configuración, identidad fiscal y datos de contacto del negocio.

### 📍 Clave de Almacenamiento
```
app.toddata.<businessId>.business
```

### 💻 Interfaz: `Business`
```typescript
interface Business {
  id: string;              // UUID: Identificador único del negocio.
  RNC: string;             // Registro Nacional de Contribuyentes (ID Fiscal).
  commercialName: string;  // Nombre comercial o Razón Social.
  category: string;        // Categoría del negocio (ej. Farmacia, Colmado).
  phoneNumber: string;     // Número de teléfono principal.
  address: string;         // Dirección física del establecimiento.
  ownerCEDULA: string;     // Cédula de identidad del propietario legal.
  logoUrl: string;         // URL pública del logo de la empresa.
}
```

---

## 📦 Products (Productos)

Gestión del inventario. Se divide en una lista de índices y los datos detallados.

### 📍 Lista de Productos
Almacena solo los IDs para iteraciones rápidas.

```typescript
// Ruta: app.toddata.<businessId>.products
string[] // Ejemplo: ["uuid-prod-1", "uuid-prod-2"]
```

### 📍 Datos del Producto
```
app.toddata.<businessId>.products.<productId>
```

### 💻 Interfaz: `Product`
```typescript
interface Product {
  id: string;          // UUID: Identificador único del producto.
  businessId: string;  // UUID: ID del negocio propietario.
  name: string;        // Nombre del producto.
  barcode: string;     // Código de barras (EAN, UPC, etc.).
  description: string; // Descripción detallada o notas.
  imageUrl: string;    // URL de la imagen del producto.
  price: number;       // Precio de venta unitario.
}
```

---

## 👥 Clients (Clientes)

Gestión de clientes.  
**Nota:** Se utiliza la **CÉDULA** como clave principal (PK) en lugar de un UUID para facilitar búsquedas y evitar duplicados.

### 📍 Lista de Clientes
```typescript
// Ruta: app.toddata.<businessId>.clients
string[] // Ejemplo: ["00100000001", "40200000002"]
```

### 📍 Datos del Cliente
```
app.toddata.<businessId>.clients.<CEDULA>
```

### 💻 Interfaz: `Client`
```typescript
interface Client {
  CEDULA: string;          // Documento de identidad (Clave Única).
  businessId: string;      // UUID: ID del negocio asociado.
  firstName: string;       // Nombres.
  lastName: string;        // Apellidos.
  cedulaFrontUrl: string;  // URL de la foto frontal de la cédula.
  cedulaBackUrl: string;   // URL de la foto trasera de la cédula.
  signatureUrl: string;    // URL de la firma digitalizada.
  profileUrl: string;      // URL de la foto de perfil.
  phoneNumber: string;     // Número de contacto.
  address: string;         // Dirección física.
}
```

---

## 🧾 Invoices (Facturas)

Registro de ventas y transacciones.

### 📍 Lista de Facturas
```typescript
// Ruta: app.toddata.<businessId>.invoices
string[] // Ejemplo: ["uuid-inv-1", "uuid-inv-2"]
```

### 📍 Datos de la Factura
```
app.toddata.<businessId>.invoices.<invoiceId>
```

### 💻 Interfaces: `InvoiceProduct` y `Invoice`

#### Detalle del Producto en Factura
Se guardan los datos históricos (*snapshot*) por si el producto cambia en el futuro.

```typescript
interface InvoiceProduct {
  productId: string; // UUID: Referencia al producto original.
  name: string;      // Nombre del producto al momento de la venta.
  unitPrice: number; // Precio al momento de la venta.
  quantity: number;  // Cantidad vendida.
}
```

## 🧾 Invoices (Facturas)

### 💻 Interfaz: `InvoicePayment`
```typescript
interface InvoicePayment {
  id: string;            // UUID
  date: string;          // TIMESTAMP (ISO 8601)
  totalPayment: number;  // Monto pagado
  plazoNumber: number;   // Número de plazo/cuota
  signatureUrl: string;  // Firma del cliente
  clientCEDULA: string;  // Cédula del cliente
}
```

### 💻 Interfaz: `Invoice`
```typescript
interface Invoice {
  id: string;
  businessId: string;
  dateIssued: string;
  paymentTermDays: number;
  dueDate: string;
  products: InvoiceProduct[];
  payments: InvoicePayment[];
  details: string;
  clientCEDULA: string;
  signatureUrl: string;
  invoiceImageUrl: string;
  totalAmount: number;
}
```
