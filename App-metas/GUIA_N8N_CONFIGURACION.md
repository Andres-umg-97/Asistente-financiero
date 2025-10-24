# 📧 GUÍA DE CONFIGURACIÓN: REPORTE FINANCIERO POR GMAIL CON N8N

## ✅ LO QUE YA ESTÁ HECHO EN LA APP

He implementado completamente la funcionalidad en tu aplicación Android:

1. ✓ **Botón agregado** en el fragmento de Inicio: "Enviar Reporte por Gmail"
2. ✓ **Servicio de integración** (N8nReportService.java) que recopila todos los datos
3. ✓ **Dependencias instaladas** (OkHttp para HTTP requests)
4. ✓ **Diálogo de entrada** para capturar el email del destinatario

---

## 🔧 CONFIGURACIÓN EN N8N

### PASO 1: Crear un nuevo Workflow en n8n

1. Abre tu instancia de n8n (https://app.n8n.cloud o tu instalación local)
2. Crea un nuevo workflow
3. Nómbralo: **"Financial Report Email Sender"**

---

### PASO 2: Agregar Nodos al Workflow

#### **NODO 1: Webhook (Trigger)**

1. Busca y agrega el nodo **"Webhook"**
2. Configura:
   - **HTTP Method**: `POST`
   - **Path**: `financial-report` (o cualquier nombre que prefieras)
   - **Response Mode**: `Last Node`

3. **Guarda la URL del webhook** que te proporciona n8n
   - Ejemplo: `https://tu-instancia.app.n8n.cloud/webhook/financial-report`

#### **🔒 CONFIGURAR SEGURIDAD (RECOMENDADO)**

**Opción 1: SIN Autenticación (más fácil para empezar)**
- Deja **Authentication** en `None`
- ⚠️ Menos seguro, cualquiera con la URL puede enviar datos
- **En N8nReportService.java** comenta la línea del API Key:
  ```java
  // .addHeader("X-API-Key", API_KEY)  // <- Comenta esta línea
  ```

**Opción 2: CON Autenticación (más seguro - RECOMENDADO)**

**Paso A: Crear tu API Key (TÚ LO INVENTAS)**
1. Genera una clave secreta aleatoria. Ejemplos:
   - `FinanzasApp2025_$ecreto`
   - `miClavePersonal123XYZ`
   - O usa un generador online: https://www.uuidgenerator.net/
   
2. **Guarda esta clave**, la usarás en 2 lugares

**Paso B: Configurar en n8n**
1. En el nodo Webhook, configura:
   - **Authentication**: `Header Auth`
   - **Header Name**: `X-API-Key` (déjalo exactamente así)
   - **Header Value**: `tu-clave-secreta-que-inventaste` (pega tu clave aquí)

**Paso C: Configurar en la App Android**
1. Abre el archivo: `N8nReportService.java`
2. En la línea 21, reemplaza:
   ```java
   private static final String API_KEY = "tu-api-key-secreta";
   ```
   Por:
   ```java
   private static final String API_KEY = "tu-clave-secreta-que-inventaste";
   ```
   ⚠️ **Debe ser EXACTAMENTE la misma que pusiste en n8n**

---

#### **NODO 2: Function (Procesar y Formatear Datos)**

1. Agrega un nodo **"Code"** o **"Function"**
2. Pega el siguiente código JavaScript:

```javascript
// Obtener datos del webhook
const data = $input.first().json;

// Formatear gastos
let gastosTexto = '';
if (data.gastos && data.gastos.length > 0) {
  gastosTexto = data.gastos.map(g => 
    `  • ${g.descripcion || 'Sin descripción'}: Q${g.monto.toFixed(2)} - ${g.fecha} [${g.cuenta}]`
  ).join('\n');
} else {
  gastosTexto = '  No hay gastos registrados';
}

// Formatear ahorros
let ahorrosTexto = '';
if (data.ahorros && data.ahorros.length > 0) {
  ahorrosTexto = data.ahorros.map(a => 
    `  • ${a.descripcion || 'Sin descripción'}: Q${a.monto.toFixed(2)} - ${a.fecha} [${a.cuenta}]`
  ).join('\n');
} else {
  ahorrosTexto = '  No hay ahorros registrados';
}

// Formatear metas
let metasTexto = '';
if (data.metas && data.metas.length > 0) {
  metasTexto = data.metas.map(m => {
    const porcentaje = ((m.progreso / m.objetivo) * 100).toFixed(1);
    const barraProgreso = '█'.repeat(Math.floor(porcentaje / 10)) + '░'.repeat(10 - Math.floor(porcentaje / 10));
    return `  • ${m.nombre}\n    Progreso: Q${m.progreso.toFixed(2)} / Q${m.objetivo.toFixed(2)} (${porcentaje}%)\n    ${barraProgreso}\n    Fecha límite: ${m.fechaLimite}`;
  }).join('\n\n');
} else {
  metasTexto = '  No hay metas registradas';
}

// Formatear cuentas
let cuentasTexto = '';
if (data.cuentas && data.cuentas.length > 0) {
  cuentasTexto = data.cuentas.map(c => 
    `  • ${c.nombre}: Q${c.saldo.toFixed(2)}`
  ).join('\n');
} else {
  cuentasTexto = '  No hay cuentas registradas';
}

// Crear el cuerpo del email
const emailBody = `
═══════════════════════════════════════════════════
         💰 REPORTE FINANCIERO DETALLADO 💰
═══════════════════════════════════════════════════

📅 Fecha de generación: ${data.fechaGeneracion}

═══════════════════════════════════════════════════
📊 RESUMEN GENERAL
═══════════════════════════════════════════════════

💵 Saldo Total:         Q${(data.saldoTotal || 0).toFixed(2)}
💸 Total Gastos:        Q${(data.totalGastos || 0).toFixed(2)}
💰 Total Ahorros:       Q${(data.totalAhorros || 0).toFixed(2)}
🎯 Progreso en Metas:   Q${(data.totalMetasProgreso || 0).toFixed(2)} / Q${(data.totalMetasObjetivo || 0).toFixed(2)}
📈 Balance General:     Q${(data.resumen?.balanceGeneral || 0).toFixed(2)}

═══════════════════════════════════════════════════
🏦 CUENTAS (${data.resumen?.numeroCuentas || 0})
═══════════════════════════════════════════════════

${cuentasTexto}

═══════════════════════════════════════════════════
💸 GASTOS (${data.resumen?.numeroGastos || 0})
═══════════════════════════════════════════════════

${gastosTexto}

TOTAL GASTADO: Q${(data.totalGastos || 0).toFixed(2)}

═══════════════════════════════════════════════════
💵 AHORROS (${data.resumen?.numeroAhorros || 0})
═══════════════════════════════════════════════════

${ahorrosTexto}

TOTAL AHORRADO: Q${(data.totalAhorros || 0).toFixed(2)}

═══════════════════════════════════════════════════
🎯 METAS FINANCIERAS (${data.resumen?.numeroMetas || 0})
═══════════════════════════════════════════════════

${metasTexto}

═══════════════════════════════════════════════════

Este reporte fue generado automáticamente desde tu 
aplicación de Gestión Financiera.

¿Necesitas ayuda? Revisa tus finanzas regularmente 
para alcanzar tus objetivos.

═══════════════════════════════════════════════════
`;

return {
  json: {
    email: data.email,
    subject: `📊 Reporte Financiero - ${new Date(data.fechaGeneracion).toLocaleDateString('es-GT')}`,
    body: emailBody,
    rawData: data
  }
};
```

---

#### **NODO 3: Gmail (Enviar Email)**

1. Agrega el nodo **"Gmail"**
2. Configura:
   - **Credential**: Conecta tu cuenta de Gmail
     - Click en "Create New Credential"
     - Autoriza con tu cuenta de Gmail
   - **Resource**: `Message`
   - **Operation**: `Send`
   - **To**: `{{ $json.email }}`
   - **Subject**: `{{ $json.subject }}`
   - **Email Type**: `Text`
   - **Message**: `{{ $json.body }}`

**Alternativa HTML (Opcional)**: Si prefieres un email más bonito con HTML:
   - **Email Type**: `HTML`
   - Usa un template HTML personalizado

---

#### **NODO 4 (OPCIONAL): Guardar en Google Sheets**

Si quieres llevar un registro de todos los reportes enviados:

1. Agrega nodo **"Google Sheets"**
2. Configura:
   - **Resource**: `Sheet`
   - **Operation**: `Append`
   - **Document**: Selecciona tu spreadsheet
   - **Sheet**: Nombre de la hoja
   - **Columns**: 
     - Fecha: `{{ $json.rawData.fechaGeneracion }}`
     - Email: `{{ $json.email }}`
     - Saldo Total: `{{ $json.rawData.saldoTotal }}`
     - Total Gastos: `{{ $json.rawData.totalGastos }}`
     - Total Ahorros: `{{ $json.rawData.totalAhorros }}`

---

### PASO 3: Conectar los Nodos

Conecta en este orden:
```
Webhook → Function → Gmail → (Opcional: Google Sheets)
```

---

### PASO 4: Activar el Workflow

1. Click en el botón **"Active"** en la esquina superior derecha
2. El workflow ahora está escuchando peticiones

---

## 📱 CONFIGURACIÓN FINAL EN LA APP ANDROID

### Actualizar la URL del Webhook

1. Abre el archivo: `N8nReportService.java`
2. En la línea 17, reemplaza:

```java
private static final String N8N_WEBHOOK_URL = "https://TU_URL_N8N_AQUI/webhook/financial-report";
```

Por tu URL real de n8n, ejemplo:

```java
private static final String N8N_WEBHOOK_URL = "https://tuinstancia.app.n8n.cloud/webhook/financial-report";
```

3. (Opcional) Si configuraste autenticación, actualiza también:

```java
private static final String API_KEY = "tu-api-key-secreta-123";
```

---

## 🧪 PROBAR LA INTEGRACIÓN

### Prueba desde n8n:

1. En n8n, click en **"Execute Workflow"** en el nodo Webhook
2. Click en **"Listen for test event"**
3. Desde la app Android, presiona el botón "Enviar Reporte por Gmail"
4. Ingresa tu email
5. Verifica que n8n reciba los datos

### Prueba completa:

1. Asegúrate de tener datos en la app:
   - Al menos una cuenta creada
   - Algunos gastos registrados
   - Ahorros (opcional)
   - Metas (opcional)

2. Presiona el botón **"Enviar Reporte por Gmail"**
3. Ingresa tu email
4. Espera la confirmación
5. Revisa tu bandeja de entrada

---

## 🔒 SEGURIDAD (RECOMENDACIONES)

1. **Usa HTTPS**: Tu URL de n8n debe usar https://
2. **Autenticación**: Implementa el Header Auth en n8n
3. **Validación en n8n**: Agrega un nodo Function al inicio para validar el API Key:

```javascript
const apiKey = $('Webhook').first().headers['x-api-key'];
const validKey = 'tu-api-key-secreta-123';

if (apiKey !== validKey) {
  throw new Error('API Key inválido');
}

return $input.all();
```

---

## 🎨 MEJORAS OPCIONALES

### 1. **Email HTML con Diseño**
Usa un template HTML profesional en lugar de texto plano

### 2. **Adjuntar PDF**
Agrega un nodo para generar PDF con los datos y adjuntarlo al email

### 3. **Notificaciones adicionales**
- Enviar también por WhatsApp (usando Twilio)
- Notificación push en la app
- Guardar en Dropbox/Google Drive

### 4. **Reportes Programados**
En lugar de manual, programa reportes automáticos:
- Diarios, semanales o mensuales
- Usa el nodo "Schedule Trigger" en n8n

### 5. **Dashboard Web**
Crea una página web que muestre los datos en tiempo real

---

## 🐛 SOLUCIÓN DE PROBLEMAS

### Error: "No se puede conectar a n8n"
- ✓ Verifica que la URL del webhook sea correcta
- ✓ Asegúrate que el workflow esté **activo** en n8n
- ✓ Verifica tu conexión a internet

### Error: "API Key inválido"
- ✓ Revisa que el API_KEY en la app coincida con el de n8n
- ✓ Verifica que el header se esté enviando correctamente

### El email no llega
- ✓ Revisa la carpeta de spam
- ✓ Verifica que la cuenta de Gmail esté correctamente conectada en n8n
- ✓ Revisa los logs del workflow en n8n

### Error de permisos de Gmail
- ✓ Reconecta tu cuenta de Gmail en n8n
- ✓ Asegúrate de dar todos los permisos necesarios

---

## 📞 DATOS DE EJEMPLO

Para probar el webhook manualmente en n8n, puedes usar este JSON:

```json
{
  "email": "tu@email.com",
  "fechaGeneracion": "2025-10-23 14:30:00",
  "saldoTotal": 5000.00,
  "totalGastos": 1500.00,
  "totalAhorros": 800.00,
  "cuentas": [
    {"nombre": "Banco Principal", "saldo": 5000.00}
  ],
  "gastos": [
    {"monto": 500, "descripcion": "Supermercado", "fecha": "2025-10-20", "cuenta": "Banco Principal"}
  ],
  "ahorros": [
    {"monto": 800, "descripcion": "Ahorro mensual", "fecha": "2025-10-15", "cuenta": "Banco Principal"}
  ],
  "metas": [
    {"nombre": "Vacaciones", "objetivo": 10000, "progreso": 3000, "fechaLimite": "2025-12-31"}
  ],
  "resumen": {
    "numeroCuentas": 1,
    "numeroGastos": 1,
    "numeroAhorros": 1,
    "numeroMetas": 1,
    "balanceGeneral": 4300
  }
}
```

---

## ✅ CHECKLIST FINAL

- [ ] Workflow creado en n8n
- [ ] Nodos configurados (Webhook, Function, Gmail)
- [ ] Workflow activado
- [ ] URL del webhook copiada
- [ ] URL actualizada en N8nReportService.java
- [ ] App compilada (Gradle Sync exitoso)
- [ ] Prueba realizada con éxito
- [ ] Email recibido correctamente

---

¡Listo! Tu app ahora puede enviar reportes financieros automáticamente por Gmail usando n8n. 🎉

