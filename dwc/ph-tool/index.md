---
layout: default
title: pH-Grundlagen & Rechner
sidebar: dwc
---

# pH-Grundlagen & Rechner

Der pH-Wert beschreibt, wie sauer oder basisch eine Lösung ist. In der
Hydroponik ist er entscheidend dafür, welche Nährstoffe von der Pflanze
aufgenommen werden können.

- pH < 7: sauer  
- pH = 7: neutral  
- pH > 7: basisch  

Im DWC-Bereich bewegen sich viele Gärtner grob zwischen pH 5,5 und 6,2
(je nach Dünger, Strain und persönlicher Strategie).

<div class="tool-card">
  <h2>pH → H⁺-Konzentration</h2>
  <div class="tool-grid">
    <div class="tool-field">
      <label for="phInput">pH-Wert</label>
      <input id="phInput" type="number" step="0.01" value="5.80">
    </div>
  </div>

  <button id="phCalcBtn" class="hero-button">Berechnen</button>

  <div id="phResult" class="tool-result" aria-live="polite"></div>

  <p class="tool-note">
    Hinweis: Der Rechner zeigt dir die ungefähre Protonenkonzentration in
    mol/L an und ordnet den Bereich (sauer / neutral / basisch) ein.
  </p>
</div>

<script>
  (function() {
    function calcPh() {
      var phField = document.getElementById('phInput');
      var ph = parseFloat(phField.value || '0');
      var resultEl = document.getElementById('phResult');

      if (isNaN(ph) || ph < 0 || ph > 14) {
        resultEl.textContent = "Bitte einen pH-Wert zwischen 0 und 14 eingeben.";
        return;
      }

      var h = Math.pow(10, -ph); // mol/L
      // wissenschaftliche Schreibweise
      var exponent = Math.floor(Math.log10(h));
      var mantissa = h / Math.pow(10, exponent);

      var zone = "neutral";
      if (ph < 7) zone = "sauer";
      if (ph > 7) zone = "basisch";

      resultEl.innerHTML =
        "<strong>H⁺-Konzentration:</strong> ca. " +
        mantissa.toFixed(2) + " × 10<sup>" + exponent + "</sup> mol/L" +
        "<br><strong>Bereich:</strong> " + zone + "." +
        "<br><small>Im DWC-Kontext ist ein leicht saurer Bereich (ca. pH 5,5–6,2) üblich.</small>";
    }

    var btn = document.getElementById('phCalcBtn');
    if (btn) {
      btn.addEventListener('click', calcPh);
    }
  })();
</script>
