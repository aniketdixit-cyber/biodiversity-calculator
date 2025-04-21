<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Biodiversity Calculator</title>
  <style>
    body { font-family: 'Segoe UI', sans-serif; background: #f0f0f0; padding: 20px; }
    .container { max-width: 600px; margin: auto; background: white; padding: 30px; border-radius: 12px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    textarea { width: 100%; height: 120px; margin: 10px 0; padding: 10px; font-size: 1rem; }
    button { padding: 10px 15px; background: #006d77; color: white; border: none; border-radius: 6px; cursor: pointer; margin-top: 10px; }
    .result { margin-top: 20px; background: #e0f7fa; padding: 15px; border-radius: 10px; }
    #downloadBtn { background: #008080; }
  </style>
</head>
<body>
  <div class="container">
    <h1>Biodiversity Indices Calculator</h1>
    <p>Enter species and count (e.g., <code>Species_A: 30</code>):</p>
    <textarea id="inputData"></textarea>
    <button onclick="calculate()">Calculate & Submit</button>

    <div id="results" class="result" style="display: none;">
      <h2>Results:</h2>
      <p id="output"></p>
      <button id="downloadBtn" onclick="downloadPDF()">Download PDF</button>
    </div>
  </div>

  <!-- jsPDF library -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>

  <script>
    let latestResults = '';

    function parseInput(text) {
      const lines = text.trim().split("\n");
      const counts = {};
      lines.forEach(line => {
        const [species, count] = line.split(":").map(s => s.trim());
        if (species && count && !isNaN(count)) {
          counts[species] = parseInt(count);
        }
      });
      return counts;
    }

    function calculate() {
      const inputText = document.getElementById("inputData").value;
      const counts = parseInput(inputText);
      const N = Object.values(counts).reduce((a, b) => a + b, 0);
      const S = Object.keys(counts).length;
      const shannon = -Object.values(counts).reduce((sum, n) => sum + (n/N) * Math.log(n/N), 0);
      const simpson = 1 - Object.values(counts).reduce((sum, n) => sum + (n * (n - 1)) / (N * (N - 1)), 0);
      const evenness = S > 1 ? shannon / Math.log(S) : 0;

      latestResults = `
        Total Individuals (N): ${N}\n
        Species Richness (S): ${S}\n
        Shannon Index (H'): ${shannon.toFixed(4)}\n
        Simpson Index (D): ${simpson.toFixed(4)}\n
        Evenness (E): ${evenness.toFixed(4)}
      `;

      document.getElementById("output").innerHTML = latestResults.replace(/\n/g, "<br>");
      document.getElementById("results").style.display = "block";

      const data = {
        species_data: inputText,
        total_individuals: N,
        species_richness: S,
        shannon_index: shannon.toFixed(4),
        simpson_index: simpson.toFixed(4),
        evenness: evenness.toFixed(4)
      };

      fetch("https://script.google.com/macros/s/AKfycbzzP4CfG-Vy8QSDH_1o1mzPhuLerPU5oFrNZ9BfLyQGWRH5v-b8QlDQCvtrO9Mxz7yl/exec", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      })
      .then(response => response.text())
      .then(msg => console.log("Submitted to Google Sheets:", msg))
      .catch(err => console.error("Google Sheets Error:", err));
    }

    async function downloadPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      doc.setFontSize(14);
      doc.text("Biodiversity Calculation Results", 20, 20);
      doc.setFontSize(12);
      const lines = latestResults.trim().split('\n');
      lines.forEach((line, index) => {
        doc.text(line.trim(), 20, 40 + index * 10);
      });
      doc.save("biodiversity-results.pdf");
    }
  </script>
</body>
</html>
