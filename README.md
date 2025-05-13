# trainermap

<!-- PTP Interactive Trainer Finder (Improved UI) -->
<div style="text-align:center; padding: 24px 12px;">
  <h1 style="font-size: 2.2em; font-weight: bold; color: #111;">
    🟡 Find the Best PTP Trainer Near You
  </h1>
  <p style="font-size: 1.1em; color: #444; max-width: 600px; margin: 12px auto;">
    Search by position, availability, or ZIP — and we’ll show you the closest match.
  </p>
</div>

<!-- Filters -->
<div style="text-align:center; margin-bottom: 20px;">
  <label style="font-weight:bold;">Position:</label>
  <select id="positionSelect" style="padding: 10px; font-size: 1em; margin: 8px;">
    <option value="">Any</option>
    <option value="Striker">Striker</option>
    <option value="Midfielder">Midfielder</option>
    <option value="Defender">Defender</option>
  </select>

  <label style="font-weight:bold;">Time:</label>
  <select id="scheduleSelect" style="padding: 10px; font-size: 1em; margin: 8px;">
    <option value="">Any</option>
    <option value="Mornings">Mornings</option>
    <option value="Afternoons">Afternoons</option>
  </select>

  <input id="zipInput" placeholder="ZIP (optional)" style="padding: 10px; width: 200px; margin-top: 12px;" />
  <br/>
  <button onclick="filterTrainers()" style="margin-top: 12px; padding: 12px 24px; font-size: 1em; font-weight: bold; background:#FFD700; border:none; border-radius:6px; cursor:pointer;">
    🔍 Find My Trainer
  </button>
</div>

<!-- Map + Results -->
<div id="map" style="width:100%; height:400px; border-radius: 12px; margin-bottom: 20px;"></div>
<div id="results" style="text-align:center;"></div>

<script>
const today = ["Sun","Mon","Tue","Wed","Thu","Fri","Sat"][new Date().getDay()];
const trainers = [{"field": "Radnor Memorial Park", "address": "61 Matsonford Rd", "city": "Wayne", "zip": "19087", "lat": 40.045, "lng": -75.371, "trainer": "Sebastian Cutler", "link": "https://ptptrainings.as.me/sebastiancutlerptp", "availability": "Mon,Tue,Wed,Thu,Fri", "position": "Midfielder", "schedule": "Mornings", "trainingType": "Ball Control", "description": "Sebastian specializes in ball control and youth mentorship."}, {"field": "Dittmar Park", "address": "831 S Valley Forge Rd", "city": "Devon", "zip": "19333", "lat": 40.034, "lng": -75.413, "trainer": "Ken Roby", "link": "https://ptptrainings.as.me/kenrobyptp", "availability": "Mon,Tue,Wed,Thu,Fri", "position": "Defender", "schedule": "Afternoons", "trainingType": "Speed", "description": "Ken is known for speed and technical training."}, {"field": "Wilson Farm Park", "address": "500 Lee Rd", "city": "Chesterbrook", "zip": "19087", "lat": 40.065, "lng": -75.452, "trainer": "Lorenzo Avalos", "link": "https://ptptrainings.as.me/lorenzoavalosptp", "availability": "Mon,Tue,Wed,Thu,Fri", "position": "Striker", "schedule": "Afternoons", "trainingType": "Finishing", "description": "Lorenzo focuses on finishing and first-touch attacking drills."}];

let map;

function initMap() {
  map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: 40.04, lng: -75.39 },
    zoom: 11
  });
  trainers.forEach(loc => {
    new google.maps.Marker({
      position: { lat: loc.lat, lng: loc.lng },
      map,
      icon: "https://maps.google.com/mapfiles/kml/shapes/soccer.png",
      title: loc.field
    });
  });
}

function filterTrainers() {
  const pos = document.getElementById("positionSelect").value;
  const sched = document.getElementById("scheduleSelect").value;
  const zip = document.getElementById("zipInput").value.trim();
  const box = document.getElementById("results");

  const filtered = trainers.filter(t =>
    (!pos || t.position === pos) &&
    (!sched || t.schedule === sched) &&
    t.availability.includes(today)
  );

  if (!zip) {
    box.innerHTML = filtered.map(t => showResult(t)).join('') || "<p>No trainers match today's filters.</p>";
    return;
  }

  const geo = new google.maps.Geocoder();
  geo.geocode({ address: zip }, (res, stat) => {
    if (stat !== "OK") {
      box.innerHTML = "<p>ZIP not found.</p>";
      return;
    }
    const userLoc = res[0].geometry.location;
    let closest = null, dist = Infinity;
    filtered.forEach(t => {
      const d = haversine(userLoc.lat(), userLoc.lng(), t.lat, t.lng);
      if (d < dist) { dist = d; closest = { ...t, dist }; }
    });
    box.scrollIntoView({ behavior: 'smooth' });
    box.innerHTML = closest ? showResult(closest, closest.dist) : "<p>No match found near you.</p>";
  });
}

function showResult(t, dist = null) {
  return `<div style="padding: 20px; margin: 12px auto; max-width: 480px; background:${dist ? '#eaffea' : '#fffbe6'}; border-radius: 10px;">
    <h2 style="color:#111;">${t.trainer}</h2>
    <p><strong>${t.field}</strong><br>${t.address}, ${t.city}, PA</p>
    <p>🕐 <strong>${t.schedule}</strong> | 📌 <strong>${t.position}</strong></p>
    <p>📅 Availability: ${t.availability}</p>
    <p>${t.description}</p>
    ${dist ? '<p><strong>Distance:</strong> ' + dist.toFixed(1) + ' mi</p>' : ''}
    <a href="${t.link}" target="_blank" style="display:inline-block;margin-top:10px;background:#FFD700;padding:10px 18px;border-radius:6px;color:#000;text-decoration:none;font-weight:bold;">
      ✅ Book with ${t.trainer}
    </a>
  </div>`;
}

function haversine(lat1, lon1, lat2, lon2) {
  const R = 3958.8;
  const toRad = x => x * Math.PI / 180;
  const dLat = toRad(lat2 - lat1), dLon = toRad(lon2 - lon1);
  const a = Math.sin(dLat/2)**2 + Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon/2)**2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}
</script>
<script async defer src="https://maps.googleapis.com/maps/api/js?key=AIzaSyASLf8mjb8iEbVo99DnaaPzXGlu5jhXrZE&callback=initMap"></script>
