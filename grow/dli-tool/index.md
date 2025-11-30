---
layout: default
title: DLI-Rechner – Daily Light Integral
sidebar: grow
---

# DLI-Rechner – Daily Light Integral

Dieser Rechner wandelt deine Beleuchtung in einen DLI-Wert um.  
Du gibst ein:

- PPFD (µmol/m²/s)
- Beleuchtungsdauer (Stunden pro Tag)

und erhältst:

- DLI (mol/m²/Tag)
- eine grobe Einschätzung für Vegi/Blüte

<div class="tool-card">
  <h2>Eingaben</h2>

  <div class="tool-grid">

    <div class="tool-field">
      <label for="dliPpfd">PPFD (µmol/m²/s)</label>
      <input id="dliPpfd" type="number" step="1" value="600">
    </div>

    <div class="tool-field">
      <label for="dliHours">Beleuchtungsdauer (Stunden/Tag)</label>
      <input id="dliHours" type="number" step="0.1" value="18">
    </div>

  </div>

  <button id="dliCalcBtn" class="hero-button">Berechnen</button>

  <div id="dliResult" class="tool-result" style="margin-top:1rem;"></div>
</div>

<script>
(function(){

  function calcDLI() {
    var ppfd  = parseFloat(document.getElementById('dliPpfd').value || "0");
    var hours = parseFloat(document.getElementById('dliHours').value || "0");

    var out   = document.getElementById('dliResult');

    if(isNaN(ppfd) || isNaN(hours) || ppfd <= 0 || hours <= 0) {
      out.innerHTML = "<p>Bitte sinnvolle PPFD- und Stunden-Werte eingeben.</p>";
      return;
    }

    // Formel: DLI = PPFD * 3600 * Stunden / 1.000.000
    var dli = ppfd * 3600 * hours / 1000000;

    var lvl = "";
    if (dli < 15) lvl = "niedrig – eher Seedlings/Schwache Beleuchtung";
    else if (dli < 25) lvl = "ok für Vegi oder sehr sanfte Blüte";
    else if (dli < 40) lvl = "typischer Bereich für starke Vegi / Blüte indoor";
    else lvl = "hoch – nur mit perfektem Klima & CO₂ sinnvoll";

    out.innerHTML =
      "<h3>Ergebnis</h3>" +
      "<p><strong>DLI:</strong> " + dli.toFixed(1) + " mol/m²/Tag</p>" +
      "<p><strong>Einschätzung:</strong> " + lvl + "</p>" +
      "<p class=\"tool-note\">" +
        "Hinweis: DLI ist nur ein Richtwert. Entscheidend ist, wie deine Pflanzen " +
        "darauf reagieren (Blattbild, Stress, Wachstum). " +
      "</p>";
  }

  var btn = document.getElementById('dliCalcBtn');
  if(btn) btn.addEventListener("click", calcDLI);

})();
</script>
