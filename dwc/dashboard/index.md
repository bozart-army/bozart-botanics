---
layout: default
title: DWC Wasserparameter Dashboard
sidebar: dwc
---

# DWC Wasserparameter Dashboard

Dieses Dashboard fasst die wichtigsten Wasserparameter für DWC zusammen:

- EC (Nährstoffkonzentration)
- pH-Wert
- Wassertemperatur und daraus abgeleitet der O₂-Gehalt
- Gesamtbewertung mit Ampel-System

Die Bereiche sind bewusst als grobe Empfehlungen für viele Grows gewählt.
Du kannst sie später an deinen Stil, Dünger und Strain anpassen.

<div class="tool-card">
  <h2>Eingaben</h2>

  <div class="dashboard-grid">
    <div>
      <div class="tool-grid">
        <div class="tool-field">
          <label for="dashWaterEc">Ausgangs-EC des Wassers (mS/cm)</label>
          <input id="dashWaterEc" type="number" step="0.01" value="0.40">
        </div>
        <div class="tool-field">
          <label for="dashTargetEc">Ziel-EC der Nährlösung (mS/cm)</label>
          <input id="dashTargetEc" type="number" step="0.01" value="1.80">
        </div>
        <div class="tool-field">
          <label for="dashStockStrength">
            Herstellerangabe: EC bei 1&nbsp;ml/L (mS/cm)
          </label>
          <input id="dashStockStrength" type="number" step="0.01" value="1.20">
        </div>
        <div class="tool-field">
          <label for="dashPh">aktueller pH-Wert</label>
          <input id="dashPh" type="number" step="0.01" value="5.80">
        </div>
        <div class="tool-field">
          <label for="dashTemp">Wassertemperatur (°C)</label>
          <input id="dashTemp" type="number" step="0.1" value="20">
        </div>
      </div>

      <button id="dashCalcBtn" class="hero-button">Alles berechnen</button>

      <p class="tool-note">
        Tipp: Trage echte Messwerte aus deinem Setup ein (EC-Meter, pH-Meter,
        Thermometer). Das Dashboard zeigt dir dann grobe Einschätzungen und
        Hinweise.
      </p>
    </div>

    <div class="status-box">
      <h3>Aktueller Status</h3>
      <div id="dashStatusSummary" class="status-detail">
        Noch keine Berechnung durchgeführt.
      </div>

      <div class="status-grid" id="dashStatusGrid">
        <!-- wird per JavaScript gefüllt -->
      </div>
    </div>
  </div>

  <div id="dashDetails" class="tool-result"></div>
</div>

