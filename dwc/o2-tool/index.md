---
layout: default
title: O₂ im Wasser – Temperaturrechner
sidebar: dwc
---

# O₂ im Wasser – Temperaturrechner

Je wärmer die Nährlösung, desto weniger Sauerstoff kann sie binden. Für
Hydro- und DWC-Systeme ist das kritisch: Zu wenig O₂ begünstigt Wurzelfäule
und Stress.

Dieses Tool schätzt die Sauerstoffsättigung (in mg/L) in Abhängigkeit von
der Wassertemperatur. Die Werte sind gerundet und dienen nur als grobe
Orientierung.

<div class="tool-card">
  <h2>Temperatur → gelöster Sauerstoff</h2>
  <div class="tool-grid">
    <div class="tool-field">
      <label for="tempInput">Wassertemperatur (°C)</label>
      <input id="tempInput" type="number" step="0.1" value="20">
    </div>
  </div>

  <button id="o2CalcBtn" class="hero-button">Berechnen</button>

  <div id="o2Result" class="tool-result" aria-live="polite"></div>

  <p class="tool-note">
    Faustregel: Für DWC sind Temperaturen um 18–20&nbsp;°C oft ein guter
    Kompromiss zwischen Wachstum und Sauerstoffgehalt.
  </p>
</div>

<script>
  (function() {
    // einfache Tabelle mit DO-Sättigung in mg/L (ungefähr, Süßwasser, Luftdruck ~1 bar)
    var points = [
      { t: 0,  do: 14.6 },
      { t: 5,  do: 12.8 },
      { t: 10, do: 11.3 },
      { t: 15, do: 10.1 },
      { t: 20, do: 9.1 },
      { t: 25, do: 8.3 },
      { t: 30, do: 7.6 }
    ];

    function interpDO(temp) {
      if (temp <= points[0].t) return points[0].do;
      if (temp >= points[points.length - 1].t) return points[points.length - 1].do;

      for (var i = 0; i < points.length - 1; i++) {
        var p1 = points[i];
        var p2 = points[i + 1];
        if (temp >= p1.t && temp <= p2.t) {
          var ratio = (temp - p1.t) / (p2.t - p1.t);
          return p1.do + (p2.do - p1.do) * ratio;
        }
      }
      return points[points.length - 1].do;
    }

    function calcO2() {
      var tempField = document.getElementById('tempInput');
      var temp = parseFloat(tempField.value || '0');
      var resultEl = document.getElementById('o2Result');

      if (isNaN(temp) || temp < 0 || temp > 35) {
        resultEl.textContent = "Bitte eine Temperatur zwischen 0 und 35 °C eingeben.";
        return;
      }

      var o2 = interpDO(temp);
      var info = "";

      if (temp < 16) {
        info = "kalt – viel Sauerstoff, aber Wachstum kann langsamer sein.";
      } else if (temp <= 22) {
        info = "guter Bereich für DWC: guter Kompromiss aus Wachstum und O₂.";
      } else if (temp <= 26) {
        info = "warm – Sauerstoff sinkt, Risiko für Wurzelstress steigt.";
      } else {
        info = "sehr warm – hohes Risiko für Wurzelfäule und O₂-Mangel.";
      }

      resultEl.innerHTML =
        "<strong>Sättigung:</strong> ca. " + o2.toFixed(2) + " mg/L O₂" +
        "<br><strong>Einschätzung:</strong> " + info;
    }

    var btn = document.getElementById('o2CalcBtn');
    if (btn) {
      btn.addEventListener('click', calcO2);
    }
  })();
</script>
