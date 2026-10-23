<!DOCTYPE html>
<html lang="ka">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>MotoTaxi Ozurgeti</title>
  
  <!-- Leaflet Map CSS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    }

    body, html {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background: #121212;
      color: #fff;
    }

    /* ზედა რეჟიმების გადამრთველი (კლიენტი / მძღოლი) */
    .top-nav {
      position: absolute;
      top: 12px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      background: rgba(30, 30, 30, 0.9);
      backdrop-filter: blur(10px);
      padding: 4px;
      border-radius: 20px;
      z-index: 2000;
      box-shadow: 0 4px 15px rgba(0,0,0,0.5);
    }

    .nav-btn {
      padding: 8px 18px;
      border: none;
      border-radius: 16px;
      background: transparent;
      color: #aaa;
      font-weight: 600;
      font-size: 13px;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    .nav-btn.active {
      background: #a3e635;
      color: #000;
    }

    /* --- კლიენტის ხედი --- */
    #client-view {
      width: 100%;
      height: 100vh;
      position: relative;
    }

    #map {
      width: 100%;
      height: 100vh;
      z-index: 1;
    }

    .center-pin {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -100%);
      z-index: 1000;
      pointer-events: none;
    }

    .pin-icon {
      width: 36px;
      height: 36px;
      background: #a3e635;
      border-radius: 50% 50% 50% 0;
      transform: rotate(-45deg);
      display: flex;
      align-items: center;
      justify-content: center;
      box-shadow: 0 4px 10px rgba(0,0,0,0.4);
    }

    .pin-icon::after {
      content: '';
      width: 14px;
      height: 14px;
      background: #000;
      border-radius: 50%;
    }

    .bottom-sheet {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background: #1e1e1e;
      border-top-left-radius: 24px;
      border-top-right-radius: 24px;
      padding: 20px;
      z-index: 1000;
      box-shadow: 0 -5px 25px rgba(0,0,0,0.6);
    }

    .sheet-handle {
      width: 40px;
      height: 4px;
      background: #444;
      border-radius: 2px;
      margin: 0 auto 15px auto;
    }

    .input-group {
      background: #2a2a2a;
      border-radius: 14px;
      padding: 12px;
      margin-bottom: 10px;
      display: flex;
      align-items: center;
      position: relative;
    }

    .dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      margin-right: 12px;
    }
    .dot-pickup { background: #a3e635; }
    .dot-destination { background: #ef4444; }

    .input-group input {
      background: transparent;
      border: none;
      color: #fff;
      font-size: 15px;
      width: 100%;
      outline: none;
    }

    .suggestions {
      position: absolute;
      top: 100%;
      left: 0;
      right: 0;
      background: #2a2a2a;
      border-radius: 12px;
      max-height: 160px;
      overflow-y: auto;
      z-index: 1005;
      margin-top: 5px;
      box-shadow: 0 8px 16px rgba(0,0,0,0.5);
    }

    .suggestion-item {
      padding: 12px 15px;
      border-bottom: 1px solid #333;
      font-size: 14px;
      cursor: pointer;
    }

    .btn-submit {
      width: 100%;
      background: #a3e635;
      color: #000;
      border: none;
      padding: 15px;
      border-radius: 14px;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
      margin-top: 6px;
    }

    /* --- მძღოლის ხედი --- */
    #driver-view {
      display: none;
      width: 100%;
      height: 100vh;
      overflow-y: auto;
      padding: 70px 16px 20px 16px;
      background: #121212;
    }

    .driver-title {
      font-size: 20px;
      font-weight: bold;
      color: #fff;
      margin-bottom: 6px;
    }

    .driver-sub {
      font-size: 13px;
      color: #888;
      margin-bottom: 20px;
    }

    .order-card {
      background: #1e1e1e;
      border-radius: 16px;
      padding: 16px;
      margin-bottom: 12px;
      border: 1px solid #2a2a2a;
    }

    .order-header {
      display: flex;
      justify-content: space-between;
      margin-bottom: 10px;
    }

    .client-name {
      font-weight: bold;
      font-size: 16px;
    }

    .status-badge {
      background: #2a3814;
      color: #a3e635;
      padding: 4px 10px;
      border-radius: 10px;
      font-size: 12px;
      font-weight: 600;
    }

    .order-row {
      font-size: 14px;
      color: #ccc;
      margin-bottom: 6px;
    }

    .btn-accept {
      width: 100%;
      background: #a3e635;
      color: #000;
      border: none;
      padding: 12px;
      border-radius: 12px;
      font-weight: bold;
      font-size: 14px;
      margin-top: 10px;
      cursor: pointer;
    }
  </style>
</head>
<body>

  <!-- ზედა მენიუ -->
  <div class="top-nav">
    <button class="nav-btn active" id="btn-client" onclick="switchView('client')">👤 კლიენტი</button>
    <button class="nav-btn" id="btn-driver" onclick="switchView('driver')">🛵 მძღოლი</button>
  </div>

  <!-- 1. კლიენტის სექცია (რუკა) -->
  <div id="client-view">
    <div id="map"></div>
    <div class="center-pin"><div class="pin-icon"></div></div>

    <div class="bottom-sheet">
      <div class="sheet-handle"></div>

      <div class="input-group">
        <div class="dot dot-pickup"></div>
        <input type="text" id="pickup" placeholder="საიდან მოგაკითხოთ?" oninput="searchAddress(this.value, 'pickup-results', 'pickup')">
        <div id="pickup-results" class="suggestions"></div>
      </div>

      <div class="input-group">
        <div class="dot dot-destination"></div>
        <input type="text" id="destination" placeholder="სად მიდიხართ?" oninput="searchAddress(this.value, 'dest-results', 'destination')">
        <div id="dest-results" class="suggestions"></div>
      </div>

      <button class="btn-submit" onclick="confirmOrder()">მზადაა</button>
    </div>
  </div>

  <!-- 2. მძღოლის სექცია -->
  <div id="driver-view">
    <div class="driver-title">მძღოლის გვერდი</div>
    <div class="driver-sub">აქ გამოჩნდება აქტიური შეკვეთები.</div>

    <div id="orders-list">
      <div class="order-card">
        <div class="order-header">
          <span class="client-name">saba</span>
          <span class="status-badge">ახალი</span>
        </div>
        <div class="order-row">📍 <b>საიდან:</b> კოსტავას ქუჩა 1</div>
        <div class="order-row">🏁 <b>სად:</b> ოზურგეთი</div>
        <div class="order-row">📞 <b>ტელეფონი:</b> 592157300</div>
        <button class="btn-accept" onclick="alert('შეკვეთა მიღებულია!')">შეკვეთის მიღება</button>
      </div>
    </div>
  </div>

  <!-- Leaflet JS -->
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const defaultLat = 41.9269;
    const defaultLng = 42.0003;

    const map = L.map('map', { zoomControl: false }).setView([defaultLat, defaultLng], 16);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(map);

    // GPS მდებარეობა
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(position => {
        const lat = position.coords.latitude;
        const lng = position.coords.longitude;
        map.setView([lat, lng], 17);
        getAddressFromCoords(lat, lng, 'pickup');
      });
    }

    // რუკის მოძრაობაზე მისამართის განახლება
    let moveTimeout;
    map.on('move', () => {
      clearTimeout(moveTimeout);
      moveTimeout = setTimeout(() => {
        const center = map.getCenter();
        getAddressFromCoords(center.lat, center.lng, 'pickup');
      }, 500);
    });

    function getAddressFromCoords(lat, lng, fieldId) {
      fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}&accept-language=ka`)
        .then(res => res.json())
        .then(data => {
          if (data && data.display_name) {
            const parts = data.display_name.split(',');
            document.getElementById(fieldId).value = parts[0] + (parts[1] ? ',' + parts[1] : '');
          }
        }).catch(() => {});
    }

    function searchAddress(query, resultsId, fieldId) {
      const resultsDiv = document.getElementById(resultsId);
      if (query.length < 3) {
        resultsDiv.innerHTML = '';
        return;
      }

      fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query + ' ოზურგეთი')}&accept-language=ka`)
        .then(res => res.json())
        .then(data => {
          resultsDiv.innerHTML = '';
          data.slice(0, 5).forEach(item => {
            const div = document.createElement('div');
            div.className = 'suggestion-item';
            div.innerText = item.display_name;
            div.onclick = () => {
              document.getElementById(fieldId).value = item.display_name.split(',')[0];
              map.setView([item.lat, item.lon], 17);
              resultsDiv.innerHTML = '';
            };
            resultsDiv.appendChild(div);
          });
        }).catch(() => {});
    }

    function switchView(view) {
      if (view === 'client') {
        document.getElementById('client-view').style.display = 'block';
        document.getElementById('driver-view').style.display = 'none';
        document.getElementById('btn-client').classList.add('active');
        document.getElementById('btn-driver').classList.remove('active');
        setTimeout(() => { map.invalidateSize(); }, 200);
      } else {
        document.getElementById('client-view').style.display = 'none';
        document.getElementById('driver-view').style.display = 'block';
        document.getElementById('btn-driver').classList.add('active');
        document.getElementById('btn-client').classList.remove('active');
      }
    }

    function confirmOrder() {
      const from = document.getElementById('pickup').value;
      const to = document.getElementById('destination').value;
      if (!from || !to) {
        alert('გთხოვთ მიუთითოთ ორივე მისამართი!');
      } else {
        alert(`შეკვეთა გაგზავნილია!\nსაიდან: ${from}\nსად: ${to}`);
      }
    }
  </script>
</body>
</html>