<script>
  (function() {
    // O2-Interpolation (wie im O2-Tool)
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

    function classifyEC(ec) {
      if (ec <= 0) return { level: "bad", label: "zu niedrig" };
      if (ec >= 0.8 && ec <= 2.2) {
        if (ec >= 1.2 && ec <= 2.0) {
          return { level: "ok", label: "im typischen Bereich" };
        }
        return { level: "warn", label: "noch ok, aber Randbereich" };
      }
      return { level: "bad", label: "außerhalb des üblichen Bereichs" };
    }

    function classifyPH(ph) {
      if (ph <= 0) return { level: "bad", label: "ungültig" };
      if (ph >= 5.3 && ph <= 6.5) {
        if (ph >= 5.5 && ph <= 6.2) {
          return { level: "ok", label: "hydro-typischer Bereich" };
        }
        return { level: "warn", label: "noch ok, aber Randbereich" };
      }
      return { level: "bad", label: "problematisch (Lockouts möglich)" };
    }

    function classifyTemp(temp) {
      if (temp < 0 || temp > 35) {
        return { level: "bad", label: "außerhalb Praxisbereich" };
      }
      if (temp >= 16 && temp <= 26) {
        if (temp >= 18 && temp <= 22) {
          return { level: "ok", label: "guter Kompromiss" };
        }
        return { level: "warn", label: "noch ok, aber erhöhtes Risiko" };
      }
      return { level: "bad", label: "hohes Risiko für Stress / Fäule" };
    }

    function classifyO2(o2) {
      if (o2 >= 8.0) return { level: "ok", label: "sehr gut gesättigt" };
      if (o2 >= 7.5) return { level: "warn", label: "noch akzeptabel" };
      return { level: "bad", label: "niedrig – O₂-Mangel möglich" };
    }

    function combineLevels(levels) {
      // bad > warn > ok
      if (levels.indexOf("bad") !== -1) return "bad";
      if (levels.indexOf("warn") !== -1) return "warn";
      return "ok";
    }

    function levelLabel(level) {
      if (level === "ok") return "optimal";
      if (level === "warn") return "riskant";
      return "kritisch";
    }

    function calcDashboard() {
      var waterEc       = parseFloat(document.getElementById('dashWaterEc').value || "0");
      var targetEc      = parseFloat(document.getElementById('dashTargetEc').value || "0");
      var stockStrength = parseFloat(document.getElementById('dashStockStrength').value || "0");
      var ph            = parseFloat(document.getElementById('dashPh').value || "0");
      var temp          = parseFloat(document.getElementById('dashTemp').value || "0");

      var statusSummaryEl = document.getElementById('dashStatusSummary');
      var statusGridEl    = document.getElementById('dashStatusGrid');
      var detailsEl       = document.getElementById('dashDetails');

      statusGridEl.innerHTML = "";
      detailsEl.innerHTML = "";

      if (isNaN(waterEc) || isNaN(targetEc) || isNaN(stockStrength) ||
          isNaN(ph) || isNaN(temp) || stockStrength <= 0) {
        statusSummaryEl.textContent = "Bitte alle Felder sinnvoll ausfüllen.";
        return;
      }

      if (targetEc <= waterEc) {
        statusSummaryEl.textContent = "Ziel-EC muss höher als der Ausgangs-EC sein.";
        return;
      }

      // EC-Dosierung
      var ecDelta = targetEc - waterEc;
      var mlPerL = ecDelta / stockStrength;
      if (mlPerL < 0) mlPerL = 0;

      // pH → H+ grob berechnen
      var hConc = Math.pow(10, -ph);
      var exponent = Math.floor(Math.log10(hConc));
      var mantissa = hConc / Math.pow(10, exponent);

      // O2
      var o2 = interpDO(temp);

      // Klassifizierungen
      var ecClass  = classifyEC(targetEc);
      var phClass  = classifyPH(ph);
      var tempClass= classifyTemp(temp);
      var o2Class  = classifyO2(o2);

      var allLevels = [ecClass.level, phClass.level, tempClass.level, o2Class.level];
      var overall   = combineLevels(allLevels);
      var overallText = levelLabel(overall);

      statusSummaryEl.innerHTML =
        "<strong>Gesamtbewertung:</strong> " +
        "<span class=\"status-pill status-" + overall + "\">" + overallText + "</span>";

      function renderRow(title, valueText, cls, desc) {
        var div = document.createElement('div');
        var pillClass = "status-pill status-" + cls;

        div.innerHTML =
          "<div class=\"status-item-title\"><strong>" + title + "</strong><br>" + valueText + "</div>" +
          "<div class=\"" + pillClass + "\">" + (cls === "ok" ? "ok" : (cls === "warn" ? "warn" : "kritisch")) + "</div>";
        statusGridEl.appendChild(div);

        var p = document.createElement('p');
        p.className = "status-detail";
        p.innerHTML = "<strong>" + title + ":</strong> " + desc;
        detailsEl.appendChild(p);
      }

      renderRow(
        "EC",
        targetEc.toFixed(2) + " mS/cm",
        ecClass.level,
        ecClass.label + " – passe ggf. an Strain, Phase und Düngerplan an."
      );

      renderRow(
        "pH",
        ph.toFixed(2),
        phClass.level,
        phClass.label + " – im DWC oft grob 5,5–6,2 als Ziel."
      );

      renderRow(
        "Temperatur",
        temp.toFixed(1) + " °C",
        tempClass.level,
        tempClass.label + " – kühleres Wasser hält mehr O₂, zu kalt bremst Wachstum."
      );

      renderRow(
        "O₂ im Wasser",
        o2.toFixed(2) + " mg/L (geschätzt)",
        o2Class.level,
        o2Class.label + " – Luftsteine, stärkerer Luftfluss oder Chiller können helfen."
      );

      var extra = document.createElement('p');
      extra.className = "tool-note";
      extra.innerHTML =
        "Hinweis: Dieses Dashboard ersetzt kein eigenes Monitoring. " +
        "Nutze es als Orientierung und dokumentiere deine Beobachtungen im Grow-Log.";
      detailsEl.appendChild(extra);

      // Kurzer Hinweis zur Dosierung
      var doseInfo = document.createElement('p');
      doseInfo.className = "tool-note";
      doseInfo.innerHTML =
        "<strong>Grobe Dosierung:</strong> ca. " + mlPerL.toFixed(2) +
        " ml Dünger pro Liter, um von " + waterEc.toFixed(2) + " auf " +
        targetEc.toFixed(2) + " mS/cm zu kommen (basierend auf der Herstellerangabe). " +
        "Immer mit EC-Meter gegenprüfen.";
      detailsEl.appendChild(doseInfo);
    }

    var btn = document.getElementById('dashCalcBtn');
    if (btn) {
      btn.addEventListener('click', calcDashboard);
    }
  })();
</script>
