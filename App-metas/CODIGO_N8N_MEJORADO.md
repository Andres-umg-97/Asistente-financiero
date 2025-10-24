# 🔧 CÓDIGO N8N MEJORADO - COPIA ESTO

## IMPORTANTE: Usa este código en el nodo Code de n8n

Este código es más robusto y mostrará exactamente qué datos recibe.

```javascript
// Obtener todos los datos del webhook
const data = $input.first().json;

// Log para debugging - ver qué llega
console.log("Datos recibidos:", JSON.stringify(data, null, 2));

// Función auxiliar para valores seguros
function safe(value, defaultValue = 0) {
  return value !== undefined && value !== null ? value : defaultValue;
}

function safeArray(arr) {
  return Array.isArray(arr) ? arr : [];
}

// Extraer datos con valores por defecto
const email = data.email || 'sin-email@ejemplo.com';
const fechaGeneracion = data.fechaGeneracion || new Date().toLocaleString('es-GT');
const saldoTotal = safe(data.saldoTotal, 0);
const totalGastos = safe(data.totalGastos, 0);
const totalAhorros = safe(data.totalAhorros, 0);
const totalMetasProgreso = safe(data.totalMetasProgreso, 0);
const totalMetasObjetivo = safe(data.totalMetasObjetivo, 0);

const cuentas = safeArray(data.cuentas);
const gastos = safeArray(data.gastos);
const ahorros = safeArray(data.ahorros);
const metas = safeArray(data.metas);

const resumen = data.resumen || {
  numeroCuentas: cuentas.length,
  numeroGastos: gastos.length,
  numeroAhorros: ahorros.length,
  numeroMetas: metas.length,
  balanceGeneral: saldoTotal - totalGastos + totalAhorros
};

// Formatear cuentas
let cuentasTexto = '';
if (cuentas.length > 0) {
  cuentasTexto = cuentas.map(c => 
    `  • ${c.nombre || 'Cuenta'}: Q${(c.saldo || 0).toFixed(2)}`
  ).join('\n');
} else {
  cuentasTexto = '  No hay cuentas registradas';
}

// Formatear gastos
let gastosTexto = '';
if (gastos.length > 0) {
  gastosTexto = gastos.map(g => 
    `  • ${g.descripcion || 'Sin descripción'}: Q${(g.monto || 0).toFixed(2)} - ${g.fecha || 'Sin fecha'} [${g.cuenta || 'Sin cuenta'}]`
  ).join('\n');
} else {
  gastosTexto = '  No hay gastos registrados';
}

// Formatear ahorros
let ahorrosTexto = '';
if (ahorros.length > 0) {
  ahorrosTexto = ahorros.map(a => 
    `  • ${a.descripcion || 'Sin descripción'}: Q${(a.monto || 0).toFixed(2)} - ${a.fecha || 'Sin fecha'} [${a.cuenta || 'Sin cuenta'}]`
  ).join('\n');
} else {
  ahorrosTexto = '  No hay ahorros registrados';
}

// Formatear metas
let metasTexto = '';
if (metas.length > 0) {
  metasTexto = metas.map(m => {
    const objetivo = m.objetivo || 1;
    const progreso = m.progreso || 0;
    const porcentaje = ((progreso / objetivo) * 100).toFixed(1);
    const barrasLlenas = Math.floor(parseFloat(porcentaje) / 10);
    const barraProgreso = '█'.repeat(barrasLlenas) + '░'.repeat(10 - barrasLlenas);
    return `  • ${m.nombre || 'Meta sin nombre'}
    Progreso: Q${progreso.toFixed(2)} / Q${objetivo.toFixed(2)} (${porcentaje}%)
    [${barraProgreso}]
    Fecha límite: ${m.fechaLimite || 'Sin fecha'}`;
  }).join('\n\n');
} else {
  metasTexto = '  No hay metas registradas';
}

// Crear el cuerpo del email
const emailBody = `
═══════════════════════════════════════════════════
         💰 REPORTE FINANCIERO DETALLADO 💰
═══════════════════════════════════════════════════

📅 Fecha de generación: ${fechaGeneracion}

═══════════════════════════════════════════════════
📊 RESUMEN GENERAL
═══════════════════════════════════════════════════

💵 Saldo Total:         Q${saldoTotal.toFixed(2)}
💸 Total Gastos:        Q${totalGastos.toFixed(2)}
💰 Total Ahorros:       Q${totalAhorros.toFixed(2)}
🎯 Progreso en Metas:   Q${totalMetasProgreso.toFixed(2)} / Q${totalMetasObjetivo.toFixed(2)}
📈 Balance General:     Q${(resumen.balanceGeneral || 0).toFixed(2)}

═══════════════════════════════════════════════════
🏦 CUENTAS (${resumen.numeroCuentas || 0})
═══════════════════════════════════════════════════

${cuentasTexto}

