
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>ค้นหาเฉพาะคอลัมน์ B, C และ H</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
  <style>
    body { font-family: sans-serif; margin: 1rem; padding: 0; background: #f9f9f9; }
    input, button { font-size: 1rem; }
    table { border-collapse: collapse; width: 100%; margin-top: 1rem; background: white; }
    th, td { border: 1px solid #ccc; padding: 0.4rem 0.6rem; word-break: break-word; }
    input.col-search { width: 100%; box-sizing: border-box; padding: 0.5rem; border: 1px solid #ccc; border-radius: 6px; }
    input#globalSearch { width: 100%; margin-top: 0.5rem; padding: 0.7rem; border: 1px solid #ccc; border-radius: 6px; }
    button { margin-top: 0.5rem; padding: 0.6rem 1.2rem; background-color: #1976d2; color: white; border: none; border-radius: 6px; cursor: pointer; width: 100%; }
    button:hover { background-color: #135ba1; }
    h2 { font-size: 1.2rem; margin-bottom: 0.5rem; text-align: center; }

    @media screen and (max-width: 768px) {
      table, thead, tbody, th, td, tr { display: block; }
      thead tr { display: none; }
      tr { margin-bottom: 1rem; border: 1px solid #ddd; border-radius: 8px; padding: 0.5rem; background: white; }
      td { border: none; position: relative; padding-left: 50%; }
      td::before {
        position: absolute;
        top: 0.4rem;
        left: 0.6rem;
        width: 45%;
        white-space: nowrap;
        font-weight: bold;
      }
    }
  </style>
</head>
<body>

<h2>ค้นหาข้อมูลจากคอลัมน์ B, C และ H</h2>

<input type="text" id="globalSearch" placeholder="ค้นหาแบบรวม (B, C, H)" />
<button id="searchBtn">🔍 ค้นหา</button>

<table id="results">
  <thead></thead>
  <tbody></tbody>
</table>

<script>
  const sheetURL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vReK1auMBXGO9QIj41JHCk6iV5UTfhFeO7_-7xbgy6yek2-k087jFwXusi8X0BJOi1TE2b_tHXDmine/pub?output=csv';
  let data = [];
  let headers = [];
  const searchColumns = ['CB', 'C', 'H'];

  Papa.parse(sheetURL, {
    download: true,
    header: true,
    complete: function(results) {
      data = results.data;
      headers = Object.keys(data[0]);
      createTableWithSearchInputs();
      displayTable(data);
    }
  });

  function createTableWithSearchInputs() {
    const thead = document.querySelector('#results thead');
    thead.innerHTML = '';

    const headerRow = document.createElement('tr');
    headers.forEach(h => {
      const th = document.createElement('th');
      th.textContent = h;
      headerRow.appendChild(th);
    });
    thead.appendChild(headerRow);

    const searchRow = document.createElement('tr');
    headers.forEach(h => {
      const th = document.createElement('th');
      if (searchColumns.includes(h)) {
        const input = document.createElement('input');
        input.className = 'col-search';
        input.placeholder = 'ค้นหา ' + h;
        input.dataset.col = h;
        input.addEventListener('keypress', function(event) {
          if (event.key === 'Enter') {
            event.preventDefault();
            filterData();
          }
        });
        th.appendChild(input);
      }
      searchRow.appendChild(th);
    });
    thead.appendChild(searchRow);
  }

  function displayTable(filteredData) {
    const tbody = document.querySelector('#results tbody');
    tbody.innerHTML = '';

    if (filteredData.length === 0) {
      const tr = document.createElement('tr');
      const td = document.createElement('td');
      td.colSpan = headers.length;
      td.textContent = 'ไม่พบข้อมูล';
      tr.appendChild(td);
      tbody.appendChild(tr);
      return;
    }

    filteredData.forEach(row => {
      const tr = document.createElement('tr');
      headers.forEach(h => {
        const td = document.createElement('td');
        td.setAttribute('data-label', h);
        td.textContent = row[h];
        tr.appendChild(td);
      });
      tbody.appendChild(tr);
    });
  }

  function filterData() {
    const globalInput = document.getElementById('globalSearch').value.trim().toLowerCase();
    const inputs = document.querySelectorAll('.col-search');

    if (globalInput !== '') {
      const filtered = data.filter(row => {
        return searchColumns.some(h => String(row[h] || '').toLowerCase().includes(globalInput));
      });
      displayTable(filtered);
      return;
    }

    const searchTerms = {};
    inputs.forEach(input => {
      const val = input.value.trim().toLowerCase();
      if (val !== '') searchTerms[input.dataset.col] = val;
    });

    const filtered = data.filter(row => {
      return Object.entries(searchTerms).every(([col, val]) => {
        return String(row[col] || '').toLowerCase().includes(val);
      });
    });

    displayTable(filtered);
  }

  document.getElementById('searchBtn').addEventListener('click', filterData);
  document.getElementById('globalSearch').addEventListener('keypress', function(event) {
    if (event.key === 'Enter') {
      event.preventDefault();
      filterData();
    }
  });
</script>

</body>
</html>
