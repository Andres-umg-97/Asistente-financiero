# 🔥 CÓDIGO N8N CORREGIDO - USA ESTE

## ⚠️ IMPORTANTE: Reemplaza TODO el código en tu nodo Code de n8n con este

```javascript
// Obtener datos del webhook
const data = $input.first().json;

// Log para debugging
console.log("=== DATOS RECIBIDOS ===");
console.log(JSON.stringify(data, null, 2));

// Extraer datos con valores seguros
const email = data.email || 'sin-email@ejemplo.com';
const fechaGeneracion = data.fechaGeneracion || new Date().toLocaleString('es-GT', { timeZone: 'America/Guatemala' });

// Convertir la fecha a hora de Guatemala
let fechaGuatemala = fechaGeneracion;
try {
  const fecha = new Date(fechaGeneracion);
  fechaGuatemala = fecha.toLocaleString('es-GT', { 
    timeZone: 'America/Guatemala',
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  });
} catch (e) {
  console.log("Error al convertir fecha:", e);
}

// Extraer totales
const saldoTotal = Number(data.saldoTotal) || 0;
const totalGastos = Number(data.totalGastos) || 0;
const totalAhorros = Number(data.totalAhorros) || 0;
const totalMetasProgreso = Number(data.totalMetasProgreso) || 0;
const totalMetasObjetivo = Number(data.totalMetasObjetivo) || 0;

// Extraer arrays
const cuentas = Array.isArray(data.cuentas) ? data.cuentas : [];
const gastos = Array.isArray(data.gastos) ? data.gastos : [];
const ahorros = Array.isArray(data.ahorros) ? data.ahorros : [];
const metas = Array.isArray(data.metas) ? data.metas : [];

// Extraer resumen
const resumen = data.resumen || {
  numeroCuentas: cuentas.length,
  numeroGastos: gastos.length,
  numeroAhorros: ahorros.length,
  numeroMetas: metas.length,
  balanceGeneral: saldoTotal - totalGastos + totalAhorros
};

console.log("=== DATOS PROCESADOS ===");
console.log("Cuentas:", cuentas.length);
console.log("Gastos:", gastos.length);
console.log("Ahorros:", ahorros.length);
console.log("Metas:", metas.length);

// FORMATEAR CUENTAS
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

// FORMATEAR GASTOS
let gastosTexto = '';
if (gastos.length > 0) {
  gastosTexto = gastos.map(g => {
    const desc = g.descripcion || 'Sin descripción';
    const monto = Number(g.monto) || 0;
    const fecha = g.fecha || 'Sin fecha';
    const cuenta = g.cuenta || 'Sin cuenta';
    return `  💸 ${desc}: Q${monto.toFixed(2)}
     📅 ${fecha} | 💳 ${cuenta}`;
  }).join('\n\n');
} else {
  gastosTexto = '  ❌ No hay gastos registrados';
}

// FORMATEAR AHORROS
let ahorrosTexto = '';
if (ahorros.length > 0) {
  ahorrosTexto = ahorros.map(a => {
    const desc = a.descripcion || 'Sin descripción';
    const monto = Number(a.monto) || 0;
    const fecha = a.fecha || 'Sin fecha';
    const cuenta = a.cuenta || 'Sin cuenta';
    return `  💰 ${desc}: Q${monto.toFixed(2)}
     📅 ${fecha} | 💳 ${cuenta}`;
  }).join('\n\n');
} else {
  ahorrosTexto = '  ❌ No hay ahorros registrados';
}

// FORMATEAR METAS
let metasTexto = '';
if (metas.length > 0) {
  metasTexto = metas.map(m => {
    const nombre = m.nombre || 'Sin nombre';
    const objetivo = Number(m.objetivo) || 1;
    const progreso = Number(m.progreso) || 0;
    const fechaLimite = m.fechaLimite || 'Sin fecha';
    const porcentaje = ((progreso / objetivo) * 100).toFixed(1);
    
    // Barra de progreso visual
    const barrasLlenas = Math.min(Math.floor(parseFloat(porcentaje) / 10), 10);
    const barraProgreso = '█'.repeat(barrasLlenas) + '░'.repeat(10 - barrasLlenas);
    
    return `  🎯 ${nombre}
     Objetivo: Q${objetivo.toFixed(2)}
     Progreso: Q${progreso.toFixed(2)} (${porcentaje}%)
     [${barraProgreso}]
     📅 Fecha límite: ${fechaLimite}`;
  }).join('\n\n');
} else {
  metasTexto = '  ❌ No hay metas registradas';
}

// CREAR EL EMAIL
const emailBody = `
╔═══════════════════════════════════════════════════╗
║       💰 REPORTE FINANCIERO DETALLADO 💰         ║
╚═══════════════════════════════════════════════════╝

📅 Fecha: ${fechaGuatemala}

╔═══════════════════════════════════════════════════╗
║                📊 RESUMEN GENERAL                 ║
╚═══════════════════════════════════════════════════╝

