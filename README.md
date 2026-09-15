<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>L'Union Pizza — MYO Reservation</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    *{box-sizing:border-box}
    body{margin:0;font-family:system-ui,-apple-system,Segoe UI,sans-serif;background:#f5f1ea;color:#25221f}
    header{background:#211f1d;color:#fff;padding:18px;text-align:center}
    main{max-width:1050px;margin:auto;padding:20px}
    .hero{text-align:center}
    .card{background:#fff;border:1px solid #ddd5ca;border-radius:16px;padding:18px;margin:15px 0}
    .tables{display:grid;grid-template-columns:1fr 1fr;gap:12px}
    .table{border:1px solid #ddd5ca;border-radius:13px;overflow:hidden;cursor:pointer}
    .table.sel{outline:3px solid #a9352a}
    .photo{height:180px;background:#e9e2d8;display:flex;align-items:center;justify-content:center;text-align:center;color:#756e65}
    .photo img{width:100%;height:100%;object-fit:cover}
    .tablebody{padding:13px}
    .slots{display:grid;grid-template-columns:repeat(4,1fr);gap:7px}
    .slot{padding:10px 4px;border:1px solid #ddd5ca;background:#fff;border-radius:9px;cursor:pointer;font-size:12px;text-align:center}
    .slot.sel{background:#a9352a;color:#fff}
    .slot.lock,.slot.past{background:#eee;color:#aaa;cursor:not-allowed;text-decoration:line-through}
    .booking-board{display:grid;grid-template-columns:120px 1fr 1fr;border:1px solid #ddd5ca;border-radius:10px;overflow:hidden}
    .cell{padding:9px;border-right:1px solid #ddd5ca;border-bottom:1px solid #ddd5ca;font-size:12px;min-height:54px}
    .head{font-weight:800;background:#f8f5ef}
    .fixed{background:#fff4f1;color:#a9352a}
    .pending{background:#f2f2f2;color:#777}
    .menu{display:grid;grid-template-columns:1fr 1fr;gap:8px}
    .mi{padding:11px;border:1px solid #ddd5ca;border-radius:9px;cursor:pointer}
    .mi.sel{border-color:#a9352a;background:#fff5f3}
    .mix{display:grid;grid-template-columns:1fr 1fr;gap:10px}
    .mix select,select,input,textarea{width:100%;padding:10px;border:1px solid #ddd5ca;border-radius:8px;background:white}
    label{display:block;font-weight:700;margin:10px 0 5px}
    .btn{background:#a9352a;color:#fff;border:0;padding:12px;border-radius:9px;font-weight:700;cursor:pointer}
    .primary{width:100%;margin-top:10px}
    .note{background:#f8f5ef;padding:11px;border-radius:9px;font-size:13px}
    .chat{position:fixed;right:18px;bottom:18px;background:#168b52;color:#fff;padding:13px 16px;border-radius:99px;text-decoration:none;font-weight:700}
    @media(max-width:700px){.tables,.menu{grid-template-columns:1fr}.slots{grid-template-columns:repeat(2,1fr)}.booking-board{grid-template-columns:90px 1fr 1fr}}
  </style>
</head>
<body>
<header>
  <b>L'UNION PIZZA</b><br>MAKE YOUR OWN PIZZA
</header>
<main>
  <div class="hero">
    <h1>Reservasi MYO</h1>
    <p>Booking fee Rp10.000 • Sisa product dibayar di kasir</p>
  </div>

  <div class="card">
    <label>Tanggal Reservasi</label>
    <input id="date" type="date">
    <div id="hours" class="note" style="margin-top:10px"></div>
  </div>

  <div class="card">
    <h2>Jadwal MYO</h2>
    <p class="note">🔴 Confirmed menampilkan nama dan menu. Pending tetap mengunci slot tetapi identitas tidak ditampilkan.</p>
    <div id="board" class="booking-board"></div>
  </div>

  <div class="card">
    <h2>1. Pilih Jam</h2>
    <div id="slots" class="slots"></div>
  </div>

  <div class="card">
    <h2>2. Pilih Table</h2>
    <div id="tables" class="tables"></div>
  </div>

  <div class="card">
    <h2>3. Pilih Menu</h2>
    <div id="menus" class="menu"></div>
    <div id="mixBox" style="display:none;margin-top:14px">
      <h3>⭐ Special Kreasi — Mix 2 Flavour</h3>
      <div class="mix">
        <div><label>Flavor 1</label><select id="mix1"></select></div>
        <div><label>Flavor 2</label><select id="mix2"></select></div>
      </div>
      <p id="mixPrice" class="note" style="margin-top:8px"></p>
    </div>
  </div>

  <div class="card">
    <h2>4. Data Customer</h2>
    <label>Nama</label>
    <input id="name" placeholder="Nama pemesan">

    <label>WhatsApp</label>
    <input id="wa" type="tel" placeholder="08xxxxxxxxxx">

    <label>Jumlah Pax</label>
    <input id="pax" type="number" min="1" value="1">

    <label>Catatan</label>
    <textarea id="note" rows="2" placeholder="Catatan tambahan..."></textarea>

    <h3>Booking Fee</h3>
    <div class="note">Transfer <b>Rp10.000</b> ke <b>SeaBank 901702376579</b>. Upload bukti pembayaran di bawah. Sisa product dibayar di kasir.</div>
    <input id="proof" type="file" accept="image/*,.pdf" style="margin-top:10px">

    <button class="btn primary" id="btnBook" onclick="book()">Konfirmasi Reservasi</button>
    <p id="msg" style="margin-top:10px;font-weight:700"></p>
  </div>
</main>

<a class="chat" href="https://wa.me/6285226099883?text=Halo%20Admin%20L%27Union%20Pizza%2C%20saya%20ingin%20bertanya%20mengenai%20reservasi%20MYO." target="_blank">💬 Chat Admin</a>

<script>
const SUPABASE_URL = "https://rjqjnqivbpgvxfhrstwj.supabase.co";
const SUPABASE_KEY = "sb_publishable_5fFP-ciiJmPy_gB-E4KDzg_zKYcyjan";

const db = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

const menus0 = [["Margherita",58000],["Primavera",67000],["Aloha",80000],["Pepperoni",67000],["Proteinata",90000],["Quattro Fromaggi",113000],["Beef Royale",97000],["Meatza",85000],["Le Jardin",75000]];
const tables0 = [{id:"Napoli",desc:"Table MYO Napoli",active:true,img:""},{id:"Romana",desc:"Table MYO Romana",active:true,img:""}];
const settings0 = {"0":["11:00","21:00",15],"1":["11:00","21:00",15],"2":["11:00","21:00",15],"3":["11:00","21:00",15],"4":["11:00","21:00",15],"5":["14:00","21:00",15],"6":["11:00","21:00",15]};

let bookings = [], menus = menus0, tables = tables0, settings = settings0, overrides = [];
let sel = {slot:null, table:null, menu:null, m1:null, m2:null};
const $ = id => document.getElementById(id);

async function syncData(){
  try {
    const { data: b } = await db.from("bookings").select("*");
    if(b) bookings = b;
    const { data: c } = await db.from("app_config").select("*");
    if(c){
      c.forEach(x => {
        if(x.key === "menus") menus = x.value;
        if(x.key === "tables") tables = x.value;
        if(x.key === "settings") settings = x.value;
        if(x.key === "overrides") overrides = x.value;
      });
    }
  } catch(e){
    console.warn("Koneksi Supabase fallback:", e);
    bookings = JSON.parse(localStorage.getItem("MYO_BOOKINGS") || "[]");
  }
  render();
}

function cfg(d){
  let o = overrides.find(x => x.date === d);
  if(o) return o.status === "closed" ? null : [o.start, o.end, o.int];
  return settings[new Date(d + "T00:00:00").getDay()];
}

function times(d){
  let c = cfg(d), a = [];
  if(!c) return a;
  let [h, m] = c[0].split(":").map(Number), [eh, em] = c[1].split(":").map(Number);
  for(let t = h * 60 + m; t < eh * 60 + em; t += +c[2]){
    a.push([t, `${String(Math.floor(t/60)).padStart(2,"0")}.${String(t%60).padStart(2,"0")}`, `${String(Math.floor((t+c[2])/60)).padStart(2,"0")}.${String((t+c[2])%60).padStart(2,"0")}`]);
  }
  return a;
}

function past(d, t){
  let n = new Date(), q = new Date(d + "T00:00:00"), today = new Date(n.getFullYear(), n.getMonth(), n.getDate());
  return q < today || (q.getTime() === today.getTime() && t <= n.getHours() * 60 + n.getMinutes());
}

function locked(d, s, t){
  return bookings.some(x => x.date === d && x.slot === s && (x.table_id === t || x.table === t) && x.status !== "Rejected");
}

function render(){
  let d = $("date").value, c = cfg(d);
  $("hours").textContent = c ? `MYO ${c[0].replace(":",".")}–${c[1].replace(":",".")} • interval ${c[2]} menit` : "MYO ditutup";

  $("slots").innerHTML = times(d).map(x => {
    let full = tables.filter(t => t.active).every(t => locked(d, x[1], t.id));
    let dis = past(d, x[0]) || full;
    return `<button class="slot ${sel.slot===x[1]?"sel ":""}${past(d,x[0])?"past":full?"lock":""}" ${dis?"disabled":""} onclick="sel.slot='${x[1]}';render()">${x[1]}–${x[2]} ${past(d,x[0])?"• Lewat":full?"• Penuh":""}</button>`;
  }).join("");

  $("tables").innerHTML = tables.filter(x => x.active).map(x => `
    <div class="table ${sel.table===x.id?"sel":""}" onclick="sel.table='${x.id}';render()">
      <div class="photo">${x.img ? `<img src="${x.img}">` : `<div><b style="font-size:35px">${x.id.toUpperCase()}</b><br><small>Foto Table</small></div>`}</div>
      <div class="tablebody"><b>${x.id}</b><br><small>${x.desc}</small><br><span style="color:#39734c">● Aktif</span></div>
    </div>
  `).join("");

  $("menus").innerHTML = menus.filter(x => x.active !== false).map((m, i) => `
    <div class="mi ${sel.menu===i?"sel":""}" onclick="sel.menu=${i};$('mixBox').style.display='none';render()">${m[0]}<br><b>Rp${Number(m[1]).toLocaleString('id-ID')}</b></div>
  `).join("") + `<div class="mi ${sel.menu==="mix"?"sel":""}" onclick="sel.menu='mix';$('mixBox').style.display='block';renderMix()">⭐ <b>Special Kreasi</b><br>Mix 2 flavour • 50% + 50%</div>`;

  renderMix();
  renderBoard();
}

function renderMix(){
  if(sel.menu !== "mix") return;
  $("mixBox").style.display = "block";
  let opts = menus.map((m, i) => `<option value="${i}">${m[0]} — Rp${Number(m[1]).toLocaleString("id-ID")}</option>`).join("");
  $("mix1").innerHTML = opts; $("mix2").innerHTML = opts;
  $("mix1").value = sel.m1 ?? 0; $("mix2").value = sel.m2 ?? 1;
  $("mix1").onchange = e => { sel.m1 = +e.target.value; mixCalc(); };
  $("mix2").onchange = e => { sel.m2 = +e.target.value; mixCalc(); };
  mixCalc();
}

function mixCalc(){
  let a = menus[sel.m1 ?? 0], b = menus[sel.m2 ?? 1];
  $("mixPrice").textContent = `${a[0]} Rp${(a[1]/2).toLocaleString("id-ID")} + ${b[0]} Rp${(b[1]/2).toLocaleString("id-ID")} = Total Rp${((a[1]+b[1])/2).toLocaleString("id-ID")}`;
}

function renderBoard(){
  let d = $("date").value, ts = times(d);
  let heads = `<div class="cell head">Jam</div>` + tables.filter(x => x.active).map(x => `<div class="cell head">${x.id}</div>`).join("");
  $("board").innerHTML = heads + ts.map(x => {
    let cells = `<div class="cell head">${x[1]}–${x[2]}</div>`;
    tables.filter(t => t.active).forEach(t => {
      let b = bookings.find(z => z.date === d && z.slot === x[1] && (z.table_id === t.id || z.table === t.id) && z.status !== "Rejected");
      cells += b ? `<div class="cell ${b.status==="Confirmed"?"fixed":"pending"}">${b.status==="Confirmed"?`🔴 <b>${b.name}</b><br>${b.menu}`:"🔒 Sudah dipesan"}</div>` : `<div class="cell">🟢 Tersedia</div>`;
    });
    return cells;
  }).join("");
}

async function book(){
  let e = [];
  if(!sel.slot) e.push("jam");
  if(!sel.table) e.push("table");
  if(sel.menu === null) e.push("menu");
  if(!$("name").value.trim()) e.push("nama");
  if(!$("wa").value.trim()) e.push("WhatsApp");
  if(!$("proof").files.length) e.push("bukti transfer");
  if(sel.slot && locked($("date").value, sel.slot, sel.table)) e.push("slot sudah dipesan");
  if(sel.menu === "mix" && sel.m1 === sel.m2) e.push("2 flavour berbeda");

  if(e.length){
    $("msg").textContent = "Lengkapi: " + e.join(", ");
    $("msg").style.color = "#c92a2a";
    return;
  }

  $("btnBook").disabled = true;
  $("btnBook").textContent = "Menyimpan Reservasi...";

  let file = $("proof").files[0];
  let proofBase64 = await new Promise(r => {
    let reader = new FileReader();
    reader.onload = ev => r(ev.target.result);
    reader.readAsDataURL(file);
  });

  let menu = sel.menu === "mix" ? `Special Kreasi: ${menus[sel.m1??0][0]} × ${menus[sel.m2??1][0]}` : menus[sel.menu][0];
  let code = "MYO-" + Math.random().toString(36).slice(2, 7).toUpperCase();

  let payload = {
    code,
    date: $("date").value,
    slot: sel.slot,
    table_id: sel.table,
    name: $("name").value.trim(),
    wa: $("wa").value.trim(),
    pax: +$("pax").value || 1,
    menu,
    proof: proofBase64,
    status: "Pending"
  };

  const { error } = await db.from("bookings").insert([payload]);
  if(error){
    console.error("Gagal simpan Supabase:", error);
    bookings.push(payload);
    localStorage.setItem("MYO_BOOKINGS", JSON.stringify(bookings));
  }

  $("msg").textContent = `✓ ${code} tersimpan. Slot langsung terkunci sebagai Pending.`;
  $("msg").style.color = "#2b8a3e";

  let textWA = encodeURIComponent(`Halo Admin L'Union Pizza, saya reservasi MYO:\nKode: ${code}\nNama: ${payload.name}\nTanggal: ${payload.date}\nJam: ${payload.slot}\nMeja: ${payload.table_id}\nMenu: ${payload.menu}\nPax: ${payload.pax}\nBukti bayar sudah diupload.`);
  window.open(`https://wa.me/6285226099883?text=${textWA}`, '_blank');

  sel = {slot:null, table:null, menu:null, m1:null, m2:null};
  $("proof").value = "";
  $("btnBook").disabled = false;
  $("btnBook").textContent = "Konfirmasi Reservasi";
  syncData();
}

$("date").min = new Date().toISOString().slice(0, 10);
$("date").value = $("date").min;
$("date").onchange = () => { sel.slot = null; render(); };

db.channel("realtime-customer")
  .on("postgres_changes", { event: "*", schema: "public", table: "bookings" }, () => syncData())
  .on("postgres_changes", { event: "*", schema: "public", table: "app_config" }, () => syncData())
  .subscribe();

syncData();
</script>
</body>
</html>
