# 🧪 GUÍA DE PRUEBA - REPORTE POR GMAIL

## ✅ CHECKLIST PREVIO

Antes de probar, asegúrate de que:
- [x] El código está sincronizado (Gradle Sync completado)
- [x] El permiso de INTERNET está en AndroidManifest.xml
- [x] La URL del webhook está configurada: `https://primary-production-78316.up.railway.app/webhook/resumen_finanzas`
- [ ] Tu workflow de n8n está ACTIVO
- [ ] Tienes datos en la app (cuentas, gastos, ahorros, metas)

---

## 🔧 PASO 1: CONFIGURAR N8N

### 1.1 Crear el Workflow

1. Entra a tu n8n: https://primary-production-78316.up.railway.app
2. Crea un nuevo workflow
3. Nómbralo: **"Reporte Financiero Gmail"**

### 1.2 Agregar Nodo Webhook

1. Agrega el nodo **"Webhook"**
2. Configura:
   - **HTTP Method**: `POST`
   - **Path**: `resumen_finanzas`
   - **Authentication**: `None`
   - **Response Mode**: `Last Node`
3. **Copia la URL del webhook** (debe ser: `https://primary-production-78316.up.railway.app/webhook/resumen_finanzas`)

### 1.3 Agregar Nodo Code (JavaScript)

1. Agrega nodo **"Code"**
2. Conecta: Webhook → Code
3. Pega este código:

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

### 1.4 Agregar Nodo Gmail

1. Agrega nodo **"Gmail"**
2. Conecta: Code → Gmail
3. Configura:
   - **Credential**: Click en "Create New Credential"
     - Autoriza con tu cuenta Gmail
     - Acepta todos los permisos
   - **Resource**: `Message`
   - **Operation**: `Send`
   - **To**: Click en "Expression" y pon: `{{ $json.email }}`
   - **Subject**: Click en "Expression" y pon: `{{ $json.subject }}`
   - **Email Type**: `Text`
   - **Message**: Click en "Expression" y pon: `{{ $json.body }}`

### 1.5 Activar el Workflow

1. Click en el botón **"Active"** (arriba a la derecha)
2. El botón debe ponerse verde
3. Guarda el workflow (Ctrl + S)

---

## 📱 PASO 2: PREPARAR DATOS EN LA APP

Antes de probar, asegúrate de tener datos de prueba:

### Opción A: Si ya tienes datos
- ✓ Continúa al siguiente paso

### Opción B: Crear datos de prueba
1. Abre la app en tu dispositivo/emulador
2. Agrega al menos:
   - 1 cuenta (Ej: "Banco", Q5000.00)
   - 2 gastos (Ej: "Supermercado" Q500, "Gasolina" Q300)
   - 1 ahorro (Ej: "Ahorro mensual" Q800)
   - 1 meta (Ej: "Vacaciones", Objetivo: Q10000, Progreso: Q3000)

---

## 🧪 PASO 3: PROBAR LA FUNCIONALIDAD

### 3.1 Compilar y Ejecutar la App

1. En Android Studio, click en **"Build"** → **"Rebuild Project"**
2. Espera a que termine (1-2 minutos)
3. Click en el botón **"Run"** (▶️ verde)
4. Selecciona tu dispositivo/emulador
5. Espera a que la app se instale

### 3.2 Enviar el Reporte

1. **Abre la app** en tu dispositivo
2. Ve a la pantalla de **"Inicio"**
3. Busca el botón **"Enviar Reporte por Gmail"**
4. **Haz click** en el botón
5. Aparecerá un diálogo pidiendo el email
6. **Ingresa tu correo** (ej: `tucorreo@gmail.com`)
7. Click en **"Enviar"**

### 3.3 Verificar el Envío

**En la App:**
- Deberías ver un Toast (mensaje temporal) que dice:
  - ⏳ "Preparando reporte..." (al inicio)
  - ✓ "Reporte enviado correctamente a tucorreo@gmail.com" (si fue exitoso)
  - ✗ "Error de conexión..." (si hubo un problema)

**En n8n:**
1. Ve a tu workflow en n8n
2. Click en **"Executions"** (lado derecho)
3. Deberías ver una nueva ejecución con estado "Success" (verde)
4. Click en la ejecución para ver los datos recibidos

**En Gmail:**
1. Abre tu bandeja de entrada
2. Busca un email con asunto: **"📊 Reporte Financiero - [fecha]"**
3. Ábrelo y verifica que contiene todos tus datos

