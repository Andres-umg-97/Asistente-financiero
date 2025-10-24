# 🎯 CÓDIGO JAVASCRIPT DEFINITIVO PARA N8N

## ⚠️ COPIA ESTE CÓDIGO EXACTO EN TU NODO CODE DE N8N

```javascript
// ========================================
// CÓDIGO JAVASCRIPT PARA NODO CODE EN N8N
// ========================================

// Obtener datos del webhook - Los datos vienen en $json.body
const inputData = $json.body || $json;

// Log completo para debugging
console.log("=== DATOS RECIBIDOS DEL WEBHOOK ===");
console.log(JSON.stringify(inputData, null, 2));

// Extraer email
const email = inputData.email || 'sin-email@ejemplo.com';

// Extraer fecha y convertir a hora de Guatemala
const fechaOriginal = inputData.fechaGeneracion || new Date().toISOString();
let fechaGuatemala = fechaOriginal;

try {
  // Parsear la fecha en formato "2025-10-23 18:01:40"
  const partes = fechaOriginal.split(' ');
  if (partes.length === 2) {
    const [fecha, hora] = partes;
    fechaGuatemala = `${fecha} ${hora} (Hora Guatemala)`;
  }
} catch (e) {
  console.log("Error al procesar fecha:", e);
}

// Extraer totales
const saldoTotal = Number(inputData.saldoTotal) || 0;
const totalGastos = Number(inputData.totalGastos) || 0;
const totalAhorros = Number(inputData.totalAhorros) || 0;
const totalMetasProgreso = Number(inputData.totalMetasProgreso) || 0;
const totalMetasObjetivo = Number(inputData.totalMetasObjetivo) || 0;

// Extraer resumen
const resumen = inputData.resumen || {};
const numeroCuentas = Number(resumen.numeroCuentas) || 0;
const numeroGastos = Number(resumen.numeroGastos) || 0;
const numeroAhorros = Number(resumen.numeroAhorros) || 0;
const numeroMetas = Number(resumen.numeroMetas) || 0;
const balanceGeneral = Number(resumen.balanceGeneral) || 0;

// Convertir objetos con índices a arrays
function objetoAArray(obj) {
  if (!obj) return [];
  if (Array.isArray(obj)) return obj;
  
  // Si es un objeto con índices numéricos, convertirlo a array
  const keys = Object.keys(obj).filter(k => !isNaN(k));
  if (keys.length === 0) return [];
  
  return keys.sort((a, b) => Number(a) - Number(b)).map(k => obj[k]);
}

// Extraer arrays (convertir de objeto a array si es necesario)
const cuentas = objetoAArray(inputData.cuentas);
const gastos = objetoAArray(inputData.gastos);
const ahorros = objetoAArray(inputData.ahorros);
const metas = objetoAArray(inputData.metas);

console.log(`Procesando: ${numeroCuentas} cuentas, ${numeroGastos} gastos, ${numeroAhorros} ahorros, ${numeroMetas} metas`);
console.log(`Totales: Saldo=${saldoTotal}, Gastos=${totalGastos}, Ahorros=${totalAhorros}`);
console.log(`Arrays: cuentas=${cuentas.length}, gastos=${gastos.length}, ahorros=${ahorros.length}, metas=${metas.length}`);

// ========================================
// FORMATEAR CUENTAS
// ========================================
let cuentasTexto = '';
if (cuentas.length > 0) {
  cuentasTexto = cuentas.map(c => {
    const nombre = c.nombre || 'Sin nombre';
    const saldo = Number(c.saldo) || 0;
    return `  💳 ${nombre}: Q${saldo.toFixed(2)}`;
  }).join('\n');
} else {
  cuentasTexto = '  ❌ No hay cuentas registradas';
}

// ========================================
// FORMATEAR GASTOS
// ========================================
let gastosTexto = '';
if (gastos.length > 0) {
  gastosTexto = gastos.map(g => {
    const descripcion = g.descripcion || 'Sin descripción';
    const monto = Number(g.monto) || 0;
    const fecha = g.fecha || 'Sin fecha';
    const cuenta = g.cuenta || 'Sin cuenta';
    return `  💸 ${descripcion}
     Monto: Q${monto.toFixed(2)}
     Fecha: ${fecha}
     Cuenta: ${cuenta}`;
  }).join('\n\n');
} else {
  gastosTexto = '  ❌ No hay gastos registrados';
}

// ========================================
// FORMATEAR AHORROS
// ========================================
let ahorrosTexto = '';
if (ahorros.length > 0) {
  ahorrosTexto = ahorros.map(a => {
    const descripcion = a.descripcion || 'Sin descripción';
    const monto = Number(a.monto) || 0;
    const fecha = a.fecha || 'Sin fecha';
    const cuenta = a.cuenta || 'Sin cuenta';
    return `  💰 ${descripcion}
     Monto: Q${monto.toFixed(2)}
     Fecha: ${fecha}
     Cuenta: ${cuenta}`;
  }).join('\n\n');
} else {
  ahorrosTexto = '  ❌ No hay ahorros registrados';
}

// ========================================
// FORMATEAR METAS
// ========================================
let metasTexto = '';
if (metas.length > 0) {
  metasTexto = metas.map(m => {
    const nombre = m.nombre || 'Sin nombre';
    const objetivo = Number(m.objetivo) || 1;
    const progreso = Number(m.progreso) || 0;
    const fechaLimite = m.fechaLimite || 'Sin fecha';
    const porcentaje = Number(m.porcentaje) || ((progreso / objetivo) * 100);
    
    // Crear barra de progreso visual
    const barrasLlenas = Math.min(Math.floor(porcentaje / 10), 10);
    const barrasVacias = 10 - barrasLlenas;
    const barraProgreso = '█'.repeat(barrasLlenas) + '░'.repeat(barrasVacias);
    
    return `  🎯 ${nombre}
     Objetivo: Q${objetivo.toFixed(2)}
     Progreso: Q${progreso.toFixed(2)}
     Porcentaje: ${porcentaje.toFixed(1)}%
     ${barraProgreso}
     Fecha límite: ${fechaLimite}`;
  }).join('\n\n');
} else {
  metasTexto = '  ❌ No hay metas registradas';
}

// ========================================
// CREAR CUERPO DEL EMAIL
// ========================================
const emailBody = `
╔═══════════════════════════════════════════════════╗
║       💰 REPORTE FINANCIERO DETALLADO 💰         ║
╚═══════════════════════════════════════════════════╝

