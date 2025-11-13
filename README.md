# clearance
32 Flavors Clearance Program
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Clearance Prototype</title>
  <style>
    body {
      font-family: system-ui, sans-serif;
      margin: 0;
      background: #f5f5f5;
    }
    .app {
      max-width: 1100px;
      margin: 0 auto;
      padding: 16px;
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: baseline;
    }
    .card {
      background: #fff;
      border-radius: 8px;
      padding: 12px 16px;
      margin-top: 16px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.08);
    }
    .row {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      align-items: center;
      margin-bottom: 8px;
    }
    input, select, button, textarea {
      font: inherit;
    }
    input, select, textarea {
      padding: 4px 6px;
    }
    button {
      padding: 4px 10px;
      border-radius: 4px;
      border: 1px solid #ccc;
      background: #fff;
    }
    button.primary {
      background: #2563eb;
      border-color: #2563eb;
      color: #fff;
    }
    button.danger {
      background: #dc2626;
      border-color: #b91c1c;
      color: #fff;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 14px;
    }
    th, td {
      padding: 6px 8px;
      border-bottom: 1px solid #eee;
      text-align: left;
    }
    th {
      background: #f9fafb;
    }
    .pill {
      display: inline-block;
      padding: 2px 6px;
      border-radius: 999px;
      font-size: 11px;
    }
    .pill.cleared { background:#dcfce7; color:#166534; }
    .pill.pending { background:#fef9c3; color:#92400e; }
    .pill.blur, .pill.remove { background:#fee2e2; color:#b91c1c; }
    .pill.legal_review { background:#e0f2fe; color:#1d4ed8; }
    .tabs {
      display: flex;
      gap: 8px;
      margin-bottom: 8px;
    }
    .tab {
      padding: 4px 10px;
      border-radius: 999px;
      border: 1px solid transparent;
      cursor: pointer;
      font-size: 14px;
    }
    .tab.active {
      background: #111827;
      color: #fff;
      border-color: #111827;
    }
    a.link {
      color: #2563eb;
      text-decoration: none;
    }
    a.link:hover {
      text-decoration: underline;
    }
    .muted {
      color:#6b7280;
      font-size: 13px;
    }
    textarea {
      resize: vertical;
      min-height: 50px;
    }
  </style>
</head>
<body>
<div class="app">
  <header>
    <div>
      <h2>Clearance Prototype</h2>
      <div class="muted">Local-only demo (data saved in your browser)</div>
    </div>
    <button id="resetDataBtn" class="danger">Reset demo data</button>
  </header>

  <div class="card" id="productionCard"></div>

  <div class="card">
    <div class="tabs">
      <button class="tab active" data-tab="episodes">Episodes</button>
      <button class="tab" data-tab="people">People</button>
      <button class="tab" data-tab="locations">Locations</button>
      <button class="tab" data-tab="releases">Releases</button>
    </div>
    <div id="tabContent"></div>
  </div>
</div>

<script>
  // --- Data layer (in localStorage) ---------------------------------
  const STORAGE_KEY = 'clearance_demo_v1';

  const defaultData = () => ({
    nextIds: { production: 2, episode: 2, person: 2, location: 2, release: 2, item: 2, clearance: 2 },
    productions: [
      { id: 1, name: 'Demo Production', description: 'Sample factual series' }
    ],
    episodes: [
      { id: 1, production_id: 1, code: 'S01E01', title: 'Pilot', status: 'logging' }
    ],
    people: [
      { id: 1, production_id: 1, name: 'John Doe', contact: 'john@example.com' }
    ],
    locations: [
      { id: 1, production_id: 1, name: 'Studio A', address: '123 Main St' }
    ],
    releases: [
      { id: 1, production_id: 1, type: 'appearance', subject_type: 'person', subject_id: 1, file_url: 'https://example.com/release1.pdf', status: 'signed', notes:'' }
    ],
    items: [
      { id: 1, episode_id: 1, kind: 'person', reference_id: 1, time_in: '00:01:00', time_out:'00:01:10', description:'John walks in' }
    ],
    clearances: [
      { id: 1, item_id: 1, release_id: 1, status: 'cleared', comments: 'All good' }
    ]
  });

  let state = loadState();

  function loadState() {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return defaultData();
    try {
      return JSON.parse(raw);
    } catch {
      return defaultData();
    }
  }
  function saveState() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  }

  function resetDemoData() {
    state = defaultData();
    saveState();
    render();
  }

  // --- Helper functions ---------------------------------------------

  function getProduction() {
    // single production for now
    return state.productions[0];
  }

  function getEpisodesForProduction(prodId) {
    return state.episodes.filter(e => e.production_id === prodId);
  }

  function getPeopleForProduction(prodId) {
    return state.people.filter(p => p.production_id === prodId);
  }

  function getLocationsForProduction(prodId) {
    return state.locations.filter(l => l.production_id === prodId);
  }

  function getReleasesForProduction(prodId) {
    return state.releases.filter(r => r.production_id === prodId);
  }

  function getItemsForEpisode(epId) {
    return state.items.filter(i => i.episode_id === epId);
  }

  function getLatestClearanceForItem(itemId) {
    const history = state.clearances.filter(c => c.item_id === itemId);
    if (!history.length) return null;
    return history[history.length - 1];
  }

  function nextId(key) {
    const id = state.nextIds[key]++;
    return id;
  }

  function statusClass(status) {
    if (!status) return '';
    return status.replace(/\s+/g, '_');
  }

  // --- Rendering ----------------------------------------------------
  let currentTab = 'episodes';
  let currentEpisodeId = 1;

  const productionCard = document.getElementById('productionCard');
  const tabContent = document.getElementById('tabContent');

  function renderProductionHeader() {
    const prod = getProduction();
    productionCard.innerHTML = `
      <div>
        <strong>Production:</strong> ${prod.name}
        <div class="muted">${prod.description || ''}</div>
      </div>
    `;
  }

  function renderTabs() {
    document.querySelectorAll('.tab').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.tab === currentTab);
    });

    if (currentTab === 'episodes') renderEpisodesTab();
    if (currentTab === 'people') renderPeopleTab();
    if (currentTab === 'locations') renderLocationsTab();
    if (currentTab === 'releases') renderReleasesTab();
  }

  // Episodes tab
  function renderEpisodesTab() {
    const prod = getProduction();
    const episodes = getEpisodesForProduction(prod.id);

    let html = `
      <h3>Episodes</h3>
      <div class="row">
        <input id="newEpCode" placeholder="Episode code (e.g. S01E02)" />
        <input id="newEpTitle" placeholder="Title (optional)" style="min-width:200px" />
        <button class="primary" id="addEpisodeBtn">Add episode</button>
      </div>
      <table>
        <thead>
          <tr>
            <th>Code</th>
            <th>Title</th>
            <th>Status</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
    `;

    if (!episodes.length) {
      html += `<tr><td colspan="4" class="muted">No episodes yet.</td></tr>`;
    } else {
      for (const ep of episodes) {
        const isSelected = ep.id === currentEpisodeId;
        html += `
          <tr>
            <td>${ep.code}</td>
            <td>${ep.title || ''}</td>
            <td>${ep.status || ''}</td>
            <td>
              <button data-ep="${ep.id}" class="primary small open-episode-btn">
                ${isSelected ? 'Viewing' : 'Open'}
              </button>
            </td>
          </tr>
        `;
      }
    }

    html += `
        </tbody>
      </table>
      <div id="episodeDetailContainer"></div>
    `;

    tabContent.innerHTML = html;

    document.getElementById('addEpisodeBtn').onclick = () => {
      const code = document.getElementById('newEpCode').value.trim();
      const title = document.getElementById('newEpTitle').value.trim();
      if (!code) {
        alert('Episode code is required');
        return;
      }
      const ep = {
        id: nextId('episode'),
        production_id: prod.id,
        code,
        title,
        status: 'logging'
      };
      state.episodes.push(ep);
      saveState();
      currentEpisodeId = ep.id;
      render();
    };

    document.querySelectorAll('.open-episode-btn').forEach(btn => {
      btn.onclick = () => {
        currentEpisodeId = Number(btn.dataset.ep);
        render();
      };
    });

    renderEpisodeDetail();
  }

  function renderEpisodeDetail() {
    const container = document.getElementById('episodeDetailContainer');
    const episode = state.episodes.find(e => e.id === currentEpisodeId);
    if (!episode) {
      container.innerHTML = `<div class="muted">Select an episode to view on-screen items.</div>`;
      return;
    }

    const items = getItemsForEpisode(episode.id);
    const releases = getReleasesForProduction(episode.production_id);

    let html = `
      <div class="card" style="margin-top:16px;">
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <div>
            <strong>Episode ${episode.code}</strong>
            ${episode.title ? ' – ' + episode.title : ''}
          </div>
          <button id="exportCsvBtn">Export clearance CSV</button>
        </div>

        <h4 style="margin-top:12px;">On-screen items</h4>
        <form id="addItemForm">
          <div class="row">
            <select name="kind">
              <option value="person">Person</option>
              <option value="location">Location</option>
              <option value="brand">Brand</option>
            </select>
            <input name="time_in" placeholder="In (HH:MM:SS)" />
            <input name="time_out" placeholder="Out (HH:MM:SS)" />
            <input name="description" placeholder="Description" style="flex:1; min-width:200px" />
            <button type="submit" class="primary">Add</button>
          </div>
        </form>

        <table style="margin-top:8px;">
          <thead>
            <tr>
              <th>In</th>
              <th>Out</th>
              <th>Kind</th>
              <th>Description</th>
              <th>Clearance</th>
              <th>Release</th>
            </tr>
          </thead>
          <tbody>
    `;

    if (!items.length) {
      html += `<tr><td colspan="6" class="muted">No on-screen items logged yet.</td></tr>`;
    } else {
      for (const item of items) {
        const latest = getLatestClearanceForItem(item.id);
        const status = latest ? latest.status : '';
        const rel = latest && latest.release_id
          ? state.releases.find(r => r.id === latest.release_id)
          : null;

        html += `
          <tr>
            <td>${item.time_in}</td>
            <td>${item.time_out}</td>
            <td>${item.kind}</td>
            <td>${item.description}</td>
            <td>
              <select class="statusSel" data-item="${item.id}">
                <option value="">(none)</option>
                <option value="cleared" ${status === 'cleared' ? 'selected' : ''}>cleared</option>
                <option value="pending" ${status === 'pending' ? 'selected' : ''}>pending</option>
                <option value="blur" ${status === 'blur' ? 'selected' : ''}>blur</option>
                <option value="remove" ${status === 'remove' ? 'selected' : ''}>remove</option>
                <option value="legal_review" ${status === 'legal_review' ? 'selected' : ''}>legal_review</option>
              </select>
              ${status ? `<span class="pill ${statusClass(status)}">${status}</span>` : ''}
            </td>
            <td>
              <select class="releaseSel" data-item="${item.id}">
                <option value="">(no release)</option>
                ${releases.map(r =>
                  `<option value="${r.id}" ${
                    rel && rel.id === r.id ? 'selected' : ''
                  }>${r.type} #${r.id} – ${r.subject_type}</option>`
                ).join('')}
              </select>
            </td>
          </tr>
        `;
      }
    }

    html += `
          </tbody>
        </table>
      </div>
    `;

    container.innerHTML = html;

    // Add item handler
    document.getElementById('addItemForm').onsubmit = (e) => {
      e.preventDefault();
      const form = e.target;
      const kind = form.kind.value;
      const time_in = form.time_in.value.trim();
      const time_out = form.time_out.value.trim();
      const description = form.description.value.trim();
      if (!kind || !time_in) {
        alert('Kind and In time are required');
        return;
      }
      const item = {
        id: nextId('item'),
        episode_id: episode.id,
        kind,
        reference_id: null,
        time_in,
        time_out,
        description
      };
      state.items.push(item);
      saveState();
      render();
    };

    // Export CSV
    document.getElementById('exportCsvBtn').onclick = () => {
      exportEpisodeCsv(episode.id);
    };

    // Clearance handlers
    document.querySelectorAll('.statusSel').forEach(sel => {
      sel.onchange = () => {
        const itemId = Number(sel.dataset.item);
        const status = sel.value || null;
        const relSel = document.querySelector('.releaseSel[data-item="' + itemId + '"]');
        const releaseId = relSel && relSel.value ? Number(relSel.value) : null;
        applyClearance(itemId, status, releaseId);
      };
    });

    document.querySelectorAll('.releaseSel').forEach(sel => {
      sel.onchange = () => {
        const itemId = Number(sel.dataset.item);
        const relVal = sel.value ? Number(sel.value) : null;
        const statusSel = document.querySelector('.statusSel[data-item="' + itemId + '"]');
        const status = statusSel && statusSel.value ? statusSel.value : null;
        if (!status) {
          // Just store linkage without status? For simplicity, require a status:
          if (!confirm('You have no clearance status set. Set one now?')) return;
        }
        applyClearance(itemId, status || 'pending', relVal);
      };
    });
  }

  // People tab
  function renderPeopleTab() {
    const prod = getProduction();
    const people = getPeopleForProduction(prod.id);

    let html = `
      <h3>People</h3>
      <form id="addPersonForm">
        <div class="row">
          <input name="name" placeholder="Name" />
          <input name="contact" placeholder="Contact info" style="min-width:200px" />
          <button type="submit" class="primary">Add</button>
        </div>
      </form>
      <table>
        <thead>
          <tr><th>Name</th><th>Contact</th></tr>
        </thead>
        <tbody>
    `;

    if (!people.length) {
      html += `<tr><td colspan="2" class="muted">No people added yet.</td></tr>`;
    } else {
      for (const p of people) {
        html += `<tr><td>${p.name}</td><td>${p.contact || ''}</td></tr>`;
      }
    }

    html += `</tbody></table>`;

    tabContent.innerHTML = html;

    document.getElementById('addPersonForm').onsubmit = e => {
      e.preventDefault();
      const form = e.target;
      const name = form.name.value.trim();
      const contact = form.contact.value.trim();
      if (!name) {
        alert('Name is required');
        return;
      }
      state.people.push({
        id: nextId('person'),
        production_id: prod.id,
        name,
        contact
      });
      saveState();
      render();
    };
  }

  // Locations tab
  function renderLocationsTab() {
    const prod = getProduction();
    const locations = getLocationsForProduction(prod.id);

    let html = `
      <h3>Locations</h3>
      <form id="addLocationForm">
        <div class="row">
          <input name="name" placeholder="Name" />
          <input name="address" placeholder="Address" style="min-width:200px" />
          <button type="submit" class="primary">Add</button>
        </div>
      </form>
      <table>
        <thead>
          <tr><th>Name</th><th>Address</th></tr>
        </thead>
        <tbody>
    `;

    if (!locations.length) {
      html += `<tr><td colspan="2" class="muted">No locations added yet.</td></tr>`;
    } else {
      for (const l of locations) {
        html += `<tr><td>${l.name}</td><td>${l.address || ''}</td></tr>`;
      }
    }

    html += `</tbody></table>`;

    tabContent.innerHTML = html;

    document.getElementById('addLocationForm').onsubmit = e => {
      e.preventDefault();
      const form = e.target;
      const name = form.name.value.trim();
      const address = form.address.value.trim();
      if (!name) {
        alert('Name is required');
        return;
      }
      state.locations.push({
        id: nextId('location'),
        production_id: prod.id,
        name,
        address
      });
      saveState();
      render();
    };
  }

  // Releases tab
  function renderReleasesTab() {
    const prod = getProduction();
    const releases = getReleasesForProduction(prod.id);

    let html = `
      <h3>Releases</h3>
      <form id="addReleaseForm">
        <div class="row">
          <select name="type">
            <option value="appearance">Appearance</option>
            <option value="location">Location</option>
            <option value="materials">Materials</option>
          </select>
          <select name="subject_type">
            <option value="person">Person</option>
            <option value="location">Location</option>
            <option value="other">Other</option>
          </select>
          <input name="subject_id" placeholder="Subject ID (optional)" style="width:120px" />
          <input name="file_url" placeholder="File URL" style="flex:1; min-width:220px" />
        </div>
        <div class="row">
          <textarea name="notes" placeholder="Notes (optional)" style="flex:1;"></textarea>
          <button type="submit" class="primary">Add release</button>
        </div>
      </form>
      <table style="margin-top:8px;">
        <thead>
          <tr>
            <th>ID</th>
            <th>Type</th>
            <th>Subject</th>
            <th>File</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
    `;

    if (!releases.length) {
      html += `<tr><td colspan="5" class="muted">No releases yet.</td></tr>`;
    } else {
      for (const r of releases) {
        html += `
          <tr>
            <td>${r.id}</td>
            <td>${r.type}</td>
            <td>${r.subject_type}${r.subject_id ? ' #' + r.subject_id : ''}</td>
            <td>${r.file_url ? `<a class="link" href="${r.file_url}" target="_blank">open</a>` : ''}</td>
            <td>${r.status}</td>
          </tr>
        `;
      }
    }

    html += `</tbody></table>`;

    tabContent.innerHTML = html;

    document.getElementById('addReleaseForm').onsubmit = e => {
      e.preventDefault();
      const form = e.target;
      const type = form.type.value;
      const subject_type = form.subject_type.value;
      const subject_id_raw = form.subject_id.value.trim();
      const file_url = form.file_url.value.trim();
      const notes = form.notes.value.trim();
      if (!type || !subject_type || !file_url) {
        alert('Type, subject type and file URL are required');
        return;
      }
      const release = {
        id: nextId('release'),
        production_id: prod.id,
        type,
        subject_type,
        subject_id: subject_id_raw ? Number(subject_id_raw) : null,
        file_url,
        status: 'signed',
        notes
      };
      state.releases.push(release);
      saveState();
      render();
    };
  }

  // --- Clearance + CSV logic ---------------------------------------

  function applyClearance(itemId, status, releaseId) {
    if (!status) {
      // If clearing status, just log pending with comment
      status = 'pending';
    }
    const entry = {
      id: nextId('clearance'),
      item_id: itemId,
      release_id: releaseId || null,
      status,
      comments: ''
    };
    state.clearances.push(entry);
    saveState();
    render();
  }

  function exportEpisodeCsv(epId) {
    const ep = state.episodes.find(e => e.id === epId);
    if (!ep) {
      alert('Episode not found');
      return;
    }
    const items = getItemsForEpisode(epId);
    const rows = [];
    const header = [
      'episode_code',
      'time_in',
      'time_out',
      'kind',
      'description',
      'clearance_status',
      'release_type',
      'subject_type',
      'subject_id',
      'clearance_comments'
    ];

    for (const item of items) {
      const latest = getLatestClearanceForItem(item.id);
      const rel = latest && latest.release_id
        ? state.releases.find(r => r.id === latest.release_id)
        : null;
      rows.push({
        episode_code: ep.code,
        time_in: item.time_in,
        time_out: item.time_out,
        kind: item.kind,
        description: item.description,
        clearance_status: latest ? latest.status : '',
        release_type: rel ? rel.type : '',
        subject_type: rel ? rel.subject_type : '',
        subject_id: rel && rel.subject_id ? rel.subject_id : '',
        clearance_comments: latest ? latest.comments : ''
      });
    }

    // Build CSV
    const lines = [];
    lines.push(header.join(','));
    for (const row of rows) {
      const vals = header.map(key => {
        const val = String(row[key] ?? '');
        const escaped = val.replace(/"/g, '""');
        return `"${escaped}"`;
      });
      lines.push(vals.join(','));
    }
    const csv = lines.join('\n');
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `clearance_${ep.code}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  // --- Main render + event wiring ----------------------------------

  function render() {
    renderProductionHeader();
    renderTabs();
  }

  document.querySelectorAll('.tab').forEach(btn => {
    btn.addEventListener('click', () => {
      currentTab = btn.dataset.tab;
      renderTabs();
    });
  });

  document.getElementById('resetDataBtn').onclick = () => {
    if (confirm('Reset demo data? This will clear any changes you made.')) {
      resetDemoData();
    }
  };

  render();
</script>
</body>
</html>
