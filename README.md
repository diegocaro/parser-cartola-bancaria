# 🔄 Parser para cartolas bancarias chilenas

> Herramientas para convertir cartolas del Banco de Chile al formato CSV compatible con Actual Budget

El **conversor** es un conjunto de utilidades especializadas para procesar y transformar los datos financieros del Banco de Chile, convirtiéndolos en un formato estándar que puede ser importado directamente en aplicaciones de presupuesto como [Actual Budget](https://actualbudget.com/).

---

## ✨ Características

- **Descarga automática** de cartolas bancarias desde el portal web del Banco de Chile
- **Procesamiento de múltiples formatos**:
  - 📄 Archivos TXT de cuentas corrientes y vista
  - 📊 Archivos XLS de tarjetas de crédito (facturados y no facturados)
- **Conversión inteligente** de campos y formatos de fecha
- **Soporte para campos variables** que se ajustan automáticamente al contenido
- **Salida en formato CSV** listo para importar en Actual Budget

---

## 🏦 Productos Soportados del Banco de Chile

### Cuentas de Depósito
- ✅ **Cuenta Corriente** (formato TXT)
- ✅ **Cuenta FAN/Vista** (formato TXT)

### Tarjetas de Crédito
- ✅ **Movimientos Facturados** (formato XLS)
- ✅ **Movimientos No Facturados** (formato XLS)

---

## 🚀 Uso

### 1. Descargar Cartola Bancaria

```bash
# Descargar cartola de cuenta corriente
bank-statement --account cte

# Descargar cartola de cuenta FAN (por defecto)
bank-statement --account fan

# Guardar en archivo
bank-statement --account cte -o mi_cartola.txt
```

**Requisitos de configuración:**
```bash
export BANK_LOGIN_URL="https://portalpersonas.bancochile.cl/persona/"
export BANK_USER="12345678-9"
export BANK_PASSWORD="TuPassword"
```

### 2. Convertir a Formato Actual Budget

```bash
# Convertir archivo TXT de cuenta corriente
convert-to-actual cartola.txt -o movimientos.csv

# Convertir archivo XLS de tarjeta de crédito
convert-to-actual --fields banco_de_chile_tarjeta_credito_facturados_xls tarjeta.xls -o tarjeta.csv

# Pipeline completo: descargar y convertir
bank-statement | convert-to-actual -o actual_import.csv
```

### 3. Opciones de Conversión Disponibles

| Opción de Campo | Descripción | Archivo |
|----------------|-------------|---------|
| `banco_de_chile_cuenta_corriente_txt` | Cuenta corriente/vista (TXT) | `cartola.txt` |
| `banco_de_chile_tarjeta_credito_facturados_xls` | Tarjeta crédito facturados (XLS) | `Mov_Facturado.xls` |
| `banco_de_chile_tarjeta_credito_no_facturados_xls` | Tarjeta crédito no facturados (XLS) | `Saldo_y_Mov_No_Facturado.xls` |

---

## 🛠️ Arquitectura

```
conversor/
├── cli/                           # Scripts de línea de comandos
│   ├── bank_statement.py          # Descarga automática con Selenium
│   └── convert_to_actual.py       # Conversión de formatos
├── parsers/                       # Procesadores de archivos
│   ├── txt.py                     # Parser para archivos de ancho fijo (TXT)
│   └── xls.py                     # Parser para archivos Excel (XLS)
├── field_definitions/             # Definiciones de campos por producto
│   ├── banco_de_chile_cuenta_corriente_txt.json
│   ├── banco_de_chile_tarjeta_credito_facturados_xls.json
│   └── banco_de_chile_tarjeta_credito_no_facturados_xls.json
└── field_def_registry.py         # Registro de definiciones de campos
```

---

## 📋 Formato de Salida

El conversor genera archivos CSV con el siguiente formato estándar para Actual Budget:

```csv
date,amount,description,notes
2025-01-15,-50000,"Pago:Supermercado XYZ","Compras alimentarias"
2025-01-14,150000,"Deposito:Sueldo Enero","Ingreso mensual"
```

### Campos de Salida:
- **`date`**: Fecha en formato YYYY-MM-DD
- **`amount`**: Monto en centavos (negativo = gasto, positivo = ingreso)
- **`description`**: Descripción de la transacción
- **`notes`**: Notas adicionales (copia de la descripción)

---

## ⚙️ Procesamiento 

### Archivos TXT (Cuentas)
- **Ancho fijo**: Cada campo tiene una posición y longitud específica
- **Campos expandibles**: El campo `description` se ajusta automáticamente si el contenido es más largo
- **Tipos de transacción**: Distingue entre abonos (A), cargos (C) y saldos (S)
- **Manejo de signos**: Convierte automáticamente según el tipo de movimiento

### Archivos XLS (Tarjetas)
- **Mapeo de columnas**: Traduce nombres de columnas del banco a campos estándar
- **Detección de pagos**: Identifica automáticamente pagos vs. compras
- **Salto de filas**: Ignora encabezados y metadatos del banco
- **Inversión de signos**: Ajusta signos según el tipo de movimiento


---

## 🚨 Consideraciones Importantes

- **Selenium WebDriver**: Requiere Chrome instalado para la descarga automática
- **Credenciales**: Maneja credenciales bancarias de forma segura (variables de entorno)
- **Formato específico**: Diseñado exclusivamente para el Banco de Chile
- **Actualizaciones del banco**: Los selectores web pueden cambiar y requerir actualización