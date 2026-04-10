
<!DOCTYPE html>
<html>
<head>
<title>MSU Play Store</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
/* Basic Body */
body {
  font-family: Arial;
  margin:0;
  background: #f0f2f5;
  color: #000;
  scroll-behavior: smooth;
  transition: background 0.3s, color 0.3s;
}

/* Dark Mode */
body.dark-mode {
  background: #121212;
  color: #f0f0f0;
}

/* Header */
header {
  background: #0d6efd;
  color: white;
  padding: 15px;
  text-align: center;
  position: sticky;
  top: 0;
  z-index: 1000;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
}
header h1 { flex-basis: 100%; margin:0; }
header input, header select, header button {
  padding: 8px; border-radius:5px; border:none; outline:none; transition: 0.3s;
}

/* Form */
form {
  background: white;
  margin: 15px;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transition: transform 0.3s;
}
form:hover { transform: translateY(-2px); }
form input, form select { width:95%; margin:8px 0; padding:10px; border-radius:6px; border:1px solid #ccc; }
form button { background:#28a745; color:white; padding:10px 20px; border:none; border-radius:8px; cursor:pointer; transition: 0.3s;}
form button:hover { background:#218838; transform: scale(1.05); }

/* Apps Grid */
.apps { display:grid; grid-template-columns: repeat(auto-fit, minmax(150px,1fr)); gap:15px; padding:15px;}
.app {
  background:white; padding:15px; border-radius:15px; text-align:center;
  box-shadow:0 4px 10px rgba(0,0,0,0.1); cursor:pointer;
  opacity:0; transform:translateY(20px); animation: fadeInUp 0.5s forwards;
  transition: transform 0.3s, box-shadow 0.3s;
}
body.dark-mode .app { background:#1e1e1e; color:#f0f0f0; }
.app:hover { transform:translateY(-5px) scale(1.02); box-shadow:0 6px 18px rgba(0,0,0,0.15);}
.app img { width:80px; height:80px; border-radius:12px; margin-bottom:8px;}
button.delete { background:#dc3545; margin-top:5px; transition:0.3s;}
button.delete:hover { background:#c82333; transform: scale(1.05); }
a button { width:100%; margin-top:5px; background:#007bff; transition:0.3s;}
a button:hover { background:#0069d9; transform: scale(1.03); }

/* Modal */
.modal { display:none; position:fixed; z-index:2000; left:0; top:0; width:100%; height:100%; overflow:auto; background-color: rgba(0,0,0,0.6); animation: fadeIn 0.3s;}
.modal-content { background-color:#fefefe; margin:10% auto; padding:20px; border-radius:15px; width:90%; max-width:400px; text-align:center; position:relative; animation: slideUp 0.3s;}
body.dark-mode .modal-content { background:#1e1e1e; color:#f0f0f0; }
.close { color:#aaa; position:absolute; right:15px; top:10px; font-size:28px; font-weight:bold; cursor:pointer; transition:0.3s;}
.close:hover { color:black;}
.modal img { width:120px; border-radius:12px; margin-bottom:10px; }

/* Animations */
@keyframes fadeInUp { to {opacity:1; transform:translateY(0);} }
@keyframes fadeIn { from{opacity:0;} to{opacity:1;} }
@keyframes slideUp { from{transform:translateY(50px); opacity:0;} to{transform:translateY(0); opacity:1;} }

/* Responsive */
@media screen and (max-width:500px){
  header input, header select, header button{ width:98%; margin:5px 0;}
}
</style>
</head>
<body>

<header>
  <h1>MSUPlay Store</h1>
  <input type="text" id="search" placeholder="Search apps..." onkeyup="filterApps()">
  <select id="category" onchange="filterApps()">
    <option value="">All Categories</option>
    <option value="Game">Game</option>
    <option value="Tool">Tool</option>
    <option value="Education">Education</option>
  </select>
  <select id="sort" onchange="sortApps()">
    <option value="">Sort By</option>
    <option value="newest">Newest First</option>
    <option value="oldest">Oldest First</option>
    <option value="alphabetical">A → Z</option>
  </select>
  <button onclick="toggleDarkMode()" id="darkModeBtn">🌙 Dark Mode</button>
</header>

<form onsubmit="addApp(); return false;">
  <h3>Upload App</h3>
  <input type="text" id="name" placeholder="App Name" required>
  <input type="text" id="image" placeholder="Image URL" required>
  <input type="text" id="link" placeholder="Download Link" required>
  <select id="appCategory" required>
    <option value="">Select Category</option>
    <option value="Game">Game</option>
    <option value="Tool">Tool</option>
    <option value="Education">Education</option>
  </select>
  <input type="text" id="description" placeholder="Short Description" required>
  <input type="text" id="review" placeholder="Initial Review (optional)">
  <input type="number" id="rating" placeholder="Initial Rating (1-5)" min="1" max="5">
  <button type="submit">Add App</button>
</form>

<div class="apps" id="appsList"></div>

<!-- Modal -->
<div id="appModal" class="modal">
  <div class="modal-content">
    <span class="close" onclick="closeModal()">&times;</span>
    <img id="modalImg" src="">
    <h2 id="modalName"></h2>
    <p id="modalCategory"></p>
    <p id="modalDescription"></p>
    <p id="modalRating"></p>
    <div id="modalReviews"></div>
    <a id="modalLink" href="" target="_blank"><button>Download</button></a>
  </div>
</div>

<script>
let apps = JSON.parse(localStorage.getItem("apps")) || [];

// Show Apps
function showApps(filteredApps = apps){
  document.getElementById("appsList").innerHTML = "";
  filteredApps.forEach((app,index)=>{
    let appBox = document.createElement("div");
    appBox.className = "app";
    appBox.innerHTML = `
      <img src="${app.image}">
      <h3>${app.name}</h3>
      <p>Category: ${app.category}</p>
      <button class="delete" onclick="deleteApp(${index})">Delete</button>
    `;
    appBox.onclick = function(e){
      if(e.target.className !== "delete"){ openModal(app); }
    };
    document.getElementById("appsList").appendChild(appBox);
  });
}

// Add App
function addApp(){
  let name=document.getElementById("name").value;
  let image=document.getElementById("image").value;
  let link=document.getElementById("link").value;
  let category=document.getElementById("appCategory").value;
  let description=document.getElementById("description").value;
  let review=document.getElementById("review").value;
  let rating=parseInt(document.getElementById("rating").value) || 0;

  let newApp={name,image,link,category,description,reviews:review?[review]:[],ratings:rating?[rating]:[]};
  apps.push(newApp);
  localStorage.setItem("apps",JSON.stringify(apps));
  showApps();

  document.getElementById("name").value="";
  document.getElementById("image").value="";
  document.getElementById("link").value="";
  document.getElementById("appCategory").value="";
  document.getElementById("description").value="";
  document.getElementById("review").value="";
  document.getElementById("rating").value="";
}

// Delete App
function deleteApp(index){
  apps.splice(index,1);
  localStorage.setItem("apps",JSON.stringify(apps));
  showApps();
}

// Search + Filter
function filterApps(){
  let query=document.getElementById("search").value.toLowerCase();
  let category=document.getElementById("category").value;
  let filtered=apps.filter(app=>app.name.toLowerCase().includes(query) && (category===""||app.category===category));
  showApps(filtered);
}

// Sorting
function sortApps(){
  let type=document.getElementById("sort").value;
  if(type==="newest"){ apps.sort((a,b)=>apps.indexOf(b)-apps.indexOf(a)); }
  else if(type==="oldest"){ apps.sort((a,b)=>apps.indexOf(a)-apps.indexOf(b)); }
  else if(type==="alphabetical"){ apps.sort((a,b)=>a.name.localeCompare(b.name)); }
  filterApps();
}

// Dark Mode
function toggleDarkMode(){
  document.body.classList.toggle("dark-mode");
  let btn=document.getElementById("darkModeBtn");
  btn.innerText=document.body.classList.contains("dark-mode")?"☀ Light Mode":"🌙 Dark Mode";
}

// Modal
function openModal(app){
  document.getElementById("modalImg").src=app.image;
  document.getElementById("modalName").innerText=app.name;
  document.getElementById("modalCategory").innerText="Category: "+app.category;
  document.getElementById("modalDescription").innerText=app.description;

  let avg=app.ratings.length ? (app.ratings.reduce((a,b)=>a+b,0)/app.ratings.length).toFixed(1) : "No ratings yet";
  document.getElementById("modalRating").innerText="⭐ Rating: "+avg;

  let rev="<h4>Reviews:</h4>";
  if(app.reviews.length){ rev+="<ul>"+app.reviews.map(r=>"<li>"+r+"</li>").join("")+"</ul>"; }
  else{ rev+="<p>No reviews yet.</p>"; }
  document.getElementById("modalReviews").innerHTML=rev;

  document.getElementById("modalLink").href=app.link;
  document.getElementById("appModal").style.display="block";
}
function closeModal(){ document.getElementById("appModal").style.display="none"; }
window.onclick=function(e){ if(e.target==document.getElementById("appModal")) closeModal(); }

showApps();
</script>
</body>
</html>