💵 Saldo Total:         Q${saldoTotal.toFixed(2)}
💸 Total Gastos:        Q${totalGastos.toFixed(2)}
💰 Total Ahorros:       Q${totalAhorros.toFixed(2)}
🎯 Progreso en Metas:   Q${totalMetasProgreso.toFixed(2)} / Q${totalMetasObjetivo.toFixed(2)}
📈 Balance General:     Q${Number(resumen.balanceGeneral || 0).toFixed(2)}

╔═══════════════════════════════════════════════════╗
║           🏦 CUENTAS (${resumen.numeroCuentas || 0})                      ║
╚═══════════════════════════════════════════════════╝

${cuentasTexto}

╔═══════════════════════════════════════════════════╗
║           💸 GASTOS (${resumen.numeroGastos || 0})                       ║
╚═══════════════════════════════════════════════════╝

${gastosTexto}

TOTAL GASTADO: Q${totalGastos.toFixed(2)}

╔═══════════════════════════════════════════════════╗
║           💵 AHORROS (${resumen.numeroAhorros || 0})                     ║
╚═══════════════════════════════════════════════════╝

${ahorrosTexto}

TOTAL AHORRADO: Q${totalAhorros.toFixed(2)}

╔═══════════════════════════════════════════════════╗
║         🎯 METAS FINANCIERAS (${resumen.numeroMetas || 0})              ║
╚═══════════════════════════════════════════════════╝

${metasTexto}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📱 Generado automáticamente desde tu App de Finanzas
🏦 Cuentas activas: ${resumen.numeroCuentas || 0}
💰 Saldo disponible: Q${saldoTotal.toFixed(2)}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 Consejo: Revisa tus finanzas regularmente para 
   alcanzar tus objetivos financieros.

`;

console.log("=== EMAIL GENERADO ===");
console.log("Destinatario:", email);
console.log("Longitud del email:", emailBody.length, "caracteres");

// Retornar el resultado
return [{
  json: {
    email: email,
    subject: `📊 Reporte Financiero - ${fechaGuatemala.split(' ')[0]}`,
    body: emailBody,
    rawData: data,
    debug: {
      cuentasCount: cuentas.length,
      gastosCount: gastos.length,
      ahorrosCount: ahorros.length,
      metasCount: metas.length
    }
  }
}];
```

---

## 🔧 INSTRUCCIONES PASO A PASO

### 1️⃣ Actualizar el código en n8n

1. Abre tu workflow en n8n
2. Click en el nodo **"Code"**
3. **SELECCIONA TODO** el código (Ctrl + A)
4. **BÓRRALO** (Delete)
5. **COPIA** todo el código JavaScript de arriba
6. **PÉGALO** en el nodo Code
7. Click en **"Save"** (Guardar)
8. Asegúrate que el workflow esté **ACTIVO** (botón verde)

### 2️⃣ Configurar el nodo Gmail

Asegúrate de que el nodo Gmail tenga:

- **To**: `{{ $json.email }}` (con doble llave)
- **Subject**: `{{ $json.subject }}` (con doble llave)
- **Email Type**: `Text`
- **Message**: `{{ $json.body }}` (con doble llave)

### 3️⃣ Probar

1. Compila y ejecuta la app
2. Agrega datos si no tienes:
   - Al menos 1 cuenta
   - Al menos 1 gasto
   - Al menos 1 ahorro
   - Al menos 1 meta
3. Ve a "Inicio"
4. Click en "Enviar Reporte por Gmail"
5. Ingresa tu email
6. Envía

### 4️⃣ Verificar

- En **Logcat** verás el JSON enviado
- En **n8n → Executions** verás la ejecución
- En **Gmail** recibirás el email con TODOS los datos

---

## 🎯 LO QUE CORREGÍ

✅ **Zona horaria de Guatemala**: Ahora usa `America/Guatemala`
✅ **Formato de hora**: Muestra fecha y hora completa
✅ **Formateo mejorado**: Cada sección está bien delimitada
✅ **Manejo de datos vacíos**: No rompe si falta algo
✅ **Logs de debugging**: Verás en n8n console qué está pasando
✅ **Emojis y formato visual**: Más fácil de leer

---

## 📧 EJEMPLO DE CÓMO SE VERÁ EL EMAIL

```
╔═══════════════════════════════════════════════════╗
║       💰 REPORTE FINANCIERO DETALLADO 💰         ║
╚═══════════════════════════════════════════════════╝

📅 Fecha: 23/10/2025, 15:30:45

╔═══════════════════════════════════════════════════╗
║                📊 RESUMEN GENERAL                 ║
╚═══════════════════════════════════════════════════╝

💵 Saldo Total:         Q5300.00
💸 Total Gastos:        Q500.00
💰 Total Ahorros:       Q800.00
🎯 Progreso en Metas:   Q3000.00 / Q10000.00
📈 Balance General:     Q5600.00

╔═══════════════════════════════════════════════════╗
║           🏦 CUENTAS (1)                          ║
╚═══════════════════════════════════════════════════╝

  💳 Banco Principal: Q5300.00

╔═══════════════════════════════════════════════════╗
║           💸 GASTOS (1)                           ║
╚═══════════════════════════════════════════════════╝

  💸 Supermercado: Q500.00
     📅 23/10/2025 | 💳 Banco Principal

...y así con todos tus datos
```

---

¡Copia el código JavaScript de arriba y pégalo en n8n AHORA! 🚀

