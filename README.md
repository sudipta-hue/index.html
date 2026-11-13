# index.html
Dashboard
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>IDN Projects Monitoring Dashboard</title>
  <script src="https://tailwindcss.com"></script>
  <script src="https://jsdelivr.net"></script>
  <script src="https://jsdelivr.net"></script>
  <style>
    body { font-family: 'Inter', system-ui, -apple-system, sans-serif; background-color: #f8fafc; }
    .card-shadow { box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.04); }
  </style>
</head>
<body class="text-slate-800">

  <!-- Top Navigation Header -->
  <header class="bg-slate-900 text-white sticky top-0 z-30 shadow-md">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3">
      <div>
        <div class="flex items-center gap-2">
          <span class="bg-blue-600 text-xs font-bold px-2.5 py-1 rounded uppercase tracking-wider">ID North</span>
          <h1 class="text-xl font-bold tracking-tight">IDN Projects Executive Dashboard</h1>
        </div>
        <p class="text-xs text-slate-400 mt-0.5">Live Google Sheets Sync • HITES Infrastructure Monitoring</p>
      </div>

      <div class="flex flex-wrap items-center gap-3">
        <div id="syncStatus" class="flex items-center gap-2 text-xs bg-slate-800 px-3 py-1.5 rounded-full border border-slate-700">
          <span id="statusDot" class="h-2.5 w-2.5 rounded-full bg-amber-400 animate-pulse"></span>
          <span id="statusText">Connecting to Sheet...</span>
        </div>

        <label class="flex items-center gap-1.5 text-xs text-slate-300 cursor-pointer select-none">
          <input type="checkbox" id="autoRefreshToggle" checked class="rounded text-blue-500 focus:ring-0" />
          Auto-Sync (60s)
        </label>

        <button onclick="fetchSheetData()" class="bg-blue-600 hover:bg-blue-500 active:bg-blue-700 text-white text-xs font-semibold px-3.5 py-2 rounded-lg transition flex items-center gap-1.5">
          <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
          </svg>
          Sync Now
        </button>
      </div>
    </div>
  </header>

  <!-- Main Content -->
  <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

    <!-- Error / Permission Banner (Hidden unless sheet is private) -->
    <div id="errorBanner" class="hidden bg-red-50 border-l-4 border-red-500 p-4 rounded-r-lg text-sm text-red-800">
      <p class="font-bold">Live Sync Notice</p>
      <p id="errorMessage" class="mt-1">Could not fetch live data. Ensure Google Sheet sharing is set to "Anyone with the link can view" and published to web if required.</p>
    </div>

    <!-- Filters Bar -->
    <section class="bg-white p-4 rounded-xl card-shadow border border-slate-200/80 flex flex-col md:flex-row gap-3 items-stretch md:items-center justify-between">
      <div class="flex flex-col sm:flex-row gap-3 flex-1">
        <input
          type="text"
          id="searchInput"
          placeholder="Search project, contractor, client, or location..."
          oninput="applyFilters()"
          class="w-full sm:w-80 px-3.5 py-2 text-sm border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <select id="zoneFilter" onchange="applyFilters()" class="px-3.5 py-2 text-sm border border-slate-300 rounded-lg bg-white focus:outline-none focus:ring-2 focus:ring-blue-500">
          <option value="ALL">All Zones</option>
        </select>
        <select id="delayFilter" onchange="applyFilters()" class="px-3.5 py-2 text-sm border border-slate-300 rounded-lg bg-white focus:outline-none focus:ring-2 focus:ring-blue-500">
          <option value="ALL">All Statuses</option>
          <option value="ON_TIME">On Schedule (0 Delay)</option>
          <option value="DELAYED">Delayed (&gt; 0 Months)</option>
          <option value="CRITICAL">Critical Delay (&gt; 12 Months)</option>
        </select>
      </div>
      <div class="text-xs text-slate-500 font-medium text-right" id="lastUpdatedText">
        Last synced: --
      </div>
    </section>

    <!-- KPI Summary Cards -->
    <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-4">
      <div class="bg-white p-4 rounded-xl card-shadow border border-slate-200/80">
        <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Active Projects</p>
        <p id="kpiCount" class="text-2xl font-bold text-slate-900 mt-1">0</p>
        <p id="kpiDelayedSub" class="text-xs text-amber-600 mt-1">-- delayed</p>
      </div>

      <div class="bg-white p-4 rounded-xl card-shadow border border-slate-200/80">
        <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Approved DPR Cost</p>
        <p id="kpiDprCost" class="text-2xl font-bold text-slate-900 mt-1">₹0 Cr</p>
        <p class="text-xs text-slate-400 mt-1">Sanctioned budget</p>
      </div>

      <div class="bg-white p-4 rounded-xl card-shadow border border-slate-200/80">
        <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Total LoA Amount</p>
        <p id="kpiLoaCost" class="text-2xl font-bold text-blue-700 mt-1">₹0 Cr</p>
        <p class="text-xs text-slate-400 mt-1">Awarded contract value</p>
      </div>

      <div class="bg-white p-4 rounded-xl card-shadow border border-slate-200/80">
        <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Work Certified</p>
        <p id="kpiCertified" class="text-2xl font-bold text-emerald-600 mt-1">₹0 Cr</p>
        <p id="kpiFinPct" class="text-xs text-slate-500 mt-1">0% overall billed</p>
      </div>

      <div class="bg-white p-4 rounded-xl card-shadow border border-slate-200/80">
        <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">Avg Physical Progress</p>
        <p id="kpiPhysPct" class="text-2xl font-bold text-indigo-600 mt-1">0%</p>
        <p id="kpiFundBal" class="text-xs text-slate-500 mt-1">Fund Bal: ₹0 Cr</p>
      </div>
    </section>

    <!-- Charts Section -->
    <section class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <div class="bg-white p-5 rounded-xl card-shadow border border-slate-200/80">
        <h2 class="text-sm font-bold text-slate-800 mb-3">Zone-Wise LoA Amount vs Work Certified (₹ Cr)</h2>
        <div class="h-64">
          <canvas id="zoneFinancialChart"></canvas>
        </div>
      </div>

      <div class="bg-white p-5 rounded-xl card-shadow border border-slate-200/80">
        <h2 class="text-sm font-bold text-slate-800 mb-3">Physical Progress: Planned % vs Achieved % (Top Projects)</h2>
        <div class="h-64">
          <canvas id="progressChart"></canvas>
        </div>
      </div>
    </section>

    <!-- Projects Data Table -->
    <section class="bg-white rounded-xl card-shadow border border-slate-200/80 overflow-hidden">
      <div class="px-5 py-4 border-b border-slate-200 flex items-center justify-between">
        <h2 class="text-sm font-bold text-slate-800">Project Portfolio Details</h2>
        <span class="text-xs text-slate-500">Click any project row to view concerns, team &amp; mitigation actions</span>
      </div>
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse">
          <thead>
            <tr class="bg-slate-50 text-slate-600 text-xs uppercase tracking-wider border-b border-slate-200">
              <th class="py-3 px-4 font-semibold">#</th>
              <th class="py-3 px-4 font-semibold">Project &amp; Client</th>
              <th class="py-3 px-4 font-semibold">Zone</th>
              <th class="py-3 px-4 font-semibold">Contractor</th>
              <th class="py-3 px-4 font-semibold text-right">LoA (Cr)</th>
              <th class="py-3 px-4 font-semibold text-right">Certified (Cr)</th>
              <th class="py-3 px-4 font-semibold">Physical Progress</th>
              <th class="py-3 px-4 font-semibold text-center">Delay (Mo)</th>
              <th class="py-3 px-4 font-semibold text-center">Details</th>
            </tr>
          </thead>
          <tbody id="projectsTableBody" class="divide-y divide-slate-100 text-sm">
            <tr>
              <td colspan="9" class="py-8 text-center text-slate-400">Loading live data from Google Sheets...</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </main>

  <!-- Project Detail Modal -->
  <div id="detailModal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-xs z-50 flex items-center justify-center p-4">
    <div class="bg-white rounded-2xl max-w-4xl w-full max-h-[90vh] overflow-y-auto shadow-2xl border border-slate-200">
      <div class="sticky top-0 bg-slate-900 text-white px-6 py-4 flex items-start justify-between gap-4">
        <div>
          <span id="modalZoneBadge" class="text-xs font-semibold bg-blue-600 px-2.5 py-0.5 rounded">Zone</span>
          <h3 id="modalTitle" class="text-lg font-bold mt-1">Project Name</h3>
          <p id="modalSubtitle" class="text-xs text-slate-300 mt-0.5">Client • Contractor</p>
        </div>
        <button onclick="closeModal()" class="text-slate-400 hover:text-white text-2xl leading-none">&times;</button>
      </div>

      <div id="modalBody" class="p-6 space-y-5 text-sm">
        <!-- Populated dynamically -->
      </div>
    </div>
  </div>

  <script>
    const SHEET_ID = "149bSf_wDVsF0H1fRprVPMUvQdp5Z6NjjO9DE1ehxJVU";
    const CSV_EXPORT_URL = `https://google.com{SHEET_ID}/gviz/tq?tqx=out:csv`;

    let allProjects = [];
    let filteredProjects = [];
    let zoneChartInstance = null;
