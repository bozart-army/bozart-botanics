---
layout: default
title: EC-Rechner für DWC & Hydro
sidebar: dwc
---

# EC-Rechner für DWC & Hydro

Dieser Rechner hilft dir grob abzuschätzen, wie viel Dünger du pro Liter
zugeben musst, um von deinem Ausgangs-EC auf einen gewünschten Ziel-EC
zu kommen.

**Wichtig:**  
Das ist nur eine Orientierungshilfe.  
Verlass dich immer auf:

- die Herstellerangaben deines Düngers und
- ein echtes EC-Messgerät im Grow.

<div id="ec-tool" class="tool-card">
  <h2>Berechnung</h2>
  <div class="tool-grid">
    <div class="tool-field">
      <label for="waterEc">Ausgangs-EC des Wassers (mS/cm)</label>
      <input id="waterEc" type="number" step="0.01" value="0.40">
    </div>
    <div class="tool-field">
      <label for="targetEc">Ziel-EC (mS/cm)</label>
      <input id="targetEc" type="number" step="0.01" value="1.80">
    </div>
    <div class="tool-field">
      <label for="stockStrength">
        Herstellerangabe: EC bei 1&nbsp;ml/L (mS/cm)
      </label>
      <input id="stockStrength" type="number" step="0.01" value="1.20">
    </div>
  </div>

  <button id="ecCalcBtn" class="hero-button">Berechnen</button>

  <div id="ecResult" class="tool-result" aria-live="polite"></div>

  <p class="tool-note">
    Beispiel: Hat dein Wasser 0,4&nbsp;mS/cm und du willst auf 1,8&nbsp;mS/cm,
    berechnet das Tool eine grobe ml/L-Dosierung basierend auf der
    Herstellerangabe deines Düngers.
  </p>
</div>

<script>
  (function() {
    function calcEc() {
      var waterEc = parseFloat(document.getElementById('waterEc').value || '0');
      var targetEc = parseFloat(document.getElementById('targetEc').value || '0');
      var stockStrength = parseFloat(document.getElementById('stockStrength').value || '0');

      var resultEl = document.getElementById('ecResult');

      if (isNaN(waterEc) || isNaN(targetEc) || isNaN(stockStrength) || stockStrength <= 0) {
        resultEl.textContent = "Bitte alle Felder korrekt ausfüllen.";
        return;
      }

      if (targetEc <= waterEc) {
        resultEl.textContent = "Ziel-EC muss höher als der Ausgangs-EC sein.";
        return;
      }

      var ecDelta = targetEc - waterEc;
      // grobe lineare Annahme: 1 ml/L erhöht EC um 'stockStrength'
      var mlPerL = ecDelta / stockStrength;

      if (mlPerL < 0) mlPerL = 0;

      resultEl.innerHTML =
        "<strong>Ergebnis:</strong> ca. <strong>" +
        mlPerL.toFixed(2) +
        " ml Dünger pro Liter</strong>." +
        "<br><small>In der Praxis auf einen einfachen Wert runden (z. B. " +
        Math.max(0, mlPerL.toFixed(1)) +
        " ml/L) und immer mit EC-Meter gegenprüfen.</small>";
    }

    var btn = document.getElementById('ecCalcBtn');
    if (btn) {
      btn.addEventListener('click', calcEc);
    }
  })();
</script>
