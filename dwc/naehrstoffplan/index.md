---
layout: default
title: Nährstoffplan-Template
sidebar: dwc
---

# Nährstoffplan-Template

Dieses Tool erzeugt einen einfachen Wochenplan für EC-Werte in Vegi und
Blüte. Es ersetzt keinen Herstellerplan, gibt dir aber eine saubere
Struktur, an der du dich orientieren kannst.

<div class="tool-card">
  <h2>Parameter</h2>
  <div class="tool-grid">
    <div class="tool-field">
      <label for="vegWeeks">Vegi-Wochen</label>
      <input id="vegWeeks" type="number" min="1" max="8" value="4">
    </div>
    <div class="tool-field">
      <label for="bloomWeeks">Blüte-Wochen</label>
      <input id="bloomWeeks" type="number" min="6" max="12" value="8">
    </div>
    <div class="tool-field">
      <label for="vegStartEc">Vegi EC-Start (mS/cm)</label>
      <input id="vegStartEc" type="number" step="0.1" value="0.8">
    </div>
    <div class="tool-field">
      <label for="vegEndEc">Vegi EC-Ende (mS/cm)</label>
      <input id="vegEndEc" type="number" step="0.1" value="1.6">
    </div>
    <div class="tool-field">
      <label for="bloomStartEc">Blüte EC-Start (mS/cm)</label>
      <input id="bloomStartEc" type="number" step="0.1" value="1.6">
    </div>
    <div class="tool-field">
      <label for="bloomEndEc">Blüte EC-Ende (mS/cm)</label>
      <input id="bloomEndEc" type="number" step="0.1" value="2.0">
    </div>
  </div>

  <button id="planCalcBtn" class="hero-button">Plan erstellen</button>

  <div id="planResult" class="tool-result" aria-live="polite"></div>

  <p class="tool-note">
    Du kannst die Werte später an deinen Dünger, Strain und deine Erfahrung
    anpassen. Der Plan ist als grobe Guideline gedacht.
  </p>
</div>

<script>
  (function() {
    function generatePlan() {
      var vegWeeks      = parseInt(document.getElementById('vegWeeks').value || '0', 10);
      var bloomWeeks    = parseInt(document.getElementById('bloomWeeks').value || '0', 10);
      var vegStartEc    = parseFloat(document.getElementById('vegStartEc').value || '0');
      var vegEndEc      = parseFloat(document.getElementById('vegEndEc').value || '0');
      var bloomStartEc  = parseFloat(document.getElementById('bloomStartEc').value || '0');
      var bloomEndEc    = parseFloat(document.getElementById('bloomEndEc').value || '0');

      var resultEl = document.getElementById('planResult');

      if (!vegWeeks || !bloomWeeks || vegWeeks < 1 || bloomWeeks < 1) {
        resultEl.textContent = "Bitte sinnvolle Wochenzahlen für Vegi und Blüte eingeben.";
        return;
      }

      var rows = [];

      function lerp(start, end, index, total) {
        if (total <= 1) return start;
        return start + (end - start) * (index / (total - 1));
      }

      // Vegi
      for (var i = 0; i < vegWeeks; i++) {
        var ecVeg = lerp(vegStartEc, vegEndEc, i, vegWeeks);
        rows.push({
          week: i + 1,
          phase: "Vegi",
          ec: ecVeg
        });
      }

      // Blüte
      for (var j = 0; j < bloomWeeks; j++) {
        var ecBloom = lerp(bloomStartEc, bloomEndEc, j, bloomWeeks);
        rows.push({
          week: vegWeeks + j + 1,
          phase: "Blüte",
          ec: ecBloom
        });
      }

      var html = "<table><thead><tr><th>Woche</th><th>Phase</th><th>Ziel-EC (mS/cm)</th></tr></thead><tbody>";

      rows.forEach(function(r) {
        html += "<tr><td>" + r.week + "</td><td>" + r.phase + "</td><td>" + r.ec.toFixed(2) + "</td></tr>";
      });

      html += "</tbody></table>";

      resultEl.innerHTML = html;
    }

    var btn = document.getElementById('planCalcBtn');
    if (btn) {
      btn.addEventListener('click', generatePlan);
    }
  })();
</script>
