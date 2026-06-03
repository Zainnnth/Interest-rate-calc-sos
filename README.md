<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Credit Control Interest Calculator</title>
<link rel="stylesheet" href="css/styles.css">
</head>
<body>
<main>
  <h1>Credit Control Interest Calculator</h1>
  <p class="muted">Static GitHub Pages version. All processing happens in the browser; no backend required.</p>

  <section class="box">
    <strong>Latest embedded rate:</strong> <span id="latest"></span>
  </section>

  <section class="box">
    <h2>Calculation Details</h2>
    <label>Client <input id="client" size="34" autocomplete="off"></label>
    <label>Matter Ref <input id="matter" size="22" autocomplete="off"></label>
    <label>Calculation Date <input id="calcDate" type="date"></label>
    <label>Terms Days <input id="terms" type="number" value="28" min="0" style="width:90px"></label>
    <button id="addBillBtn" type="button">Add Bill</button>
    <button id="recalcBtn" type="button">Recalculate</button>
    <button id="clearBtn" type="button" class="danger">Clear</button>
  </section>

  <section id="importBox" class="box noprint">
    <h2>Import SOS / Crystal Ledger Export</h2>
    <p class="muted">Use this for exported RTF/DOC/TXT/CSV ledger reports. Large Crystal RTF files are parsed in a worker so the page should not lock up.</p>
    <input type="file" id="file" accept=".doc,.rtf,.txt,.csv">
    <button id="parseBtn" type="button" class="secondary">Parse Pasted Text</button>
    <label style="display:block">Paste ledger export text
      <textarea id="paste"></textarea>
    </label>
    <div id="parseMsg" class="status">Ready.</div>
    <div class="progress"><div id="parseBar" class="bar"></div></div>
  </section>

  <section class="box">
    <h2>Bills</h2>
    <div class="table-wrap">
      <table id="bills">
        <thead><tr>
          <th>Use</th><th>Bill Ref</th><th>Bill Date</th><th>Narrative</th><th>Bill Amount</th><th>Outstanding</th><th>Due Date</th><th>Days</th><th>Interest</th><th>Total</th><th></th>
        </tr></thead>
        <tbody></tbody>
      </table>
    </div>
  </section>

  <section class="box">
    <h2>Totals</h2>
    <div class="totals">
      <div class="total">Outstanding <b id="principal">£0.00</b></div>
      <div class="total">Interest <b id="interest">£0.00</b></div>
      <div class="total">Total Claim <b id="total">£0.00</b></div>
    </div>
    <button type="button" onclick="window.print()">Print / Save PDF</button>
    <button id="exportBtn" type="button" class="secondary">Export CSV</button>
  </section>

  <section class="box">
    <h2>Report</h2>
    <pre id="report"></pre>
  </section>

  <section id="rateBox" class="box noprint">
    <h2>Rate Maintenance</h2>
    <label>Effective Date <input id="rateDate" type="date"></label>
    <label>Base Rate % <input id="rateVal" type="number" step="0.01"></label>
    <button id="addRateBtn" type="button">Add/Update Rate</button>
    <button id="showRatesBtn" type="button" class="secondary">Show Rates</button>
    <div id="ratesOut"></div>
  </section>
</main>
<script src="js/rates.js"></script>
<script src="js/app.js"></script>
</body>
</html>