═══════════════════════════════════════════════════
💸 GASTOS (${resumen.numeroGastos || 0})
═══════════════════════════════════════════════════

${gastosTexto}

TOTAL GASTADO: Q${totalGastos.toFixed(2)}

═══════════════════════════════════════════════════
💵 AHORROS (${resumen.numeroAhorros || 0})
═══════════════════════════════════════════════════

${ahorrosTexto}

TOTAL AHORRADO: Q${totalAhorros.toFixed(2)}

═══════════════════════════════════════════════════
🎯 METAS FINANCIERAS (${resumen.numeroMetas || 0})
═══════════════════════════════════════════════════

${metasTexto}

═══════════════════════════════════════════════════

Este reporte fue generado automáticamente desde tu 
aplicación de Gestión Financiera.

📱 App: Gestión Financiera Personal
🏦 Cuentas activas: ${resumen.numeroCuentas || 0}
💰 Saldo disponible: Q${saldoTotal.toFixed(2)}

═══════════════════════════════════════════════════
`;

// Log del email generado
console.log("Email generado con éxito");

return {
  json: {
    email: email,
    subject: `📊 Reporte Financiero - ${new Date().toLocaleDateString('es-GT')}`,
    body: emailBody,
    rawData: data
  }
};
```

---

## 📋 INSTRUCCIONES PARA ACTUALIZAR N8N

### Paso 1: Ve a tu workflow en n8n

### Paso 2: Abre el nodo "Code"

### Paso 3: BORRA TODO el código que tenga

### Paso 4: COPIA Y PEGA el código de arriba (todo el bloque JavaScript)

### Paso 5: Guarda el workflow (Ctrl + S)

### Paso 6: Asegúrate que esté ACTIVO (botón verde)

---

## 🧪 CÓMO PROBAR SI ESTÁ FUNCIONANDO

### Método 1: Prueba Manual en n8n (SIN la app)

1. En n8n, en el nodo **Webhook**, click en "Listen for test event"
2. Usa esta herramienta para enviar datos de prueba: https://reqbin.com/
3. O usa este comando en PowerShell (desde tu PC):

```powershell
$body = @{
    email = "tu-email@gmail.com"
    fechaGeneracion = "2025-10-23 15:30:00"
    saldoTotal = 5000.50
    totalGastos = 1500.75
    totalAhorros = 800.25
    totalMetasProgreso = 3000
    totalMetasObjetivo = 10000
    cuentas = @(
        @{ nombre = "Banco Principal"; saldo = 5000.50 }
    )
    gastos = @(
        @{ monto = 500.50; descripcion = "Supermercado"; fecha = "2025-10-20"; cuenta = "Banco Principal" }
        @{ monto = 300.25; descripcion = "Gasolina"; fecha = "2025-10-19"; cuenta = "Banco Principal" }
    )
    ahorros = @(
        @{ monto = 800.25; descripcion = "Ahorro mensual"; fecha = "2025-10-15"; cuenta = "Banco Principal" }
    )
    metas = @(
        @{ nombre = "Vacaciones"; objetivo = 10000; progreso = 3000; fechaLimite = "2025-12-31" }
    )
    resumen = @{
        numeroCuentas = 1
        numeroGastos = 2
        numeroAhorros = 1
        numeroMetas = 1
        balanceGeneral = 4300
    }
} | ConvertTo-Json -Depth 10

Invoke-RestMethod -Uri "https://primary-production-78316.up.railway.app/webhook/resumen_finanzas" -Method POST -Body $body -ContentType "application/json"
```

Si esto funciona, verás:
- ✅ Ejecución verde en n8n
- ✅ Email recibido con todos los datos

### Método 2: Desde la App Android

1. **Compila la app de nuevo** (para que incluya el log que agregué)
2. **Ejecuta la app** en tu dispositivo/emulador
3. **Abre Logcat** en Android Studio (ventana inferior)
4. **Filtra por:** `N8nReportService`
5. **Presiona el botón** "Enviar Reporte por Gmail"
6. **Ingresa tu email**
7. **Mira en Logcat** - deberías ver el JSON completo que se envía

Copia ese JSON y compártelo conmigo si sigue sin funcionar.

---

## 🔍 VERIFICACIÓN EN N8N

Después de enviar desde la app:

1. Ve a n8n → **Executions** (panel derecho)
2. Click en la última ejecución
3. Click en el nodo **Webhook**
4. Deberías ver el JSON recibido
5. Click en el nodo **Code**
6. Deberías ver el email formateado
7. Click en el nodo **Gmail**
8. Deberías ver "Success"

---

## ❓ SI SIGUE SIN FUNCIONAR

Compárteme:
1. ✓ El JSON que aparece en Logcat (lo que envía la app)
2. ✓ El JSON que aparece en n8n Webhook (lo que recibe n8n)
3. ✓ Captura de pantalla del error en n8n (si hay)

Así podré ver exactamente qué está pasando.

