<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>my location</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<style>
  :root { color-scheme: dark; }
  * { box-sizing: border-box; }
  body {
    margin: 0; background: #111; color: #eee;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }
  .top { padding: 16px; text-align: center; background: #1b1b1b; border-bottom: 1px solid #333; }
  button {
    font-size: 17px; padding: 14px 24px; border: none; border-radius: 12px;
    background: #4f8cff; color: #fff; cursor: pointer; width: 100%; max-width: 320px;
  }
  button:disabled { opacity: 0.5; }
  .muted { color: #999; font-size: 13px; margin-top: 10px; }
  .error { color: #ff8080; font-size: 13px; margin-top: 10px; }
  #map { height: 45vh; width: 100%; }
  table { width: 100%; border-collapse: collapse; font-size: 13px; }
  th, td { padding: 6px 10px; border-bottom: 1px solid #2a2a2a; text-align: left; }
  tr:hover { background: #1f1f1f; cursor: pointer; }
  .wrap { max-height: 35vh; overflow-y: auto; }
  .row { display: flex; justify-content: center; gap: 8px; padding: 8px; }
  .row button.secondary { background: #333; max-width: 160px; font-size: 14px; padding: 8px 12px; }
</style>
</head>
<body>

<div class="top">
  <button id="btn">share my location</button>
  <p class="muted" id="helper">saved only on this device</p>
</div>
<div class="row">
  <button class="secondary" id="clearBtn">clear history</button>
</div>
<div id="map"></div>
<div class="wrap">
  <table id="pointsTable">
    <thead><tr><th>time</th><th>lat</th><th>lng</th><th>accuracy</th></tr></thead>
    <tbody></tbody>
  </table>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
  const STORAGE_KEY = "myLocationHistory";
  const btn = document.getElementById("btn");
  const clearBtn = document.getElementById("clearBtn");
  const helper = document.getElementById("helper");
  const tbody = document.querySelector("#pointsTable tbody");

  const map = L.map("map").setView([20, 0], 2);
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "&copy; OpenStreetMap contributors", maxZoom: 19,
  }).addTo(map);
  let polyline = L.polyline([], { color: "#4f8cff" }).addTo(map);
  let latestMarker = null;

  function loadPoints() {
    try {
      return JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    } catch {
      return [];
    }
  }

  function savePoints(points) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(points));
  }

  function fmtMeters(acc) {
    return typeof acc === "number" ? `${Math.round(acc)}m` : "unknown";
  }

  function redraw() {
    const points = loadPoints();
    polyline.setLatLngs(points.map(p => [p.lat, p.lng]));

    if (latestMarker) map.removeLayer(latestMarker);
    const last = points[points.length - 1];
    if (last) {
      latestMarker = L.marker([last.lat, last.lng]).addTo(map);
      map.setView([last.lat, last.lng], Math.max(map.getZoom(), 14));
    }

    tbody.innerHTML = "";
    for (const p of [...points].reverse()) {
      const tr = document.createElement("tr");
      const time = new Date(p.ts).toLocaleString();
      tr.innerHTML = `<td>${time}</td><td>${p.lat.toFixed(5)}</td><td>${p.lng.toFixed(5)}</td><td>${fmtMeters(p.acc)}</td>`;
      tr.addEventListener("click", () => map.setView([p.lat, p.lng], 16));
      tbody.appendChild(tr);
    }
  }

  function showError(msg) {
    helper.textContent = msg;
    helper.className = "error";
    btn.disabled = false;
  }

  function showSaved(pos) {
    helper.textContent = `saved — accuracy ${fmtMeters(pos.coords.accuracy)}`;
    helper.className = "muted";
    btn.disabled = false;
  }

  btn.addEventListener("click", () => {
    if (location.protocol !== "https:" && location.hostname !== "localhost") {
      showError("this page needs https to access location"); return;
    }
    if (!("geolocation" in navigator)) {
      showError("this browser doesn't support location"); return;
    }
    btn.disabled = true;
    helper.textContent = "getting your location…";
    helper.className = "muted";

    // must be called directly inside the click handler, or iOS Safari
    // silently suppresses the permission prompt
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        const points = loadPoints();
        points.push({
          lat: pos.coords.latitude,
          lng: pos.coords.longitude,
          acc: pos.coords.accuracy,
          ts: Date.now(),
        });
        savePoints(points);
        redraw();
        showSaved(pos);
      },
      (err) => {
        if (err.code === err.PERMISSION_DENIED) showError("permission denied — allow location and try again");
        else if (err.code === err.TIMEOUT) showError("timed out — try again, maybe move outside");
        else showError("couldn't get location (" + err.message + ")");
      },
      { enableHighAccuracy: true, timeout: 20000, maximumAge: 0 }
    );
  });

  clearBtn.addEventListener("click", () => {
    if (confirm("clear all saved location history?")) {
      localStorage.removeItem(STORAGE_KEY);
      redraw();
    }
  });

  redraw();
</script>
</body>
</html>
