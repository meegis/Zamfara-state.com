<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Admin – Zamfara Grant Applications</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body data-page="admin">
  <header class="site-header">
    <img src="slogan.jpg" alt="Zamfara State Government" class="logo" />
    <div>
      <h1>Admin Panel</h1>
      <p class="slogan">Financial Grant – Applications</p>
    </div>
    <img src="zamfara-governor.jpg" alt="Governor of Zamfara State" class="governor" />
  </header>

  <main class="container">
    <nav class="tabs">
      <a class="tab" href="index.html">Apply</a>
      <a class="tab active">Admin / Download Data</a>
    </nav>

    <section class="card">
      <div class="toolbar">
        <input type="text" id="search" placeholder="Search by name…" />
        <input type="text" id="filter-lga" placeholder="Filter by LGA…" />
        <button id="download-csv-all" class="btn">Download CSV</button>
        <button id="download-json-all" class="btn">Download JSON</button>
        <button id="clear-all" class="btn danger">Clear All</button>
      </div>

      <div class="table-wrap">
        <table id="apps-table">
          <thead>
            <tr>
              <th>Passport</th>
              <th>Name</th>
              <th>Phone</th>
              <th>LGA</th>
              <th>Advice</th>
              <th>Submitted</th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>
      </div>
    </section>
  </main>

  <footer class="site-footer">© <span id="year"></span> Zamfara State Government</footer>

  <script src="script.js"></script>
</body>
</html>
