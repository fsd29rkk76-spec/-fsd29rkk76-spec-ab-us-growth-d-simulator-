
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>外貨積立シミュレーター</title>
  <!-- PWA設定 -->
  <meta name="theme-color" content="#2563eb">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    * { box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background-color: #f8fafc; color: #0f172a; margin: 0; padding: 16px; }
    .card { background: white; border-radius: 16px; padding: 20px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); max-width: 500px; margin: 0 auto 16px; }
    h2 { font-size: 1.25rem; margin-top: 0; color: #1e293b; }
    .form-group { margin-bottom: 16px; }
    label { display: block; font-size: 0.875rem; font-weight: 600; margin-bottom: 6px; color: #475569; }
    input[type="number"], input[type="range"] { width: 100%; padding: 10px; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 1rem; }
    .range-val { text-align: right; font-size: 0.875rem; color: #2563eb; font-weight: bold; }
    .result-box { background: #eff6ff; border-radius: 12px; padding: 16px; margin-top: 16px; }
    .result-row { display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 0.95rem; }
    .result-row.total { font-weight: bold; font-size: 1.1rem; color: #1d4ed8; border-top: 1px solid #bfdbfe; padding-top: 8px; }
    canvas { max-width: 100%; margin-top: 16px; }
  </style>
</head>
<body>

  <div class="card">
    <h2>外貨積立・利息再投資計算</h2>
    
    <div class="form-group">
      <label>初期投資額 (円)</label>
      <input type="number" id="initial" value="100000" step="10000" oninput="calculate()">
    </div>

    <div class="form-group">
      <label>毎月の積立額 (円)</label>
      <input type="number" id="monthly" value="30000" step="5000" oninput="calculate()">
    </div>

    <div class="form-group">
      <label>想定年利 (%)</label>
      <div class="range-val" id="rateVal">4.0 %</div>
      <input type="range" id="rate" min="0" max="15" value="4.0" step="0.5" oninput="calculate()">
    </div>

    <div class="form-group">
      <label>運用期間 (年)</label>
      <div class="range-val" id="yearsVal">10 年</div>
      <input type="range" id="years" min="1" max="30" value="10" step="1" oninput="calculate()">
    </div>

    <div class="result-box">
      <div class="result-row">
        <span>積立元本合計:</span>
        <span id="resPrincipal">0 円</span>
      </div>
      <div class="result-row">
        <span>運用益（利息合計）:</span>
        <span id="resInterest">0 円</span>
      </div>
      <div class="result-row total">
        <span>最終評価額:</span>
        <span id="resTotal">0 円</span>
      </div>
    </div>

    <canvas id="chart"></canvas>
  </div>

  <script>
    let myChart = null;

    function calculate() {
      const initial = parseFloat(document.getElementById('initial').value) || 0;
      const monthly = parseFloat(document.getElementById('monthly').value) || 0;
      const rate = parseFloat(document.getElementById('rate').value) || 0;
      const years = parseInt(document.getElementById('years').value) || 1;

      document.getElementById('rateVal').innerText = rate.toFixed(1) + ' %';
      document.getElementById('yearsVal').innerText = years + ' 年';

      const monthlyRate = rate / 100 / 12;
      const labels = [];
      const principalData = [];
      const totalData = [];

      let currentTotal = initial;
      let currentPrincipal = initial;

      labels.push('0年');
      principalData.push(initial);
      totalData.push(initial);

      for (let y = 1; y <= years; y++) {
        for (let m = 0; m < 12; m++) {
          currentTotal = (currentTotal + monthly) * (1 + monthlyRate);
          currentPrincipal += monthly;
        }
        labels.push(y + '年');
        principalData.push(Math.round(currentPrincipal));
        totalData.push(Math.round(currentTotal));
      }

      const finalTotal = totalData[totalData.length - 1];
      const finalPrincipal = principalData[principalData.length - 1];
      const finalInterest = finalTotal - finalPrincipal;

      document.getElementById('resPrincipal').innerText = finalPrincipal.toLocaleString() + ' 円';
      document.getElementById('resInterest').innerText = finalInterest.toLocaleString() + ' 円';
      document.getElementById('resTotal').innerText = finalTotal.toLocaleString() + ' 円';

      updateChart(labels, principalData, totalData);
    }

    function updateChart(labels, principalData, totalData) {
      const ctx = document.getElementById('chart').getContext('2d');
      if (myChart) myChart.destroy();

      myChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [
            {
              label: '最終資産額',
              data: totalData,
              borderColor: '#2563eb',
              backgroundColor: 'rgba(37, 99, 235, 0.1)',
              fill: true,
              tension: 0.2
            },
            {
              label: '積立元本',
              data: principalData,
              borderColor: '#94a3b8',
              borderDash: [5, 5],
              fill: false,
              tension: 0
            }
          ]
        },
        options: {
          responsive: true,
          plugins: { legend: { position: 'bottom' } },
          scales: {
            y: {
              ticks: {
                callback: function(value) { return (value / 10000) + '万円'; }
              }
            }
          }
        }
      });
    }

    calculate();
  </script>
</body>
</html>