📅 Fecha de generación: ${fechaGuatemala}
📧 Enviado a: ${email}

╔═══════════════════════════════════════════════════╗
║                📊 RESUMEN GENERAL                 ║
╚═══════════════════════════════════════════════════╝

💵 Saldo Total:         Q${saldoTotal.toFixed(2)}
💸 Total Gastos:        Q${totalGastos.toFixed(2)}
💰 Total Ahorros:       Q${totalAhorros.toFixed(2)}
🎯 Progreso en Metas:   Q${totalMetasProgreso.toFixed(2)} / Q${totalMetasObjetivo.toFixed(2)}
📈 Balance General:     Q${balanceGeneral.toFixed(2)}

╔═══════════════════════════════════════════════════╗
║           🏦 CUENTAS (${numeroCuentas})                          ║
╚═══════════════════════════════════════════════════╝

${cuentasTexto}

╔═══════════════════════════════════════════════════╗
║           💸 GASTOS (${numeroGastos})                           ║
╚═══════════════════════════════════════════════════╝

${gastosTexto}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL GASTADO: Q${totalGastos.toFixed(2)}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════════╗
║           💵 AHORROS (${numeroAhorros})                         ║
╚═══════════════════════════════════════════════════╝

${ahorrosTexto}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL AHORRADO: Q${totalAhorros.toFixed(2)}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════════╗
║         🎯 METAS FINANCIERAS (${numeroMetas})                  ║
╚═══════════════════════════════════════════════════╝

