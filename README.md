<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Team Performance Dashboard</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <link rel="stylesheet" href="styles.css" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
</head>
<body>
  <div class="container">
    <header>
      <h1>Team Performance Dashboard</h1>
      <button id="reloadBtn" title="Reload from Google Sheet">🔄 Reload Live Data</button>
      <label class="upload-label">
        Upload Excel
        <input type="file" id="excelInput" accept=".xlsx"/>
      </label>
      <span id="status"></span>
    </header>
    <section>
      <h2>Performance Table</h2>
      <div class="table-responsive">
        <table id="performanceTable">
          <thead>
            <tr>
              <th>Employee Name</th>
              <th>Monthly Target</th>
              <th>Pending Items</th>
            </tr>
          </thead>
          <tbody>
            <!-- Data Rows Here -->
          </tbody>
        </table>
      </div>
    </section>
    <section class="charts">
      <div>
        <h2>Pending Items per Agent</h2>
        <canvas id="pendingChart"></canvas>
      </div>
      <div>
        <h2>Target vs Pending</h2>
        <canvas id="targetChart"></canvas>
      </div>
    </section>
  </div>
  <script src="app.js"></script>
</body>
</html>
