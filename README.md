<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Calculadora de Costes de Espejo</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50 text-gray-900">
  <main class="max-w-3xl mx-auto p-6 space-y-6">
    <h1 class="text-2xl font-semibold">Calculadora de Costes de Espejo</h1>

    <!-- Panel de entradas (SIEMPRE tamaño en centímetros) -->
    <section class="grid grid-cols-1 gap-4 rounded-2xl border bg-white p-5 shadow-sm">
      <h2 class="text-base font-medium">Entradas (obligatorias)</h2>
      <div class="grid grid-cols-2 md:grid-cols-5 gap-4 items-end">
        <label class="block text-sm md:col-span-2">
          <span class="text-gray-600">Precio del espejo (€)</span>
          <input id="precioEspejo" type="number" step="0.01" class="mt-1 w-full rounded-lg border px-3 py-2" value="9.9" required />
        </label>
        <label class="block text-sm">
          <span class="text-gray-600">Minutos</span>
          <input id="minutos" type="number" step="1" class="mt-1 w-full rounded-lg border px-3 py-2" value="60" required />
        </label>
        <label class="block text-sm">
          <span class="text-gray-600">Transporte (€)</span>
          <input id="transporte" type="number" step="0.01" class="mt-1 w-full rounded-lg border px-3 py-2" value="11" required />
        </label>
        <div class="md:col-span-5 h-px bg-gray-200"></div>
        <label class="block text-sm">
          <span class="text-gray-600">Ancho (cm)</span>
          <input id="anchoCm" type="number" step="1" class="mt-1 w-full rounded-lg border px-3 py-2" value="80" required />
        </label>
        <label class="block text-sm">
          <span class="text-gray-600">Alto (cm)</span>
          <input id="altoCm" type="number" step="1" class="mt-1 w-full rounded-lg border px-3 py-2" value="80" required />
        </label>
      </div>
      <p class="text-xs text-gray-500">Introduce SIEMPRE el tamaño del espejo en centímetros. Las longitudes calculadas se muestran también en centímetros. Los costes usan tarifas en €/m (el sistema convierte cm → m internamente).</p>
    </section>

    <!-- Resultados -->
    <section class="grid grid-cols-1 gap-4 rounded-2xl border bg-white p-5 shadow-sm">
      <h2 class="text-base font-medium">Resultado</h2>
      <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-sm">
        <div class="rounded-xl border p-3">
          <div class="text-gray-500">Perímetro</div>
          <div id="kpiPerimetro" class="text-lg font-semibold">—</div>
        </div>
        <div class="rounded-xl border p-3">
          <div class="text-gray-500">LED (cm)</div>
          <div id="kpiLed" class="text-lg font-semibold">—</div>
        </div>
        <div class="rounded-xl border p-3">
          <div class="text-gray-500">Perfil (cm)</div>
          <div id="kpiPerfil" class="text-lg font-semibold">—</div>
        </div>
        <div class="rounded-xl border p-3">
          <div class="text-gray-500">Cartón (cm)</div>
          <div id="kpiCarton" class="text-lg font-semibold">—</div>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="rounded-xl border p-4 space-y-2">
          <div class="flex justify-between text-sm"><span class="text-gray-600">Perfil REF-655</span><span id="cPerfil" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Tira LED 4000K</span><span id="cLed" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Cartón fan-fold</span><span id="cCarton" class="font-medium">—</span></div>
        </div>
        <div class="rounded-xl border p-4 space-y-2">
          <div class="flex justify-between text-sm"><span class="text-gray-600">Vidrio (entrada)</span><span id="cVidrio" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Mano de obra</span><span id="cMano" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Transporte</span><span id="cTransporte" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Fijos</span><span id="cFijos" class="font-medium">—</span></div>
        </div>
      </div>

      <div class="flex items-center justify-between rounded-2xl border p-4">
        <span class="text-base font-medium">Total</span>
        <span id="cTotal" class="text-2xl font-semibold">—</span>
      </div>

      <p class="text-xs text-gray-500">Parámetros: perfil 0,65625 €/m, LED 2,00 €/m (factor 0,953125), cartón 1,76 €/m (factor 1,125), fijos 11,47 €, mano de obra 0,25 €/min. Perímetro, LED, perfil y cartón se muestran en centímetros.</p>
    </section>
  </main>

  <script>
    // Parámetros (tarifas en €/m; conversiones cm→m internas)
    const PROFILE_RATE_PER_M = 0.65625;  // €/m
    const LED_RATE_PER_M = 2.00;         // €/m
    const LED_FACTOR = 0.953125;         // proporción de perímetro
    const CARTON_RATE_PER_M = 1.76;      // €/m (precio original 0.88 × 2)
    const CARTON_FACTOR = 1.125;         // proporción de perímetro
    const FIXED_COST = 11.47;            // €
    const LABOR_RATE_PER_MIN = 0.25;     // €/min

    const $ = (id) => document.getElementById(id);
    const euros = (n) => (new Intl.NumberFormat('es-ES', { style: 'currency', currency: 'EUR' })).format(n || 0);

    const inputs = ["precioEspejo", "minutos", "transporte", "anchoCm", "altoCm"];  
    inputs.forEach(id => $(id).addEventListener('input', calc));

    function valNum(id) {
      const v = parseFloat($(id).value);
      return Number.isFinite(v) ? v : 0;
    }

    function calc() {
      const E = valNum("precioEspejo");
      const Min = valNum("minutos");
      const T = valNum("transporte");
      const Wcm = valNum("anchoCm");
      const Hcm = valNum("altoCm");

      const invalido = (Wcm <= 0 || Hcm <= 0);

      const Pcm = 2 * (Wcm + Hcm);        // cm
      const Pm  = Pcm / 100;              // m

      const led_m = LED_FACTOR * Pm;
      const perfil_m = Pm;
      const carton_m = CARTON_FACTOR * Pm;

      const costePerfil = PROFILE_RATE_PER_M * perfil_m;
      const costeLed = LED_RATE_PER_M * led_m;
      const costeCarton = CARTON_RATE_PER_M * carton_m;
      const manoObra = LABOR_RATE_PER_MIN * Min;

      const total = (invalido ? 0 : (E + costePerfil + costeLed + costeCarton + manoObra + T + FIXED_COST));

      $("kpiPerimetro").textContent = invalido ? '—' : (Pcm.toFixed(0) + ' cm');
      $("kpiLed").textContent = invalido ? '—' : ((led_m * 100).toFixed(0) + ' cm');
      $("kpiPerfil").textContent = invalido ? '—' : ((perfil_m * 100).toFixed(0) + ' cm');
      $("kpiCarton").textContent = invalido ? '—' : ((carton_m * 100).toFixed(0) + ' cm');

      $("cPerfil").textContent = invalido ? '—' : euros(costePerfil);
      $("cLed").textContent = invalido ? '—' : euros(costeLed);
      $("cCarton").textContent = invalido ? '—' : euros(costeCarton);
      $("cVidrio").textContent = invalido ? '—' : euros(E);
      $("cMano").textContent = euros(manoObra);
      $("cTransporte").textContent = euros(T);
      $("cFijos").textContent = euros(FIXED_COST);
      $("cTotal").textContent = euros(total);
    }

    calc();
  </script>
</body>
</html>
