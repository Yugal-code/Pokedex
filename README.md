import os, zipfile, textwrap

base = "/mnt/data/ultimate-pokedex"
os.makedirs(base, exist_ok=True)

files = {
"index.html": r'''<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#e3350d">
  <title>Pokédex — Explore Every Pokémon</title>
  <link rel="preconnect" href="https://pokeapi.co">
  <link rel="preconnect" href="https://raw.githubusercontent.com">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header class="topbar">
    <a class="brand" href="#" aria-label="Pokédex home"><span class="brand-mark">◉</span><span>Poké<span>dex</span></span></a>
    <div class="header-note">THE WORLD OF POKÉMON, AT YOUR FINGERTIPS</div>
    <button id="favoritesToggle" class="ghost-btn" type="button">♡ Favorites <span id="favoriteCount">0</span></button>
  </header>

  <main>
    <section class="hero">
      <div class="hero-copy">
        <p class="eyebrow">NATIONAL POKÉDEX</p>
        <h1>Discover your<br><span>next favorite.</span></h1>
        <p class="hero-sub">Explore Pokémon, learn their abilities, check their stats and follow their evolution journeys.</p>
        <div class="search-wrap">
          <span aria-hidden="true">⌕</span>
          <input id="searchInput" type="search" placeholder="Search by name or Pokédex number…" autocomplete="off">
          <button id="searchClear" type="button" aria-label="Clear search">×</button>
        </div>
      </div>
      <div class="hero-orb" aria-hidden="true"><div class="orb-ball"><div></div></div><span>Gotta explore<br>them all!</span></div>
    </section>

    <section class="toolbar" aria-label="Pokémon filters">
      <div class="filter-group">
        <label for="typeFilter">Type</label>
        <select id="typeFilter"><option value="">All types</option></select>
      </div>
      <div class="filter-group">
        <label for="sortFilter">Sort by</label>
        <select id="sortFilter">
          <option value="id">Pokédex number</option>
          <option value="name">Name (A–Z)</option>
        </select>
      </div>
      <div class="filter-group">
        <label for="generationFilter">Generation</label>
        <select id="generationFilter">
          <option value="">All generations</option>
          <option value="1">Generation I</option><option value="2">Generation II</option>
          <option value="3">Generation III</option><option value="4">Generation IV</option>
          <option value="5">Generation V</option><option value="6">Generation VI</option>
          <option value="7">Generation VII</option><option value="8">Generation VIII</option>
          <option value="9">Generation IX</option>
        </select>
      </div>
      <label class="check-filter"><input id="favoritesOnly" type="checkbox"> Favorites only</label>
      <button id="resetFilters" class="reset-btn" type="button">Reset filters</button>
    </section>

    <section class="results-heading">
      <div><p class="eyebrow">YOUR FIELD GUIDE</p><h2 id="resultsTitle">Pokémon collection</h2></div>
      <span id="resultCount" class="result-count">Loading…</span>
    </section>

    <div id="status" class="status"><span class="loader"></span> Connecting to the PokéAPI…</div>
    <section id="pokemonGrid" class="pokemon-grid" aria-live="polite"></section>
    <div class="load-more-wrap"><button id="loadMore" class="primary-btn" type="button" hidden>Load more Pokémon</button></div>
  </main>

  <footer><span class="footer-ball">◉</span><p>Pokédex fan project · Pokémon data provided by <a href="https://pokeapi.co/" target="_blank" rel="noopener">PokéAPI</a>.</p><p class="fine-print">Pokémon and Pokémon character names are trademarks of Nintendo, Game Freak and The Pokémon Company.</p></footer>

  <div id="detailModal" class="modal" aria-hidden="true">
    <div class="modal-backdrop" data-close="true"></div>
    <article class="detail-panel" role="dialog" aria-modal="true" aria-labelledby="detailName">
      <button id="closeModal" class="close-modal" type="button" aria-label="Close details">×</button>
      <div id="detailContent"></div>
    </article>
  </div>
  <script src="script.js"></script>
</body>
</html>
''',
"style.css": r''':root{--red:#e3350d;--red-dark:#bd2608;--ink:#20232b;--muted:#747987;--paper:#f6f7fb;--line:#e7e9ef;--white:#fff;--shadow:0 16px 45px rgba(31,38,55,.09)}
*{box-sizing:border-box}html{scroll-behavior:smooth}body{margin:0;background:var(--paper);color:var(--ink);font-family:Inter,ui-sans-serif,system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif}.topbar{height:76px;padding:0 clamp(18px,5vw,76px);display:flex;align-items:center;justify-content:space-between;background:#fff;border-bottom:1px solid var(--line);position:sticky;top:0;z-index:5}.brand{display:flex;align-items:center;gap:10px;text-decoration:none;color:var(--ink);font-size:23px;font-weight:900;letter-spacing:-1px}.brand span:last-child span{color:var(--red)}.brand-mark{width:34px;height:34px;border:5px solid var(--red);border-radius:50%;display:grid;place-items:center;color:var(--red);font-size:18px;line-height:1}.header-note,.eyebrow{font-size:10px;letter-spacing:2px;font-weight:800;color:var(--muted)}.ghost-btn,.reset-btn{border:1px solid var(--line);background:#fff;border-radius:999px;padding:10px 15px;font-weight:750;color:var(--ink);cursor:pointer}.ghost-btn:hover,.reset-btn:hover{border-color:var(--red);color:var(--red)}.hero{margin:0 auto;max-width:1400px;min-height:330px;padding:55px clamp(20px,7vw,100px);background:linear-gradient(115deg,#20242e 0%,#303744 60%,#414b59 100%);color:#fff;display:flex;align-items:center;justify-content:space-between;overflow:hidden;position:relative}.hero:after{content:"";position:absolute;width:440px;height:440px;border:1px solid rgba(255,255,255,.08);border-radius:50%;right:10%;top:-170px;box-shadow:0 0 0 35px rgba(255,255,255,.025),0 0 0 75px rgba(255,255,255,.02)}.hero-copy{max-width:600px;position:relative;z-index:1}.hero .eyebrow{color:#ffb09c}.hero h1{font-size:clamp(38px,5vw,62px);line-height:1.02;letter-spacing:-2.5px;margin:13px 0}.hero h1 span{color:#ff795b}.hero-sub{color:#c5cbd5;max-width:480px;line-height:1.65;margin:16px 0 24px}.search-wrap{height:54px;max-width:530px;background:#fff;border-radius:12px;display:flex;align-items:center;padding:0 15px;gap:10px;color:#8b909c;box-shadow:0 10px 30px #1115}.search-wrap>span{font-size:27px}.search-wrap input{border:0;outline:0;flex:1;min-width:0;font:inherit;color:var(--ink)}.search-wrap button{border:0;background:none;font-size:24px;color:#8b909c;cursor:pointer}.hero-orb{width:230px;height:230px;position:relative;z-index:1;display:grid;place-items:center;margin-right:6%}.orb-ball{width:175px;height:175px;border:13px solid #fff;border-radius:50%;background:linear-gradient(#ed4a2d 0 47%,#252a34 47% 53%,#fff 53%);box-shadow:0 20px 50px #0005;position:relative}.orb-ball:after{content:"";position:absolute;width:48px;height:48px;border:10px solid #252a34;border-radius:50%;background:#fff;left:50%;top:50%;transform:translate(-50%,-50%)}.orb-ball div{position:absolute;width:17px;height:17px;border-radius:50%;background:#fff;left:50%;top:50%;transform:translate(-50%,-50%);z-index:1}.hero-orb>span{position:absolute;right:-45px;bottom:8px;font-weight:900;font-size:13px;line-height:1.35;transform:rotate(-8deg);color:#ffc0b1}.toolbar{max-width:1240px;margin:30px auto 42px;padding:0 22px;display:flex;align-items:end;gap:14px;flex-wrap:wrap}.filter-group{display:grid;gap:7px}.filter-group label{font-size:11px;font-weight:800;color:var(--muted)}select{min-width:145px;padding:11px 34px 11px 13px;border:1px solid var(--line);border-radius:9px;background:#fff;color:var(--ink);font:inherit;font-size:13px;outline-color:var(--red)}.check-filter{display:flex;align-items:center;gap:8px;font-size:13px;font-weight:700;padding:11px 0}.check-filter input{accent-color:var(--red);width:16px;height:16px}.reset-btn{margin-left:auto}.results-heading{max-width:1200px;margin:0 auto 20px;padding:0 22px;display:flex;justify-content:space-between;align-items:end}.results-heading .eyebrow{margin:0 0 6px;color:var(--red)}h2{font-size:27px;letter-spacing:-.8px;margin:0}.result-count{font-size:12px;color:var(--muted);font-weight:700;padding-bottom:4px}.pokemon-grid{max-width:1200px;margin:0 auto;padding:0 22px 35px;display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:17px}.pokemon-card{--tint:#f1f2f5;background:#fff;border:1px solid var(--line);border-radius:16px;min-width:0;padding:14px;position:relative;overflow:hidden;cursor:pointer;transition:transform .2s,box-shadow .2s,border-color .2s}.pokemon-card:before{content:"";position:absolute;width:155px;height:155px;border-radius:50%;background:var(--tint);right:-50px;top:30px;opacity:.75}.pokemon-card:hover{transform:translateY(-5px);box-shadow:var(--shadow);border-color:#d4d8e2}.card-top{display:flex;justify-content:space-between;align-items:center;position:relative;z-index:1}.poke-number{font-size:11px;color:#9298a5;font-weight:800}.favorite-btn{border:0;background:#f5f6f8;border-radius:50%;width:31px;height:31px;font-size:18px;color:#9ba0aa;cursor:pointer}.favorite-btn.active{color:var(--red);background:#fff0ec}.pokemon-art{height:142px;display:grid;place-items:center;position:relative;z-index:1}.pokemon-art img{width:145px;height:145px;object-fit:contain;filter:drop-shadow(0 9px 7px #20232b18);transition:transform .2s}.pokemon-card:hover .pokemon-art img{transform:scale(1.08)}.pokemon-card h3{text-transform:capitalize;font-size:18px;margin:2px 0 9px;position:relative;z-index:1}.types{display:flex;gap:6px;position:relative;z-index:1}.type-pill{font-size:10px;font-weight:800;text-transform:uppercase;letter-spacing:.5px;color:#fff;padding:5px 9px;border-radius:99px}.card-bottom{display:flex;justify-content:space-between;align-items:center;margin-top:14px;color:#9298a5;font-size:10px;font-weight:700;position:relative;z-index:1}.status{text-align:center;color:var(--muted);padding:35px 15px;font-size:14px}.loader{display:inline-block;width:15px;height:15px;border:2px solid #ddd;border-top-color:var(--red);border-radius:50%;vertical-align:-3px;margin-right:8px;animation:spin .8s linear infinite}@keyframes spin{to{transform:rotate(360deg)}}.load-more-wrap{text-align:center;padding:8px 20px 60px}.primary-btn{border:0;border-radius:10px;background:var(--red);color:#fff;padding:14px 25px;font-weight:800;cursor:pointer;box-shadow:0 8px 20px #e3350d30}.primary-btn:hover{background:var(--red-dark)}footer{text-align:center;padding:30px 15px;background:#fff;border-top:1px solid var(--line);color:var(--muted);font-size:12px}.footer-ball{font-size:23px;color:var(--red)}footer p{margin:8px}footer a{color:var(--red);font-weight:700}.fine-print{font-size:10px}.modal{position:fixed;inset:0;z-index:10;display:none;align-items:center;justify-content:center;padding:18px}.modal.open{display:flex}.modal-backdrop{position:absolute;inset:0;background:#151923b8;backdrop-filter:blur(5px)}.detail-panel{position:relative;background:#fff;border-radius:22px;width:min(760px,100%);max-height:92vh;overflow:auto;box-shadow:0 25px 90px #0005;z-index:1}.close-modal{position:absolute;right:15px;top:13px;border:0;background:#ffffffd9;border-radius:50%;width:36px;height:36px;font-size:25px;cursor:pointer;z-index:3}.detail-hero{padding:28px 32px 22px;background:var(--detail-tint,#f1f2f5);display:grid;grid-template-columns:1fr 220px;align-items:center;min-height:235px}.detail-hero .eyebrow{color:#727987}.detail-hero h2{text-transform:capitalize;font-size:36px;margin:8px 0}.detail-hero img{width:210px;height:210px;object-fit:contain;filter:drop-shadow(0 10px 8px #0002)}.detail-body{padding:25px 32px 32px}.detail-body h3{font-size:14px;margin:23px 0 12px}.detail-body h3:first-child{margin-top:0}.description{color:#687080;font-size:13px;line-height:1.65}.quick-facts{display:flex;gap:25px;flex-wrap:wrap;margin:16px 0}.fact strong{display:block;font-size:17px}.fact span{font-size:10px;color:var(--muted);text-transform:uppercase;font-weight:800}.stat-row{display:grid;grid-template-columns:105px 35px 1fr;align-items:center;gap:10px;margin:9px 0;font-size:11px}.stat-row span:first-child{color:#707786;font-weight:700}.stat-track{height:7px;background:#edf0f4;border-radius:9px;overflow:hidden}.stat-fill{height:100%;background:var(--red);border-radius:9px}.evolution{display:flex;align-items:center;gap:12px;flex-wrap:wrap}.evo-item{text-align:center;text-transform:capitalize;font-size:11px;font-weight:800}.evo-item img{width:78px;height:78px;object-fit:contain;display:block}.evo-arrow{color:#9aa0aa;font-size:20px}.ability-list{display:flex;gap:7px;flex-wrap:wrap}.ability-chip{background:#f1f3f7;padding:7px 10px;border-radius:8px;font-size:11px;font-weight:700;text-transform:capitalize}.error{color:#b42318;background:#fff0ee;border-radius:10px;padding:15px;text-align:center}
@media(max-width:900px){.pokemon-grid{grid-template-columns:repeat(3,minmax(0,1fr))}.hero-orb{margin-right:2%}}
@media(max-width:650px){.topbar{height:64px;padding:0 16px}.header-note{display:none}.brand{font-size:21px}.hero{min-height:300px;padding:38px 22px}.hero-orb{position:absolute;right:-80px;top:30px;opacity:.22}.hero-copy{max-width:100%}.hero h1{font-size:43px}.hero-sub{font-size:13px;max-width:390px}.search-wrap{height:50px}.toolbar{margin:22px auto 32px;gap:10px}.filter-group{flex:1;min-width:calc(50% - 10px)}.filter-group select{width:100%;min-width:0}.check-filter{flex:1}.reset-btn{margin-left:0}.results-heading{align-items:center}.results-heading h2{font-size:23px}.pokemon-grid{grid-template-columns:repeat(2,minmax(0,1fr));gap:11px;padding:0 13px 25px}.pokemon-card{padding:11px;border-radius:13px}.pokemon-art{height:115px}.pokemon-art img{width:120px;height:120px}.pokemon-card h3{font-size:16px}.detail-hero{grid-template-columns:1fr 130px;padding:25px 20px 18px;min-height:180px}.detail-hero h2{font-size:28px}.detail-hero img{width:135px;height:135px}.detail-body{padding:22px 20px 28px}.stat-row{grid-template-columns:90px 28px 1fr;gap:7px}}
@media(max-width:360px){.pokemon-grid{grid-template-columns:1fr}.pokemon-art img{width:140px;height:140px}.pokemon-art{height:135px}}
''',
"script.js": r'''const API = "https://pokeapi.co/api/v2";
const grid = document.getElementById("pokemonGrid");
const statusBox = document.getElementById("status");
const searchInput = document.getElementById("searchInput");
const typeFilter = document.getElementById("typeFilter");
const generationFilter = document.getElementById("generationFilter");
const sortFilter = document.getElementById("sortFilter");
const loadMoreBtn = document.getElementById("loadMore");
const modal = document.getElementById("detailModal");
const detailContent = document.getElementById("detailContent");

const generationRanges = {1:[1,151],2:[152,251],3:[252,386],4:[387,493],5:[494,649],6:[650,721],7:[722,809],8:[810,905],9:[906,1025]};
const typeColors = {normal:"#a8a77a",fire:"#ee8130",water:"#6390f0",electric:"#f7d02c",grass:"#7ac74c",ice:"#96d9d6",fighting:"#c22e28",poison:"#a33ea1",ground:"#e2bf65",flying:"#a98ff3",psychic:"#f95587",bug:"#a6b91a",rock:"#b6a136",ghost:"#735797",dragon:"#6f35fc",dark:"#705746",steel:"#b7b7ce",fairy:"#d685ad"};
const generationById = id => Object.entries(generationRanges).find(([,r])=>id>=r[0]&&id<=r[1])?.[0] || "9";
let pokemon = [];
let nextOffset = 0;
let currentLimit = 36;
let showingFavorites = false;
let favorites = new Set(JSON.parse(localStorage.getItem("pokedex-favorites") || "[]"));
let searchTimer;

function titleCase(s){return s.charAt(0).toUpperCase()+s.slice(1)}
function imageFor(id){return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${id}.png`}
function typePill(type){return `<span class="type-pill" style="background:${typeColors[type]||"#888"}">${type}</span>`}
function setStatus(message, show=true){statusBox.textContent=message;statusBox.style.display=show?"block":"none"}
function updateFavoriteCount(){document.getElementById("favoriteCount").textContent=favorites.size}
function saveFavorites(){localStorage.setItem("pokedex-favorites",JSON.stringify([...favorites]));updateFavoriteCount()}
function cardTemplate(p){
  const types=p.types.map(t=>t.type.name);
  const tint=typeColors[types[0]]||"#eee";
  return `<article class="pokemon-card" style="--tint:${tint}22" data-id="${p.id}" tabindex="0" role="button" aria-label="View ${p.name}">
    <div class="card-top"><span class="poke-number">#${String(p.id).padStart(4,"0")}</span><button class="favorite-btn ${favorites.has(p.id)?"active":""}" data-fav="${p.id}" aria-label="Toggle favorite">${favorites.has(p.id)?"♥":"♡"}</button></div>
    <div class="pokemon-art"><img loading="lazy" src="${imageFor(p.id)}" alt="${p.name}" onerror="this.src='https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${p.id}.png'"></div>
    <h3>${p.name.replace(/-/g," ")}</h3><div class="types">${types.map(typePill).join("")}</div>
    <div class="card-bottom"><span>GENERATION ${generationById(p.id)}</span><span>VIEW DETAILS ↗</span></div>
  </article>`;
}
function render(){
  const query=searchInput.value.trim().toLowerCase();
  let list=pokemon.filter(p=>{
    const matchesName=p.name.includes(query)||String(p.id)===query;
    const matchesType=!typeFilter.value||p.types.some(t=>t.type.name===typeFilter.value);
    const gen=generationFilter.value;
    const matchesGen=!gen||generationById(p.id)===gen;
    const matchesFav=!showingFavorites||favorites.has(p.id);
    return matchesName&&matchesType&&matchesGen&&matchesFav;
  });
  if(sortFilter.value==="name")list.sort((a,b)=>a.name.localeCompare(b.name));
  else list.sort((a,b)=>a.id-b.id);
  grid.innerHTML=list.map(cardTemplate).join("");
  document.getElementById("resultCount").textContent=`Showing ${list.length} Pokémon`;
  document.getElementById("resultsTitle").textContent=showingFavorites?"Your favorites":"Pokémon collection";
  loadMoreBtn.hidden=showingFavorites||Boolean(query)||Boolean(typeFilter.value)||Boolean(generationFilter.value)||nextOffset>=1025;
  if(!list.length)setStatus(showingFavorites?"No favorites yet. Tap the heart on a Pokémon to save it.":"No Pokémon found. Try changing your search or filters.");
  else setStatus("",false);
}
async function loadTypes(){
  try{const r=await fetch(`${API}/type`);const d=await r.json();d.results.filter(x=>!["unknown","shadow"].includes(x.name)).forEach(t=>{const o=document.createElement("option");o.value=t.name;o.textContent=titleCase(t.name);typeFilter.appendChild(o)})}catch(e){console.warn("Could not load type list",e)}
}
async function loadBatch(){
  loadMoreBtn.disabled=true;loadMoreBtn.textContent="Loading…";
  setStatus("Loading Pokémon from the PokéAPI…");
  try{
    const r=await fetch(`${API}/pokemon?limit=${currentLimit}&offset=${nextOffset}`);
    if(!r.ok)throw new Error("PokéAPI request failed");
    const d=await r.json();
    const details=await Promise.all(d.results.map(async item=>{const response=await fetch(item.url);return response.json()}));
    pokemon.push(...details);nextOffset+=d.results.length;render();
  }catch(e){setStatus("Unable to load Pokémon. Please check your internet connection and try again.");console.error(e)}
  loadMoreBtn.disabled=false;loadMoreBtn.textContent="Load more Pokémon";
}
async function fetchPokemon(idOrName){
  const r=await fetch(`${API}/pokemon/${encodeURIComponent(idOrName)}`);
  if(!r.ok)throw new Error("Pokémon not found");
  return r.json();
}
async function openDetails(id){
  modal.classList.add("open");modal.setAttribute("aria-hidden","false");document.body.style.overflow="hidden";
  detailContent.innerHTML='<div class="status"><span class="loader"></span>Loading Pokémon documentation…</div>';
  try{
    const p=await fetchPokemon(id);
    const [species, evolution]=await Promise.all([fetch(p.species.url).then(r=>r.json()),fetch(p.species.url).then(r=>r.json()).then(s=>fetch(s.evolution_chain.url).then(r=>r.json()))]);
    const flavor=species.flavor_text_entries.find(x=>x.language.name==="en")?.flavor_text.replace(/[\n\f]/g," ")||"No Pokédex description available.";
    const stats=p.stats.map(s=>`<div class="stat-row"><span>${s.stat.name.replace("-"," ")}</span><strong>${s.base_stat}</strong><div class="stat-track"><div class="stat-fill" style="width:${Math.min(100,s.base_stat/255*100)}%"></div></div></div>`).join("");
    const abilities=p.abilities.map(a=>`<span class="ability-chip">${a.ability.name.replace(/-/g," ")}${a.is_hidden?" · hidden":""}</span>`).join("");
    const chain=flattenEvolution(evolution.chain);
    const evoHtml=chain.length>1?chain.map((item,i)=>`${i?'<span class="evo-arrow">→</span>':""}<div class="evo-item"><img loading="lazy" src="${imageFor(item.id)}" alt="${item.name}">${item.name.replace(/-/g," ")}</div>`).join(""):"<p class='description'>This Pokémon has no recorded evolution chain.</p>";
    const types=p.types.map(t=>t.type.name);
    const tint=(typeColors[types[0]]||"#eee")+"22";
    detailContent.innerHTML=`
      <section class="detail-hero" style="--detail-tint:${tint}">
        <div><p class="eyebrow">NATIONAL POKÉDEX #${String(p.id).padStart(4,"0")}</p><h2 id="detailName">${p.name.replace(/-/g," ")}</h2><div class="types">${types.map(typePill).join("")}</div></div>
        <img src="${imageFor(p.id)}" alt="${p.name}">
      </section>
      <section class="detail-body">
        <h3>Pokédex entry</h3><p class="description">${flavor}</p>
        <div class="quick-facts"><div class="fact"><strong>${(p.height/10).toFixed(1)} m</strong><span>Height</span></div><div class="fact"><strong>${(p.weight/10).toFixed(1)} kg</strong><span>Weight</span></div><div class="fact"><strong>${p.base_experience??"—"}</strong><span>Base XP</span></div><div class="fact"><strong>Gen ${generationById(p.id)}</strong><span>Generation</span></div></div>
        <h3>Base stats</h3>${stats}
        <h3>Abilities</h3><div class="ability-list">${abilities}</div>
        <h3>Evolution chain</h3><div class="evolution">${evoHtml}</div>
        <h3>Pokémon data</h3><p class="description">Species: ${species.genera.find(g=>g.language.name==="en")?.genus||"Unknown"} · Capture rate: ${species.capture_rate} · Habitat: ${species.habitat?.name||"Unknown"} · Legendary: ${species.is_legendary?"Yes":"No"} · Mythical: ${species.is_mythical?"Yes":"No"}</p>
      </section>`;
  }catch(e){detailContent.innerHTML='<div class="error">Could not load this Pokémon. Please try again.</div>';console.error(e)}
}
function flattenEvolution(node,out=[]){
  const match=node.species.url.match(/\/pokemon-species\/(\d+)\//);
  out.push({name:node.species.name,id:Number(match?.[1]||0)});
  if(node.evolves_to.length)flattenEvolution(node.evolves_to[0],out);
  return out;
}
function closeDetails(){modal.classList.remove("open");modal.setAttribute("aria-hidden","true");document.body.style.overflow=""}

grid.addEventListener("click",e=>{
  const fav=e.target.closest("[data-fav]");
  if(fav){const id=Number(fav.dataset.fav);favorites.has(id)?favorites.delete(id):favorites.add(id);saveFavorites();render();return}
  const card=e.target.closest("[data-id]");if(card)openDetails(card.dataset.id);
});
grid.addEventListener("keydown",e=>{if((e.key==="Enter"||e.key===" ")&&e.target.matches("[data-id]")){e.preventDefault();openDetails(e.target.dataset.id)}});
searchInput.addEventListener("input",()=>{clearTimeout(searchTimer);searchTimer=setTimeout(render,120)});
document.getElementById("searchClear").addEventListener("click",()=>{searchInput.value="";render();searchInput.focus()});
[typeFilter,generationFilter,sortFilter].forEach(el=>el.addEventListener("change",render));
document.getElementById("favoritesToggle").addEventListener("click",()=>{showingFavorites=!showingFavorites;document.getElementById("favoritesToggle").classList.toggle("active",showingFavorites);render()});
document.getElementById("favoritesOnly").addEventListener("change",e=>{showingFavorites=e.target.checked;render()});
document.getElementById("resetFilters").addEventListener("click",()=>{searchInput.value="";typeFilter.value="";generationFilter.value="";sortFilter.value="id";showingFavorites=false;document.getElementById("favoritesOnly").checked=false;render()});
loadMoreBtn.addEventListener("click",loadBatch);
document.getElementById("closeModal").addEventListener("click",closeDetails);
modal.addEventListener("click",e=>{if(e.target.dataset.close==="true")closeDetails()});
document.addEventListener("keydown",e=>{if(e.key==="Escape")closeDetails()});
updateFavoriteCount();loadTypes();loadBatch();
'''
,
"README.md": r'''# Ultimate Pokédex

A responsive Pokémon encyclopedia website built with plain HTML, CSS and JavaScript.

## Run locally
1. Download and extract the ZIP.
2. Open `index.html` in a modern browser.
3. Keep an internet connection available: Pokémon data is loaded from the public PokéAPI.

For a local development server, you can also run:
```bash
python -m http.server 8000
```
Then visit `http://localhost:8000`.

## Deploy to GitHub Pages
1. Create a new GitHub repository.
2. Upload `index.html`, `style.css`, `script.js`, and `README.md` to the repository root.
3. Open **Settings → Pages**.
4. Under **Build and deployment**, choose **Deploy from a branch**.
5. Select `main` and `/ (root)`, then save.
6. Wait for GitHub Pages to publish your site.

## Features
- Responsive layout for phones, tablets and laptops
- Pokémon listing loaded in batches from PokéAPI
- Search by Pokémon name or Pokédex number
- Type, generation and name/number sorting filters
- Favorites saved in browser local storage
- Detail view with Pokédex description, types, height, weight, base stats, abilities, evolution chain and species information

## Data and attribution
Pokémon data is provided by [PokéAPI](https://pokeapi.co/). Pokémon and Pokémon character names are trademarks of Nintendo, Game Freak and The Pokémon Company.

## Notes
- An internet connection is required to fetch Pokémon data and artwork.
- Favorites are stored in the current browser on the current device.
- The app loads Pokémon in batches to avoid fetching the entire Pokédex at once.
'''
}
for name, content in files.items():
    with open(os.path.join(base, name), "w", encoding="utf-8") as f:
        f.write(content)

zip_path = "/mnt/data/ultimate-pokedex-website.zip"
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for name in files:
        z.write(os.path.join(base, name), arcname=f"ultimate-pokedex/{name}")

print(zip_path, os.path.getsize(zip_path), list(files.keys()))
