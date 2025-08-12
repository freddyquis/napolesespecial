<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Calculadora Nápoles</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50 text-gray-900">
  <main class="max-w-3xl mx-auto p-6 space-y-6">
    <h1 class="text-2xl font-semibold">Calculadora Nápoles</h1>

    <!-- Entradas (obligatorias) -->
    <section class="grid grid-cols-1 gap-4 rounded-2xl border bg-white p-5 shadow-sm">
      <h2 class="text-base font-medium">Entradas</h2>
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
    </section>

    <!-- Resultado -->
    <section class="grid grid-cols-1 gap-4 rounded-2xl border bg-white p-5 shadow-sm">
      <h2 class="text-base font-medium">Resultado</h2>

      <div id="driverAlert" class="hidden rounded-xl border border-amber-300 bg-amber-50 p-3 text-amber-800">
        ⚠️ <span id="driverAlertText"></span>
      </div>

      <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-sm">
        <div class="rounded-xl border p-3"><div class="text-gray-500">Perímetro</div><div id="kpiPerimetro" class="text-lg font-semibold">—</div></div>
        <div class="rounded-xl border p-3"><div class="text-gray-500">LED (cm)</div><div id="kpiLed" class="text-lg font-semibold">—</div></div>
        <div class="rounded-xl border p-3"><div class="text-gray-500">Perfil (cm)</div><div id="kpiPerfil" class="text-lg font-semibold">—</div></div>
        <div class="rounded-xl border p-3"><div class="text-gray-500">Cartón (cm)</div><div id="kpiCarton" class="text-lg font-semibold">—</div></div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="rounded-xl border p-4 space-y-2">
          <div class="text-sm font-medium mb-1">Costes dimensionables</div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Perfil REF-655</span><span id="cPerfil" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Tira LED 4000K</span><span id="cLed" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Cartón fan-fold</span><span id="cCarton" class="font-medium">—</span></div>
        </div>
        <div class="rounded-xl border p-4 space-y-2">
          <div class="text-sm font-medium mb-1">Costes variables</div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Vidrio (entrada)</span><span id="cVidrio" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Mano de obra</span><span id="cMano" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Transporte</span><span id="cTransporte" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Plastificado (€/m²)</span><span id="cPlastificado" class="font-medium">—</span></div>
          <div class="flex justify-between text-sm"><span class="text-gray-600">Driver seleccionado</span><span id="cDriver" class="font-medium">—</span></div>
        </div>
      </div>

      <div class="rounded-xl border p-4 space-y-1 text-sm">
        <div class="font-medium">Desglose de precios fijos:</div>
        <ul id="fixedList" class="list-disc pl-6"></ul>
      </div>

      <div class="flex items-center justify-between rounded-2xl border p-4">
        <span class="text-base font-medium">Coste de fabricación</span>
        <span id="cTotal" class="text-2xl font-semibold">—</span>
      </div>
      <div class="flex items-center justify-between rounded-2xl border p-4 bg-green-50">
        <span class="text-base font-medium">PVP (x2,8)</span>
        <span id="cPVP" class="text-2xl font-semibold text-green-700">—</span>
      </div>
    </section>
  </main>

  <script>
    const PROFILE_RATE_PER_M = 0.65625;
    const LED_RATE_PER_M = 2.00;
    const LED_FACTOR = 0.953125;
    const CARTON_PRICE_PER_M = 1.76;
    const LABOR_RATE_PER_MIN = 0.25;

    // Precio por m² de plastificado (variable)
    const PLASTIFICADO_RATE_PER_M2 = 1.00;

    // Fijos (sin plastificado ni driver)
    const FIXED_ITEMS = [
      { label: "4 codos marco trasero", price: 1.40 },
      { label: "8 tornillos TORX", price: 0.40 },
      { label: "Silicona", price: 0.50 },
      { label: "Base adhesiva brida", price: 0.15 },
      { label: "Bridas", price: 0.06 },
      { label: "Caja conexiones", price: 0.50 },
      { label: "Cola caliente", price: 0.50 },
      { label: "Asa modelo 1000", price: 0.07 },
      { label: "Refuerzo modelo 200", price: 0.11 },
      { label: "Amortización máquina", price: 1.00 },
      { label: "Etiqueta técnica", price: 0.20 },
      { label: "Bolsa + manual", price: 0.50 },
      { label: "Pegatina espejo", price: 0.50 },
    ];
    const FIXED_COST = FIXED_ITEMS.reduce((acc, it) => acc + it.price, 0);

    // Selección dinámica de driver por perímetro (cm)
    function selectDriver(perimetroCm) {
      if (perimetroCm <= 300) return { label: "Driver 30W", price: 7.65, alert: null };
      if (perimetroCm <= 450) return { label: "Driver 45W", price: 9.90, alert: null };
      if (perimetroCm <= 600) return { label: "Driver 60W", price: 13.30, alert: null };
      if (perimetroCm <= 1000) return { label: "Driver 100W", price: 23.00, alert: null };
      return { label: "—", price: 0, alert: "Perímetro supera 1000 cm. Revisar driver." };
    }

    // Modelo fanfold para cartón (del código original)
    const FANFOLD_WIDTHS_MM = [990, 1500, 1980, 2400];
    const K_FACTORS = {
      "990x990": 2.37997 / 3.60,
      "1500x990": 3.63600 / 4.80,
      "990x1500": 3.63600 / 4.80,
      "1500x1500": 4.50600 / 5.31,
    };
    const EXP_FALLBACK = 0.6;
    const LEN_BASE_PER_H = 1215 / 800;
    const LEN_TAPA_PER_H = 1188 / 800;

    const $ = (id) => document.getElementById(id);
    const euros = (n) => (new Intl.NumberFormat('es-ES', { style: 'currency', currency: 'EUR' })).format(n || 0);

    function pickWidth(requiredMm) { for (const w of FANFOLD_WIDTHS_MM) if (w >= requiredMm) return w; return FANFOLD_WIDTHS_MM[FANFOLD_WIDTHS_MM.length - 1]; }
    function areaPartM2(fanfoldMm, lengthMm) { return (fanfoldMm / 1000) * (lengthMm / 1000); }
    function getK(wb, wt) {
      const key = `${wb}x${wt}`;
      let k = K_FACTORS[key] ?? K_FACTORS[`${wt}x${wb}`];
      if (k) return k;
      const kref = K_FACTORS["990x990"];
      const avg = (wb + wt) / 2;
      return kref * Math.pow(avg / 990, EXP_FALLBACK);
    }
    function estimateLengthsMm(Hmm) { return { base: Math.round(Hmm * LEN_BASE_PER_H), tapa: Math.round(Hmm * LEN_TAPA_PER_H) }; }
    function cartonUnitsMeters(Wcm, Hcm) {
      const Wmm = Wcm * 10, Hmm = Hcm * 10;
      const { base: Lb, tapa: Lt } = estimateLengthsMm(Hmm);
      const w_base = pickWidth(Wmm);
      const w_tapa = pickWidth(Wmm);
      const areaTotal = areaPartM2(w_base, Lb) + areaPartM2(w_tapa, Lt);
      const k = getK(w_base, w_tapa);
      return areaTotal / k;
    }

    ["precioEspejo", "minutos", "transporte", "anchoCm", "altoCm"].forEach(id => $(id).addEventListener('input', calc));

    function valNum(id) { const v = parseFloat($(id).value); return Number.isFinite(v) ? v : 0; }

    function calc() {
      const E = valNum("precioEspejo"), Min = valNum("minutos"), T = valNum("transporte"), Wcm = valNum("anchoCm"), Hcm = valNum("altoCm");
      const invalido = (Wcm <= 0 || Hcm <= 0);

      // Geometría y métricas
      const Pcm = 2 * (Wcm + Hcm);
      const Pm  = Pcm / 100;
      const led_m = LED_FACTOR * Pm;
      const perfil_m = Pm;
      const area_m2 = (Wcm * Hcm) / 10000;

      // Cartón por fanfold
      const carton_m = cartonUnitsMeters(Wcm, Hcm);
      const costeCarton = CARTON_PRICE_PER_M * carton_m;

      // Plastificado por m² (variable)
      const costePlastificado = PLASTIFICADO_RATE_PER_M2 * area_m2;

      // Driver dinámico
      const driver = selectDriver(Pcm);
      const costeDriver = driver.price;

      // Costes principales
      const costePerfil = PROFILE_RATE_PER_M * perfil_m;
      const costeLed = LED_RATE_PER_M * led_m;
      const manoObra = LABOR_RATE_PER_MIN * Min;

      const costeFabricacion = (invalido ? 0 :
        (E + costePerfil + costeLed + costeCarton + manoObra + T + FIXED_COST + costePlastificado + costeDriver)
      );

      // KPIs
      $("kpiPerimetro").textContent = invalido ? '—' : (Pcm.toFixed(0) + ' cm');
      $("kpiLed").textContent = invalido ? '—' : ((led_m * 100).toFixed(0) + ' cm');
      $("kpiPerfil").textContent = invalido ? '—' : ((perfil_m * 100).toFixed(0) + ' cm');
      $("kpiCarton").textContent = invalido ? '—' : ((carton_m * 100).toFixed(0) + ' cm');

      // Desglose
      $("cPerfil").textContent = invalido ? '—' : euros(costePerfil);
      $("cLed").textContent = invalido ? '—' : euros(costeLed);
      $("cCarton").textContent = invalido ? '—' : euros(costeCarton);
      $("cVidrio").textContent = invalido ? '—' : euros(E);
      $("cMano").textContent = euros(manoObra);
      $("cTransporte").textContent = euros(T);
      $("cPlastificado").textContent = invalido ? '—' : euros(costePlastificado);
      $("cDriver").textContent = invalido ? '—' : `${driver.label}: ${euros(costeDriver)}`;

      // Fijos
      $("fixedList").innerHTML = FIXED_ITEMS.map(it => `<li>${it.label}: ${euros(it.price)}</li>`).join('');

      // Alerta si perímetro > 1000 cm
      const alertBox = $("driverAlert");
      const alertText = $("driverAlertText");
      if (!invalido && driver.alert) {
        alertBox.classList.remove('hidden');
        alertText.textContent = driver.alert;
      } else {
        alertBox.classList.add('hidden');
        alertText.textContent = '';
      }

      // Totales
      $("cTotal").textContent = euros(costeFabricacion);
      const pvp = costeFabricacion * 2.8;
      $("cPVP").textContent = euros(pvp);
    }

    calc();
  </script>
</body>
</html>

    calc();
  </script>
</body>
</html>