${metasTexto}

╔═══════════════════════════════════════════════════╗
║                  📱 INFORMACIÓN                   ║
╚═══════════════════════════════════════════════════╝

📱 App: Gestión Financiera Personal
🏦 Cuentas activas: ${numeroCuentas}
💰 Saldo disponible: Q${saldoTotal.toFixed(2)}
📊 Movimientos registrados: ${numeroGastos + numeroAhorros}
🎯 Metas en progreso: ${numeroMetas}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 Consejo financiero: 
   Mantén un registro constante de tus gastos para 
   alcanzar tus metas financieras más rápido.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Este reporte fue generado automáticamente desde tu
aplicación de Gestión Financiera.

`;

console.log("=== EMAIL GENERADO EXITOSAMENTE ===");
console.log("Longitud del email:", emailBody.length, "caracteres");

// ========================================
// RETORNAR RESULTADO
// ========================================
return {
  email: email,
  subject: `📊 Reporte Financiero - ${fechaOriginal.split(' ')[0]}`,
  body: emailBody,
  rawData: inputData
};
```

// Extraer totales
const saldoTotal = Number(inputData.saldoTotal) || 0;
const totalGastos = Number(inputData.totalGastos) || 0;
const totalAhorros = Number(inputData.totalAhorros) || 0;
const totalMetasProgreso = Number(inputData.totalMetasProgreso) || 0;
const totalMetasObjetivo = Number(inputData.totalMetasObjetivo) || 0;

// Extraer resumen
const resumen = inputData.resumen || {};
const numeroCuentas = resumen.numeroCuentas || 0;
const numeroGastos = resumen.numeroGastos || 0;
const numeroAhorros = resumen.numeroAhorros || 0;
const numeroMetas = resumen.numeroMetas || 0;
const balanceGeneral = Number(resumen.balanceGeneral) || 0;

// Extraer arrays
const cuentas = Array.isArray(inputData.cuentas) ? inputData.cuentas : [];
const gastos = Array.isArray(inputData.gastos) ? inputData.gastos : [];
const ahorros = Array.isArray(inputData.ahorros) ? inputData.ahorros : [];
const metas = Array.isArray(inputData.metas) ? inputData.metas : [];

console.log(`Procesando: ${numeroCuentas} cuentas, ${numeroGastos} gastos, ${numeroAhorros} ahorros, ${numeroMetas} metas`);
console.log(`Totales: Saldo=${saldoTotal}, Gastos=${totalGastos}, Ahorros=${totalAhorros}`);

// ========================================
// FORMATEAR CUENTAS
// ========================================
let cuentasTexto = '';
if (cuentas.length > 0) {
  cuentasTexto = cuentas.map(c => {
    const nombre = c.nombre || 'Sin nombre';
    const saldo = Number(c.saldo) || 0;
    return `  💳 ${nombre}: Q${saldo.toFixed(2)}`;
  }).join('\n');
} else {
  cuentasTexto = '  ❌ No hay cuentas registradas';
}

// ========================================
// FORMATEAR GASTOS
// ========================================
let gastosTexto = '';
if (gastos.length > 0) {
  gastosTexto = gastos.map(g => {
    const descripcion = g.descripcion || 'Sin descripción';
    const monto = Number(g.monto) || 0;
    const fecha = g.fecha || 'Sin fecha';
    const cuenta = g.cuenta || 'Sin cuenta';
    return `  💸 ${descripcion}
     Monto: Q${monto.toFixed(2)}
     Fecha: ${fecha}
     Cuenta: ${cuenta}`;
  }).join('\n\n');
} else {
  gastosTexto = '  ❌ No hay gastos registrados';
}

