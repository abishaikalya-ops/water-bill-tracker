<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Rental House Water Tracker & History</title>
  <style>
    :root {
      --primary: #0284c7;
      --primary-dark: #0369a1;
      --bg: #f8fafc;
      --card-bg: #ffffff;
      --text: #0f172a;
      --text-muted: #64748b;
      --border: #e2e8f0;
      --success: #16a34a;
      --success-bg: #dcfce7;
      --danger: #dc2626;
      --danger-bg: #fee2e2;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
    body { background-color: var(--bg); color: var(--text); padding-bottom: 80px; }

    header {
      background: linear-gradient(135deg, var(--primary), var(--primary-dark));
      color: white; padding: 16px; position: sticky; top: 0; z-index: 100;
      box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1);
    }
    header h1 { font-size: 1.25rem; font-weight: 700; text-align: center; }

    .nav-tabs { display: flex; background: var(--card-bg); border-bottom: 1px solid var(--border); }
    .nav-tab { flex: 1; padding: 12px; text-align: center; font-weight: 600; font-size: 0.9rem; color: var(--text-muted); border: none; background: none; cursor: pointer; }
    .nav-tab.active { color: var(--primary); border-bottom: 3px solid var(--primary); }

    .container { padding: 12px; max-width: 600px; margin: 0 auto; }

    .summary-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; margin-bottom: 12px; }
    .summary-card { background: var(--card-bg); padding: 10px; border-radius: 8px; border: 1px solid var(--border); text-align: center; }
    .summary-card .label { font-size: 0.75rem; color: var(--text-muted); text-transform: uppercase; font-weight: 600; }
    .summary-card .val { font-size: 1.1rem; font-weight: 700; margin-top: 2px; }

    .filter-bar { display: flex; gap: 6px; overflow-x: auto; padding-bottom: 8px; margin-bottom: 12px; }
    .filter-btn { padding: 6px 12px; border-radius: 20px; font-size: 0.8rem; font-weight: 600; white-space: nowrap; border: 1px solid var(--border); background: var(--card-bg); color: var(--text-muted); cursor: pointer; }
    .filter-btn.active { background: var(--primary); color: white; border-color: var(--primary); }

    .house-card { background: var(--card-bg); border-radius: 10px; border: 1px solid var(--border); padding: 12px; margin-bottom: 10px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
    .house-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
    .house-title { font-size: 1.1rem; font-weight: 700; color: var(--primary-dark); }
    
    .status-badge { padding: 4px 10px; border-radius: 12px; font-size: 0.75rem; font-weight: 700; cursor: pointer; border: none; }
    .status-badge.paid { background: var(--success-bg); color: var(--success); }
    .status-badge.bado { background: var(--danger-bg); color: var(--danger); }

    .input-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 8px; }
    .input-group label { display: block; font-size: 0.7rem; color: var(--text-muted); margin-bottom: 2px; font-weight: 600; }
    .input-group input { width: 100%; padding: 8px; border: 1px solid var(--border); border-radius: 6px; font-size: 1rem; font-weight: 600; text-align: center; }

    .calc-row { display: flex; justify-content: space-between; background: #f1f5f9; padding: 6px 10px; border-radius: 6px; font-size: 0.85rem; font-weight: 600; }

    .btn-action { width: 100%; padding: 12px; border-radius: 8px; border: none; font-weight: 700; font-size: 0.95rem; cursor: pointer; margin-top: 8px; }
    .btn-primary { background: var(--primary); color: white; }
    .btn-outline { background: transparent; border: 1px solid var(--primary); color: var(--primary); }

    .history-card { background: var(--card-bg); border-radius: 8px; border: 1px solid var(--border); padding: 12px; margin-bottom: 10px; font-size: 0.85rem; }
    .history-header { display: flex; justify-content: space-between; font-weight: 700; margin-bottom: 6px; color: var(--primary-dark); }
    .history-body { display: flex; justify-content: space-between; align-items: center; color: var(--text); margin-bottom: 8px; }
    .history-actions { display: flex; gap: 8px; justify-content: flex-end; border-top: 1px solid var(--border); padding-top: 8px; }

    .btn-sm { padding: 4px 10px; font-size: 0.75rem; font-weight: 600; border-radius: 4px; border: none; cursor: pointer; }
    .btn-edit { background: #e0f2fe; color: #0284c7; }
    .btn-delete { background: var(--danger-bg); color: var(--danger); }

    select { width: 100%; padding: 10px; border-radius: 8px; border: 1px solid var(--border); font-size: 1rem; font-weight: 600; margin-bottom: 12px; background: white; }
  </style>
</head>
<body>

  <header>
    <h1>Rental Water Billing & History</h1>
  </header>

  <div class="nav-tabs">
    <button class="nav-tab active" onclick="switchView('entry')">Current Month</button>
    <button class="nav-tab" onclick="switchView('history')">House History</button>
  </div>

  <!-- CURRENT ENTRY VIEW -->
  <div id="view-entry" class="container">
    
    <div class="summary-grid">
      <div class="summary-card">
        <div class="label">Total Billed</div>
        <div class="val" id="total-billed">KSh 0</div>
      </div>
      <div class="summary-card">
        <div class="label">Total Paid</div>
        <div class="val" style="color: var(--success);" id="total-paid">KSh 0</div>
      </div>
      <div class="summary-card">
        <div class="label">Total Bado</div>
        <div class="val" style="color: var(--danger);" id="total-bado">KSh 0</div>
      </div>
      <div class="summary-card">
        <div class="label">Rate / Unit</div>
        <div class="val">KSh <input type="number" id="rate-input" value="250" style="width: 50px; text-align: center; font-size: 1rem; font-weight:700;" onchange="renderApp()"></div>
      </div>
    </div>

    <div class="filter-bar">
      <button class="filter-btn active" onclick="setBlock('ALL', this)">All</button>
      <button class="filter-btn" onclick="setBlock('GND', this)">Ground (1-10)</button>
      <button class="filter-btn" onclick="setBlock('B', this)">Block B</button>
      <button class="filter-btn" onclick="setBlock('C', this)">Block C</button>
      <button class="filter-btn" onclick="setBlock('D', this)">Block D</button>
      <button class="filter-btn" onclick="setBlock('E', this)">Block E</button>
    </div>

    <div id="houses-container"></div>

    <button class="btn-action btn-primary" onclick="closeAndSaveMonth()">Save & Start New Month</button>
  </div>

  <!-- HISTORY VIEW -->
  <div id="view-history" class="container" style="display: none;">
    <label style="font-size: 0.8rem; font-weight: 700; color: var(--text-muted);">SELECT HOUSE FOR HISTORY:</label>
    <select id="history-house-select" onchange="renderHistory()">
      <!-- Populated dynamically -->
    </select>

    <div id="history-container"></div>

    <button class="btn-action btn-outline" onclick="exportCSV()">Export All History (CSV)</button>
  </div>

  <script>
    const BLOCKS = {
      'GND': Array.from({length: 10}, (_, i) => `${i + 1}`),
      'B': Array.from({length: 10}, (_, i) => `${i + 1}B`),
      'C': Array.from({length: 10}, (_, i) => `${i + 1}C`),
      'D': Array.from({length: 10}, (_, i) => `${i + 1}D`),
      'E': Array.from({length: 10}, (_, i) => `${i + 1}E`),
    };

    const INITIAL_LEDGER = {
      '1D': { prev: 39, curr: 40, status: 'Paid' },
      '2D': { prev: 36, curr: 41, status: 'Paid' },
      '3D': { prev: 34, curr: 36, status: 'Paid' },
      '4D': { prev: 38, curr: 42, status: 'Paid' },
      '5D': { prev: 39, curr: 40, status: 'Paid' },
      '6D': { prev: 22, curr: 23, status: 'Paid' },
      '7D': { prev: 23, curr: 25, status: 'Paid' },
      '8D': { prev: 32, curr: 34, status: 'Paid' },
      '9D': { prev: 39, curr: 43, status: 'Paid' },
      '10D': { prev: 0, curr: 0, status: 'Bado' },
      '1E': { prev: 29, curr: 31, status: 'Paid' },
      '2E': { prev: 30, curr: 32, status: 'Paid' },
      '3E': { prev: 36, curr: 37, status: 'Paid' },
      '4E': { prev: 24, curr: 26, status: 'Paid' },
      '5E': { prev: 18, curr: 21, status: 'Bado' },
      '6E': { prev: 33, curr: 35, status: 'Paid' },
      '7E': { prev: 40, curr: 41, status: 'Paid' },
      '8E': { prev: 31, curr: 33, status: 'Bado' },
      '9E': { prev: 36, curr: 38, status: 'Paid' },
      '10E': { prev: 0, curr: 0, status: 'Bado' }
    };

    let houses = JSON.parse(localStorage.getItem('water_app_houses')) || {};
    let historyLogs = JSON.parse(localStorage.getItem('water_app_history')) || [];
    let activeBlock = 'ALL';

    function initData() {
      Object.keys(BLOCKS).forEach(b => {
        BLOCKS[b].forEach(h => {
          if (!houses[h]) {
            houses[h] = INITIAL_LEDGER[h] || { prev: 0, curr: 0, status: 'Bado' };
          }
        });
      });
      saveHouses();
      populateHistoryDropdown();
    }

    function saveHouses() { localStorage.setItem('water_app_houses', JSON.stringify(houses)); }
    function saveHistory() { localStorage.setItem('water_app_history', JSON.stringify(historyLogs)); }

    function switchView(view) {
      document.getElementById('view-entry').style.display = view === 'entry' ? 'block' : 'none';
      document.getElementById('view-history').style.display = view === 'history' ? 'block' : 'none';
      document.querySelectorAll('.nav-tab').forEach((tab, idx) => {
        tab.classList.toggle('active', (view === 'entry' && idx === 0) || (view === 'history' && idx === 1));
      });
      if (view === 'history') renderHistory();
    }

    function setBlock(block, btn) {
      activeBlock = block;
      document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      renderApp();
    }

    function updateVal(house, field, val) {
      houses[house][field] = Number(val) || 0;
      saveHouses();
      renderApp();
    }

    function toggleStatus(house) {
      houses[house].status = houses[house].status === 'Paid' ? 'Bado' : 'Paid';
      saveHouses();
      renderApp();
    }

    function renderApp() {
      const container = document.getElementById('houses-container');
      const rate = Number(document.getElementById('rate-input').value) || 250;
      container.innerHTML = '';

      let totalBilled = 0, totalPaid = 0, totalBado = 0;

      Object.keys(houses).forEach(h => {
        const belongsToBlock = activeBlock === 'ALL' || BLOCKS[activeBlock]?.includes(h);
        const data = houses[h];
        const consumption = Math.max(0, data.curr - data.prev);
        const amount = consumption * rate;

        totalBilled += amount;
        if (data.status === 'Paid') totalPaid += amount;
        else totalBado += amount;

        if (!belongsToBlock) return;

        const card = document.createElement('div');
        card.className = 'house-card';
        card.innerHTML = `
          <div class="house-header">
            <span class="house-title">House ${h}</span>
            <button class="status-badge ${data.status.toLowerCase()}" onclick="toggleStatus('${h}')">${data.status}</button>
          </div>
          <div class="input-grid">
            <div class="input-group">
              <label>PREVIOUS</label>
              <input type="number" value="${data.prev}" onchange="updateVal('${h}', 'prev', this.value)">
            </div>
            <div class="input-group">
              <label>CURRENT</label>
              <input type="number" value="${data.curr}" onchange="updateVal('${h}', 'curr', this.value)">
            </div>
          </div>
          <div class="calc-row">
            <span>Units: <b>${consumption}</b></span>
            <span>Amount: <b>KSh ${amount.toLocaleString()}</b></span>
          </div>
        `;
        container.appendChild(card);
      });

      document.getElementById('total-billed').innerText = `KSh ${totalBilled.toLocaleString()}`;
      document.getElementById('total-paid').innerText = `KSh ${totalPaid.toLocaleString()}`;
      document.getElementById('total-bado').innerText = `KSh ${totalBado.toLocaleString()}`;
    }

    function closeAndSaveMonth() {
      if (!confirm("Are you sure you want to save this month to History and start a new month?")) return;

      const monthName = prompt("Enter Month & Year (e.g., October 2026):", new Date().toLocaleString('default', { month: 'long', year: 'numeric' }));
      if (!monthName) return;

      const rate = Number(document.getElementById('rate-input').value) || 250;

      Object.keys(houses).forEach(h => {
        const data = houses[h];
        const consumption = Math.max(0, data.curr - data.prev);
        const amount = consumption * rate;

        historyLogs.push({
          id: Date.now() + Math.random(),
          month: monthName,
          house: h,
          prev: data.prev,
          curr: data.curr,
          consumption: consumption,
          amount: amount,
          status: data.status,
          dateSaved: new Date().toLocaleDateString()
        });

        // Rollover: Current reading becomes Previous reading for next month
        data.prev = data.curr;
        data.status = 'Bado';
      });

      saveHistory();
      saveHouses();
      renderApp();
      alert("Month archived successfully to History!");
    }

    function populateHistoryDropdown() {
      const select = document.getElementById('history-house-select');
      select.innerHTML = '';
      Object.keys(houses).forEach(h => {
        const opt = document.createElement('option');
        opt.value = h;
        opt.innerText = `House ${h}`;
        select.appendChild(opt);
      });
    }

    function renderHistory() {
      const selectedHouse = document.getElementById('history-house-select').value;
      const container = document.getElementById('history-container');
      container.innerHTML = '';

      const houseRecords = historyLogs.filter(log => log.house === selectedHouse);

      if (houseRecords.length === 0) {
        container.innerHTML = `<p style="text-align: center; color: var(--text-muted); padding: 20px;">No historical records saved for House ${selectedHouse} yet.</p>`;
        return;
      }

      houseRecords.slice().reverse().forEach(record => {
        const card = document.createElement('div');
        card.className = 'history-card';
        card.innerHTML = `
          <div class="history-header">
            <span>${record.month}</span>
            <span style="color: ${record.status === 'Paid' ? 'var(--success)' : 'var(--danger)'}">${record.status}</span>
          </div>
          <div class="history-body">
            <span>Readings: ${record.prev} → ${record.curr} (${record.consumption} units)</span>
            <b>KSh ${record.amount.toLocaleString()}</b>
          </div>
          <div class="history-actions">
            <button class="btn-sm btn-edit" onclick="editHistoryItem(${record.id})">Edit</button>
            <button class="btn-sm btn-delete" onclick="deleteHistoryItem(${record.id})">Delete</button>
          </div>
        `;
        container.appendChild(card);
      });
    }

    function editHistoryItem(id) {
      const item = historyLogs.find(r => r.id === id);
      if (!item) return;

      const newPrev = prompt(`House ${item.house} - Previous Reading:`, item.prev);
      if (newPrev === null) return;

      const newCurr = prompt(`House ${item.house} - Current Reading:`, item.curr);
      if (newCurr === null) return;

      const newStatus = prompt(`Status (Paid / Bado):`, item.status);
      if (newStatus === null) return;

      const rate = Number(document.getElementById('rate-input').value) || 250;
      const prevNum = Number(newPrev) || 0;
      const currNum = Number(newCurr) || 0;
      const consumption = Math.max(0, currNum - prevNum);

      item.prev = prevNum;
      item.curr = currNum;
      item.consumption = consumption;
      item.amount = consumption * rate;
      item.status = newStatus.trim().toLowerCase() === 'paid' ? 'Paid' : 'Bado';

      saveHistory();
      renderHistory();
      alert("Record updated!");
    }

    function deleteHistoryItem(id) {
      if (!confirm("Are you sure you want to delete this historical record?")) return;

      historyLogs = historyLogs.filter(r => r.id !== id);
      saveHistory();
      renderHistory();
    }

    function exportCSV() {
      if (historyLogs.length === 0) {
        alert("No history available to export yet.");
        return;
      }
      let csv = "Month,House,Previous,Current,Consumption,Amount,Status\n";
      historyLogs.forEach(r => {
        csv += `"${r.month}","${r.house}",${r.prev},${r.curr},${r.consumption},${r.amount},"${r.status}"\n`;
      });

      const blob = new Blob([csv], { type: 'text/csv' });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `Water_Billing_History.csv`;
      a.click();
    }

    initData();
    renderApp();
  </script>
</body>
</html># water-bill-tracker
