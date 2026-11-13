<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>share location</title>
<style>
  :root { color-scheme: dark; }
  * { box-sizing: border-box; }
  body {
    margin: 0; min-height: 100vh; display: flex; align-items: center; justify-content: center;
    background: #111; color: #eee; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    padding: 16px;
  }
  .card { background: #1b1b1b; border-radius: 16px; padding: 32px 24px; max-width: 360px; width: 100%; text-align: center; }
  button { font-size: 18px; padding: 16px 24px; width: 100%; border: none; border-radius: 12px; background: #4f8cff; color: #fff; cursor: pointer; }
  button:disabled { opacity: 0.5; }
  .muted { color: #999; font-size: 14px; margin-top: 12px; }
  .error { color: #ff8080; font-size: 14px; margin-top: 12px; }
  .ok { font-size: 20px; margin-bottom: 8px; }
  a { color: #4f8cff; }
</style>
</head>
<body>
  <div class="card" id="card">
    <button id="btn">share location</button>
    <p class="muted" id="helper">one tap, nothing saved on this device</p>
  </div>

<script src="https://www.gstatic.com/firebasejs/10.13.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.13.0/firebase-database-compat.js"></script>
<script>
  const firebaseConfig = {
    apiKey: "AIzaSyAl_uYFgzSIqfg1p2CLMgP0lGsmkR5fJvI",
    authDomain: "my-location-app-204c4.firebaseapp.com",
    databaseURL: "https://my-location-app-204c4-default-rtdb.firebaseio.com",
    projectId: "my-location-app-204c4",
    storageBucket: "my-location-app-204c4.firebasestorage.app",
    messagingSenderId: "359565273563",
    appId: "1:359565273563:web:716c70bffce2af3c2d1ffe",
  };
  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  const params = new URLSearchParams(location.search);
  const token = params.get("t");

  const btn = document.getElementById("btn");
  const helper = document.getElementById("helper");
  const card = document.getElementById("card");

  let watchId = null;
  let lastPingAt = 0;
  const MIN_PING_INTERVAL_MS = 45000;

  function showError(msg) {
    helper.textContent = msg;
    helper.className = "error";
    btn.disabled = false;
  }

  function fmtMeters(acc) {
    return typeof acc === "number" ? `${Math.round(acc)}m` : "unknown";
  }

  async function sendPing(pos) {
    const { latitude, longitude, accuracy } = pos.coords;
    if (accuracy > 5000) return false;
    try {
      await db.ref(`locations/${token}`).push({
        lat: latitude, lng: longitude, acc: accuracy, ts: Date.now(),
      });
      return true;
    } catch (err) {
      helper.textContent = "sent, but couldn't confirm save (offline?)";
      helper.className = "muted";
      return false;
    }
  }

  function showSuccess(pos) {
    const geoUri = `geo:${pos.coords.latitude},${pos.coords.longitude}`;
    card.innerHTML = `
      <div class="ok">location shared ✅</div>
      <p class="muted">accuracy: ${fmtMeters(pos.coords.accuracy)}</p>
      <p class="muted"><a href="${geoUri}">open in maps app</a></p>
    `;
  }

  async function handleFirstFix(pos) {
    await sendPing(pos);
    lastPingAt = Date.now();
    showSuccess(pos);
    startWatching();
  }

  function startWatching() {
    if (watchId !== null || !("geolocation" in navigator)) return;
    watchId = navigator.geolocation.watchPosition(
      (pos) => {
        const now = Date.now();
        if (now - lastPingAt < MIN_PING_INTERVAL_MS) return;
        lastPingAt = now;
        sendPing(pos);
      },
      () => {},
      { enableHighAccuracy: true, maximumAge: 0 }
    );
  }

  document.addEventListener("visibilitychange", () => {
    if (document.visibilityState === "hidden" && watchId !== null) {
      navigator.geolocation.clearWatch(watchId);
      watchId = null;
    }
  });

  btn.addEventListener("click", () => {
    if (!token) { showError("this link is missing its token"); return; }
    if (location.protocol !== "https:" && location.hostname !== "localhost") {
      showError("this page needs https to access location"); return;
    }
    if (!("geolocation" in navigator)) {
      showError("this browser doesn't support location sharing"); return;
    }
    btn.disabled = true;
    helper.textContent = "getting your location…";
    helper.className = "muted";

    navigator.geolocation.getCurrentPosition(
      handleFirstFix,
      (err) => {
        if (err.code === err.PERMISSION_DENIED) showError("permission denied — allow location and try again");
        else if (err.code === err.TIMEOUT) showError("timed out — try again, maybe move outside");
        else showError("couldn't get location (" + err.message + ")");
      },
      { enableHighAccuracy: true, timeout: 20000, maximumAge: 0 }
    );
  });
</script>
</body>
</html>