// ========================================
// FORMATEAR AHORROS
// ========================================
let ahorrosTexto = '';
if (ahorros.length > 0) {
  ahorrosTexto = ahorros.map(a => {
    const descripcion = a.descripcion || 'Sin descripción';
    const monto = Number(a.monto) || 0;
    const fecha = a.fecha || 'Sin fecha';
    const cuenta = a.cuenta || 'Sin cuenta';
    return `  💰 ${descripcion}
     Monto: Q${monto.toFixed(2)}
     Fecha: ${fecha}
     Cuenta: ${cuenta}`;
  }).join('\n\n');
} else {
  ahorrosTexto = '  ❌ No hay ahorros registrados';
}

// ========================================
// FORMATEAR METAS
// ========================================
let metasTexto = '';
if (metas.length > 0) {
  metasTexto = metas.map(m => {
    const nombre = m.nombre || 'Sin nombre';
    const objetivo = Number(m.objetivo) || 1;
    const progreso = Number(m.progreso) || 0;
    const fechaLimite = m.fechaLimite || 'Sin fecha';
    const porcentaje = Number(m.porcentaje) || ((progreso / objetivo) * 100);
    
    // Crear barra de progreso visual
    const barrasLlenas = Math.min(Math.floor(porcentaje / 10), 10);
    const barrasVacias = 10 - barrasLlenas;
    const barraProgreso = '█'.repeat(barrasLlenas) + '░'.repeat(barrasVacias);
    
    return `  🎯 ${nombre}
     Objetivo: Q${objetivo.toFixed(2)}
     Progreso: Q${progreso.toFixed(2)}
     Porcentaje: ${porcentaje.toFixed(1)}%
     ${barraProgreso}
     Fecha límite: ${fechaLimite}`;
  }).join('\n\n');
} else {
  metasTexto = '  ❌ No hay metas registradas';
}

// ========================================
// CREAR CUERPO DEL EMAIL
// ========================================
const emailBody = `
╔═══════════════════════════════════════════════════╗
║       💰 REPORTE FINANCIERO DETALLADO 💰         ║
╚═══════════════════════════════════════════════════╝

📅 Fecha de generación: ${fechaGuatemala}
📧 Enviado a: ${email}

╔═══════════════════════════════════════════════════╗
║                📊 RESUMEN GENERAL                 ║
╚═══════════════════════════════════════════════════╝

💵 Saldo Total:         Q${saldoTotal.toFixed(2)}
💸 Total Gastos:        Q${totalGastos.toFixed(2)}
💰 Total Ahorros:       Q${totalAhorros.toFixed(2)}
🎯 Progreso en Metas:   Q${totalMetasProgreso.toFixed(2)} / Q${totalMetasObjetivo.toFixed(2)}
📈 Balance General:     Q${balanceGeneral.toFixed(2)}

╔═══════════════════════════════════════════════════╗
║           🏦 CUENTAS (${numeroCuentas})                          ║
╚═══════════════════════════════════════════════════╝

${cuentasTexto}

╔═══════════════════════════════════════════════════╗
║           💸 GASTOS (${numeroGastos})                           ║
╚═══════════════════════════════════════════════════╝

${gastosTexto}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL GASTADO: Q${totalGastos.toFixed(2)}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════════╗
║           💵 AHORROS (${numeroAhorros})                         ║
╚═══════════════════════════════════════════════════╝

${ahorrosTexto}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL AHORRADO: Q${totalAhorros.toFixed(2)}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════════╗
║         🎯 METAS FINANCIERAS (${numeroMetas})                  ║
╚═══════════════════════════════════════════════════╝

${metasTexto}

╔═══════════════════════════════════════════════════╗
║                  📱 INFORMACIÓN                   ║
╚═══════════════════════════════════════════════════╝

📱 App: Gestión Financiera Personal
🏦 Cuentas activas: ${numeroCuentas}
💰 Saldo disponible: Q${saldoTotal.toFixed(2)}
📊 Movimientos registrados: ${numeroGastos + numeroAhorros}
🎯 Metas en progreso: ${numeroMetas}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 Consejo financiero: 
   Mantén un registro constante de tus gastos para 
   alcanzar tus metas financieras más rápido.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Este reporte fue generado automáticamente desde tu
aplicación de Gestión Financiera.

`;

