# Ciggerate-tracker-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Cigarette Tracker</title>
  <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <h1>Cigarette Tracker</h1>
  <label for="cigType">Cigarette Type:</label>
  <select id="cigType">
    <option value="Classic">Classic</option>
    <option value="Gold Flake">Gold Flake</option>
    <option value="Marlboro">Marlboro</option>
    <option value="Custom">Custom</option>
  </select>

  <div id="customCig" style="display: none;">
    <input type="text" id="customCigName" placeholder="Custom Cigarette Name">
    <button onclick="addCustomCigarette()">Add Custom Cigarette</button>
  </div>

  <label for="cigCount">Number of Cigarettes:</label>
  <input type="number" id="cigCount" min="1" value="1">

  <label for="cigPrice">Price per Cigarette (₹):</label>
  <input type="number" id="cigPrice" min="10" max="20" value="15">

  <button onclick="logCigarette()">Log Cigarette</button>
  <button onclick="resetDailySummary()">Reset Daily Summary</button>

  <table>
    <thead>
      <tr>
        <th>Cigarette Type</th>
        <th>Quantity</th>
        <th>Total Cost (₹)</th>
        <th>Last Entry</th>
      </tr>
    </thead>
    <tbody id="summaryTableBody">
      <tr>
        <td colspan="4">No data recorded yet.</td>
      </tr>
    </tbody>
  </table>
  <script src="js/script.js"></script>
</body>
</html>
```

**script.js**:

```javascript
let totalCount = 0;
let totalCost = 0;
let typeTracker = {};
let customCigarettes = JSON.parse(localStorage.getItem('customCigarettes')) || [];

document.getElementById('cigType').addEventListener('change', function() {
  document.getElementById('customCig').style.display = this.value === 'Custom' ? 'block' : 'none';
});

function loadCustomCigarettes() {
  const cigTypeSelect = document.getElementById('cigType');
  customCigarettes.forEach(customCig => {
    const option = document.createElement('option');
    option.value = customCig;
    option.textContent = customCig;
    cigTypeSelect.appendChild(option);
  });
}

function addCustomCigarette() {
  const customCigName = document.getElementById('customCigName').value.trim();
  if (customCigName && !customCigarettes.includes(customCigName)) {
    customCigarettes.push(customCigName);
    localStorage.setItem('customCigarettes', JSON.stringify(customCigarettes));
    const option = document.createElement('option');
    option.value = customCigName;
    option.textContent = customCigName;
    document.getElementById('cigType').appendChild(option);
    document.getElementById('cigType').value = customCigName;
    document.getElementById('customCigName').value = '';
  }
}

function logCigarette() {
  const cigTypeSelect = document.getElementById('cigType');
  const cigCount = parseInt(document.getElementById('cigCount').value);
  const cigPrice = parseFloat(document.getElementById('cigPrice').value);
  const now = new Date();

  let cigType = cigTypeSelect.value;

  totalCount += cigCount;
  totalCost += cigCount * cigPrice;
  typeTracker[cigType] = (typeTracker[cigType] || 0) + cigCount;

  updateSummaryTable(now);
  saveData();
}

function updateSummaryTable(now) {
  const summaryTableBody = document.getElementById('summaryTableBody');
  summaryTableBody.innerHTML = ''; // Clear previous entries

  for (const [type, count] of Object.entries(typeTracker)) {
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${type}</td>
      <td>${count}</td>
      <td>₹${(count * parseFloat(document.getElementById('cigPrice').value)).toFixed(2)}</td>
      <td>${now.toLocaleString()}</td>
    `;
    summaryTableBody.appendChild(row);
  }
}

function resetDailySummary() {
  totalCount = 0;
  totalCost = 0;
  typeTracker = {};

  document.getElementById('summaryTableBody').innerHTML = '<tr><td colspan="4">No data recorded yet.</td></tr>';
  localStorage.removeItem('cigaretteData');
}

function saveData() {
  const data = {
    totalCount,
    totalCost,
    typeTracker,
  };
  localStorage.setItem('cigaretteData', JSON.stringify(data));
}

function loadData() {
  const data = JSON.parse(localStorage.getItem('cigaretteData'));
  if (data) {
    totalCount = data.totalCount;
    totalCost = data.totalCost;
    typeTracker = data.typeTracker;

    updateSummaryTable(new Date());
  }
  loadCustomCigarettes();
}

window.onload = loadData;