---

## 🐛 SOLUCIÓN DE PROBLEMAS

### ❌ Error: "Error de conexión"

**Causas posibles:**
1. **No hay internet**: Verifica tu conexión WiFi/datos
2. **Workflow no está activo**: Ve a n8n y activa el workflow
3. **URL incorrecta**: Verifica que la URL en `N8nReportService.java` sea exacta

**Solución:**
```java
// Verifica que esta línea tenga la URL correcta:
private static final String N8N_WEBHOOK_URL = "https://primary-production-78316.up.railway.app/webhook/resumen_finanzas";
```

### ❌ Error: "Error del servidor: 404"

**Causa:** El webhook no existe o el path es incorrecto

**Solución:**
1. Ve a n8n → tu workflow → nodo Webhook
2. Verifica que el **Path** sea: `resumen_finanzas`
3. Verifica que el workflow esté **activo** (botón verde)

### ❌ Error: "Error del servidor: 500"

**Causa:** Error en el código JavaScript de n8n

**Solución:**
1. Ve a n8n → Executions
2. Click en la ejecución fallida
3. Mira el error en el nodo Code
4. Revisa que hayas copiado bien el código JavaScript

### ❌ El email no llega

**Posibles causas:**

1. **Gmail no está conectado en n8n**
   - Ve al nodo Gmail → Credential → Reconecta tu cuenta

2. **El email está en spam**
   - Revisa tu carpeta de spam/correo no deseado

3. **Email incorrecto**
   - Verifica que escribiste bien tu correo

4. **Error en el nodo Gmail**
   - Ve a n8n → Executions → Mira si el nodo Gmail tiene error rojo

### ⚠️ La app no compila

**Causa:** Gradle no sincronizó correctamente

**Solución:**
1. File → Invalidate Caches → Invalidate and Restart
2. Espera a que Android Studio reinicie
3. Vuelve a hacer Gradle Sync

---

## 📊 VERIFICAR LOS DATOS ENVIADOS

### Ver JSON enviado en Logcat (Android Studio):

1. Mientras la app está corriendo, abre **Logcat** (abajo)
2. Filtra por: `N8nReportService`
3. Deberías ver logs como:
   ```
   D/N8nReportService: Reporte enviado exitosamente
   ```

### Ver JSON recibido en n8n:

1. Ve a n8n → Executions
2. Click en la última ejecución
3. Click en el nodo **Webhook**
4. Verás el JSON completo enviado por la app:

```json
{
  "email": "tucorreo@gmail.com",
  "fechaGeneracion": "2025-10-23 14:30:00",
  "saldoTotal": 5000.00,
  "totalGastos": 1500.00,
  "totalAhorros": 800.00,
  "cuentas": [...],
  "gastos": [...],
  "ahorros": [...],
  "metas": [...],
  "resumen": {...}
}
```

---

## 🎯 PRUEBA EXITOSA SI...

✅ La app muestra: "Reporte enviado correctamente a..."
✅ En n8n ves la ejecución en verde (Success)
✅ Recibes el email en tu bandeja de entrada
✅ El email contiene todos tus datos financieros correctamente formateados

---

## 📸 CAPTURAS DE PANTALLA RECOMENDADAS

Toma capturas de:
1. ✓ Botón "Enviar Reporte por Gmail" en la app
2. ✓ Diálogo de ingreso de email
3. ✓ Toast de confirmación
4. ✓ Ejecución exitosa en n8n (verde)
5. ✓ Email recibido en Gmail

---

## 🚀 SIGUIENTE NIVEL

Una vez que funcione, puedes mejorar:

### 📅 Reportes Automáticos Programados
- Usa el nodo **"Schedule Trigger"** en n8n
- Programa reportes diarios/semanales/mensuales
- En lugar del webhook, usa un trigger de tiempo

### 📊 Email HTML con Gráficas
- Cambia el email de texto plano a HTML
- Agrega colores, tablas y diseño profesional
- Usa Chart.js para agregar gráficas visuales

### 📎 Adjuntar PDF
- Agrega un nodo para generar PDF
- Adjúntalo al email con el reporte detallado

### 💾 Guardar en Google Sheets
- Agrega nodo de Google Sheets
- Guarda cada reporte en una hoja de cálculo
- Lleva un historial completo

---

¡Listo para probar! 🎉