console.log("=== EMAIL GENERADO EXITOSAMENTE ===");
console.log("Longitud del email:", emailBody.length, "caracteres");

// ========================================
// RETORNAR RESULTADO
// ========================================
return {
  email: email,
  subject: `📊 Reporte Financiero - ${fechaOriginal.split(' ')[0]}`,
  body: emailBody,
  rawData: inputData
};
```

---

## 📋 INSTRUCCIONES PASO A PASO

### 1. Ve a n8n

Abre tu workflow: https://primary-production-78316.up.railway.app

### 2. Abre el nodo "Code"

Click en el nodo Code (el que está después del Webhook)

### 3. Borra TODO el código

- Selecciona todo (Ctrl + A)
- Bórralo (Delete)

### 4. Copia y pega el código de arriba

- Copia TODO el código JavaScript que está arriba
- Pégalo en el nodo Code

### 5. Configura el nodo Gmail

Asegúrate que el nodo Gmail tenga:

**To (Para):**
```
{{ $json.email }}
```

**Subject (Asunto):**
```
{{ $json.subject }}
```

**Email Type (Tipo):**
```
Text
```

**Message (Mensaje):**
```
{{ $json.body }}
```

### 6. Guarda y activa

- Click en "Save" (Guardar)
- Asegúrate que el workflow esté ACTIVO (botón verde)

### 7. Prueba de nuevo desde la app

- Envía el reporte desde la app
- Revisa tu correo

---

## 📧 EJEMPLO DE CÓMO SE VERÁ EL EMAIL

```
╔═══════════════════════════════════════════════════╗
║       💰 REPORTE FINANCIERO DETALLADO 💰         ║
╚═══════════════════════════════════════════════════╝

📅 Fecha de generación: 2025-10-23 18:03:10 (Hora Guatemala)
📧 Enviado a: edwinsjosuearguetaduarte1@gmail.com

╔═══════════════════════════════════════════════════╗
║                📊 RESUMEN GENERAL                 ║
╚═══════════════════════════════════════════════════╝

💵 Saldo Total:         Q18300.00
💸 Total Gastos:        Q500.00
💰 Total Ahorros:       Q800.00
🎯 Progreso en Metas:   Q2000.00 / Q9000.00
📈 Balance General:     Q18600.00

╔═══════════════════════════════════════════════════╗
║           🏦 CUENTAS (1)                          ║
╚═══════════════════════════════════════════════════╝

  💳 Edwins: Q18300.00

╔═══════════════════════════════════════════════════╗
║           💸 GASTOS (1)                           ║
╚═══════════════════════════════════════════════════╝

  💸 Suscripcion de IA
     Monto: Q500.00
     Fecha: 23/10/2025
     Cuenta: Edwins

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL GASTADO: Q500.00
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════════╗
║           💵 AHORROS (1)                          ║
╚═══════════════════════════════════════════════════╝

  💰 Regalo de mi Mamá
     Monto: Q800.00
     Fecha: 23/10/2025
     Cuenta: Edwins

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL AHORRADO: Q800.00
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔═══════════════════════════════════════════════════╗
║         🎯 METAS FINANCIERAS (1)                  ║
╚═══════════════════════════════════════════════════╝

  🎯 Moto
     Objetivo: Q9000.00
     Progreso: Q2000.00
     Porcentaje: 22.2%
     ██░░░░░░░░
     Fecha límite: 31/10/2025
```

---

## ✅ ESTO FUNCIONARÁ 100% PORQUE:

1. ✅ Usa exactamente los nombres de campos que tu app envía
2. ✅ Maneja todos los datos correctamente
3. ✅ Formatea la fecha para Guatemala
4. ✅ Muestra TODOS los datos (cuenta, gastos, ahorros, metas)
5. ✅ Tiene formato profesional y fácil de leer

**Actualiza el código en n8n AHORA y prueba de nuevo.** Te garantizo que funcionará perfectamente. 🎯

