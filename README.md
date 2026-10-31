<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Rental House Water & Trash Tracker</title>
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
    header h1 { font-size: 1.2rem; font-weight: 700; text-align: center; }

    .nav-tabs { display: flex; background: var(--card-bg); border-bottom: 1px solid var(--border); }
    .nav-tab { flex: 1; padding: 12px; text-align: center; font-weight: 600; font-size: 0.9rem; color: var(--text-muted); border: none; background: none; cursor: pointer; }
    .nav-tab.active { color: var(--primary); border-bottom: 3px solid var(--primary); }

    .container { padding: 12px; max-width: 600px; margin: 0 auto; }

    .summary-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; margin-bottom: 12px; }
    .summary-card { background: var(--card-bg); padding: 10px; border-radius: 8px; border: 1px solid var(--border); text-align: center; }
    .summary-card .label { font-size: 0.72rem; color: var(--text-muted); text-transform: uppercase; font-weight: 600; }
    .summary-card .val { font-size: 1.05rem; font-weight: 700; margin-top: 2px; }

    .filter-bar { display: flex; gap: 6px; overflow-x: auto; padding-bottom: 8px; margin-bottom: 12px; }
    .filter-btn { padding: 6px 12px; border-radius: 20px; font-size: 0.8rem; font-weight: 600; white-space: nowrap; border: 1px solid var(--border); background: var(--card-bg); color: var(--text-muted); cursor: pointer; }
    .filter-btn.active { background: var(--primary); color: white; border-color: var(--primary); }

    .house-card { background: var(--card-bg); border-radius: 10px; border: 1px solid var(--border); padding: 12px; margin-bottom: 10px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
    .house-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
    .house-title { font-size: 1.15rem; font-weight: 700; color: var(--primary-dark); }
    
    /* TOGGLE SWITCH STYLING */
    .status-toggle {
      display: inline-flex;
      align-items: center;
      background: #e2e8f0;
      border-radius: 20px;
      padding: 3px;
      cursor: pointer;
      user-select: none;
    }
    .status-toggle span {
      padding: 4px 10px;
      border-radius: 16px;
      font-size: 0.75rem;
      font-weight: 700;
      transition: all 0.2s ease;
    }
    .status-toggle.paid .paid-tab { background: var(--success); color: white; }
    .status-toggle.paid .bado-tab { color: var(--text-muted); }
    .status-toggle.bado .bado-tab { background: var(--danger); color: white; }
    .status-toggle.bado .paid-tab { color: var(--text-muted); }

    .input-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 8px; }
    .input-group label { display: block; font-size: 0.7rem; color: var(--text-muted); margin-bottom: 2px; font-weight: 600; }
    .input-group input { width: 100%; padding: 8px; border: 1px solid var(--border); border-radius: 6px; font-size: 1rem; font-weight: 600; text-align: center; }

    .calc-box { background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 8px; padding: 8px 10px; font-size: 0.85rem; }
    .calc-row-detail { display: flex; justify-content: space-between; color: var(--text-muted); margin-bottom: 4px; }
    .calc-row-total { display: flex; justify-content: space-between; border-top: 1px dashed var(--border); padding-top: 6px; font-weight: 700; font-size: 0.95rem; color: var(--text); }

    .btn-action { width: 100%; padding: 12px; border-radius: 8px; border: none; font-weight: 700; font-size: 0.95rem; cursor: pointer; margin-top: 8px; }
    .btn-primary { background: var(--primary); color: white; }
    .btn-outline { background: transparent; border: 1px solid var(--primary); color: var(--primary); }

    .history-card { background: var(--card-bg); border-radius: 8px; border: 1px solid var(--border); padding: 12px; margin-bottom: 10px; font-size: 0.85rem; }
    .history-header { display: flex; justify-content: space-between; font-weight: 700; margin-bottom: 6px; color: var(--primary-dark); }
    .history-body { color: var(--text); margin-bottom: 8px; }
    .history-actions { display: flex; gap: 8px; justify-content: flex-end; border-top: 1px solid var(--border); padding-top: 8px; }

    .btn-sm { padding: 4px 10px; font-size: 0.75rem; font-weight: 600; border-radius: 4px; border: none; cursor: pointer; }
    .btn-edit { background: #e0f2fe; color: #0284c7; }
    .btn-delete { background: var(--danger-bg); color: var(--danger); }

    select { width: 100%; padding: 10px; border-radius: 8px; border: 1px solid var(--border); font-size: 1rem; font-weight: 600; margin-bottom: 12px; background: white; }
  </style>
</head>
<body>

  <header>
    <h1>Rental Water & Trash Collector</h1>
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
