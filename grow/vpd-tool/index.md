---
layout: default
title: VPD-Rechner – Vapour Pressure Deficit
sidebar: grow
---

# VPD-Rechner

Dieser Rechner bestimmt den Vapour Pressure Deficit (VPD) anhand von:

- Temperatur (°C)
- relativer Luftfeuchtigkeit (%)
- und optional der **Blatt-Temperatur** (korrigiert VPD deutlich besser)

Für DWC-, Coco- und Hydro-Grows extrem hilfreich, um Transpiration,
Nährstofffluss und Klima optimal einzustellen.

<div class="tool-card">
  <h2>Eingaben</h2>

  <div class="tool-grid">

    <div class="tool-field">
      <label for="vpdTemp">Lufttemperatur (°C)</label>
      <input id="vpdTemp" type="number" step="0.1" value="25">
    </div>

    <div class="tool-field">
      <label for="vpdRH">Luftfeuchtigkeit (%)</label>
      <input id="vpdRH" type="number" step="1" value="60">
    </div>

    <div class="tool-field">
      <label for="vpdLeaf">Blatt-Temperatur (optional, °C)</label>
      <input id="vpdLeaf" type="number" step="0.1" placeholder="optional">
    </div>

  </div>

  <button id="vpdCalcBtn" class="hero-button">Berechnen</button>

  <div id="vpdResult" class="tool-result" style="margin-top:1rem;"></div>
</div>

<script>
(function(){

  function saturationVP(temp) {
    return 0.6108 * Math.exp((17.27 * temp) / (temp + 237.3)); // kPa
  }

  function calcVPD() {
    var T = parseFloat(document.getElementById('vpdTemp').value || "0");
    var RH = parseFloat(document.getElementById('vpdRH').value || "0");
    var leafField = document.getElementById('vpdLeaf');
    var leafTemp = parseFloat(leafField.value);

    var useLeaf = !isNaN(leafTemp);

    var tempForVPD = useLeaf ? leafTemp : T;

    var svp = saturationVP(tempForVPD);      // kPa
    var avp = svp * (RH / 100);              // tatsächl. Dampfdruck
    var vpd = svp - avp;

    var box = document.getElementById('vpdResult');

    var lvl = "";
    if (vpd < 0.6) lvl = "zu niedrig (Pflanze transpir. kaum)";
    else if (vpd < 1.2) lvl = "optimal für Vegi";
    else if (vpd < 1.5) lvl = "optimal für frühe Blüte";
    else if (vpd < 1.8) lvl = "etwas hoch, aufpassen";
    else lvl = "hoch → Risiko für Stress";

    box.innerHTML =
      "<h3>Ergebnis</h3>" +
      "<p><strong>VPD:</strong> " + vpd.toFixed(2) + " kPa</p>" +
      "<p><strong>Einschätzung:</strong> " + lvl + "</p>" +
      (useLeaf
        ? "<p>Berechnung mit Blatt-Temperatur ("+leafTemp+" °C)</p>"
        : "<p>Berechnung ohne Blatt-Temperatur (Lufttemp verwendet)</p>"
      );
  }

  var btn = document.getElementById('vpdCalcBtn');
  if(btn) btn.addEventListener("click", calcVPD);

})();
</script>
