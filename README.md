<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Pinsort</title>

<style>
*{box-sizing:border-box}
body{margin:0;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Arial;background:#fafafa}

/* HEADER */
header{
  position:sticky;top:0;z-index:10;
  background:white;padding:10px 16px;
  display:flex;align-items:center;gap:12px;
  border-bottom:1px solid #ddd
}
.logo{font-size:28px;font-weight:800;color:#e91e63}
.search{flex:1}
.search input{
  width:100%;padding:10px 16px;border-radius:999px;
  border:none;background:#efefef;outline:none
}
header button{background:none;border:none;font-size:20px;cursor:pointer}

/* AUTH */
#auth,#adminPanel,#rating{text-align:center;padding:15px}
input,textarea{
  padding:8px;margin:4px;border-radius:10px;
  border:1px solid #ccc
}

/* GRID */
#grid{
  column-count:5;
  column-gap:16px;
  padding:16px
}
.pin{
  background:white;border-radius:16px;
  margin-bottom:16px;overflow:hidden;
  break-inside:avoid;position:relative
}
.pin img{width:100%;display:block}

/* ICONS */
.like,.delete{
  position:absolute;top:8px;
  background:white;border-radius:50%;
  padding:6px;cursor:pointer
}
.like{right:8px}
.delete{left:8px;background:#e91e63;color:white}
.liked{color:#e91e63}

/* STARS */
.star{font-size:26px;cursor:pointer;color:#ccc}
.star.selected{color:#ffc107}

/* RESPONSIVE */
@media(max-width:1000px){#grid{column-count:3}}
@media(max-width:600px){#grid{column-count:2}}
</style>
</head>

<body>

<header>
  <div class="logo">Pinsort</div>
  <div class="search"><input placeholder="Buscar ideas"></div>
  <button onclick="showAll()">🏠</button>
  <button onclick="showFavs()">❤️</button>
</header>

<div id="auth">
  <h3>Iniciar sesión / Registro</h3>
  <input id="user" placeholder="Usuario">
  <input id="pass" type="password" placeholder="Contraseña"><br>
  <button onclick="login()">Entrar</button>
  <button onclick="register()">Registrarse</button>
  <p id="welcome"></p>
</div>

<div id="adminPanel" style="display:none">
  <h3>Panel administrador</h3>
  <input id="imgUrl" placeholder="URL de la imagen">
  <button onclick="addPhoto()">Subir foto</button>
  <hr>
  <h4>Opiniones de usuarios</h4>
  <div id="reviews"></div>
</div>

<div id="rating" style="display:none">
  <h3>Califica Pinsort</h3>
  <div id="stars"></div>
  <textarea id="comment" placeholder="Comentario (opcional)"></textarea><br>
  <button onclick="sendRating()">Enviar</button>
</div>

<div id="grid"></div>

<script>
const ADMIN_USER="Fernanda123",ADMIN_PASS="fernñ";

let users=JSON.parse(localStorage.users||"[]");
let photos=JSON.parse(localStorage.photos||"[]");
let likes=JSON.parse(localStorage.likes||"{}");
let ratings=JSON.parse(localStorage.ratings||"[]");

let currentUser=null,selectedStars=0;

/* REGISTRO */
function register(){
  if(!user.value||!pass.value) return alert("Completa campos");
  if(user.value===ADMIN_USER||users.find(u=>u.user===user.value))
    return alert("Usuario ya existe");
  users.push({user:user.value,pass:pass.value,first:Date.now()});
  localStorage.users=JSON.stringify(users);
  alert("Usuario creado");
}

/* LOGIN */
function login(){
  if(user.value===ADMIN_USER && pass.value===ADMIN_PASS){
    currentUser=ADMIN_USER;
    adminPanel.style.display="block";
  }else{
    let u=users.find(x=>x.user===user.value&&x.pass===pass.value);
    if(!u) return alert("Datos incorrectos");
    currentUser=u.user;
    adminPanel.style.display="none";
    checkRating(u);
  }
  if(!likes[currentUser]) likes[currentUser]=[];
  localStorage.likes=JSON.stringify(likes);
  welcome.innerText="Hola, "+currentUser;
  render();
  showReviews();
}

/* SUBIR FOTO */
function addPhoto(){
  if(currentUser!==ADMIN_USER||!imgUrl.value) return;
  photos.push({url:imgUrl.value});
  localStorage.photos=JSON.stringify(photos);
  imgUrl.value="";
  render();
}

/* BORRAR FOTO */
function deletePhoto(i){
  if(currentUser!==ADMIN_USER) return;
  if(!confirm("¿Borrar esta foto?")) return;
  photos.splice(i,1);
  localStorage.photos=JSON.stringify(photos);
  render();
}

/* LIKE */
function toggleLike(i){
  let f=likes[currentUser];
  f.includes(i)?f.splice(f.indexOf(i),1):f.push(i);
  localStorage.likes=JSON.stringify(likes);
  render();
}

/* MOSTRAR */
function render(fav=false){
  grid.innerHTML="";
  photos.forEach((p,i)=>{
    if(fav && !likes[currentUser]?.includes(i)) return;
    grid.innerHTML+=`
    <div class="pin">
      ${currentUser===ADMIN_USER?`<div class="delete" onclick="deletePhoto(${i})">✖</div>`:""}
      <img src="${p.url}">
      <div class="like ${likes[currentUser]?.includes(i)?"liked":""}"
           onclick="toggleLike(${i})">❤️</div>
    </div>`;
  });
}
function showFavs(){if(currentUser)render(true)}
function showAll(){render()}

/* ⭐ CALIFICACIÓN */
function checkRating(u){
  let days=(Date.now()-u.first)/(1000*60*60*24);
  if(days>=7 && !ratings.find(r=>r.user===u.user)){
    rating.style.display="block";
    stars.innerHTML="";
    for(let i=1;i<=5;i++)
      stars.innerHTML+=`<span class="star" onclick="selectStar(${i})">★</span>`;
  }
}
function selectStar(n){
  selectedStars=n;
  document.querySelectorAll(".star")
    .forEach((s,i)=>s.classList.toggle("selected",i<n));
}
function sendRating(){
  if(!selectedStars) return alert("Elige estrellas");
  ratings.push({user:currentUser,stars:selectedStars,comment:comment.value});
  localStorage.ratings=JSON.stringify(ratings);
  rating.style.display="none";
  alert("Gracias por tu opinión 💗");
}

/* ADMIN VE OPINIONES */
function showReviews(){
  if(currentUser!==ADMIN_USER) return;
  reviews.innerHTML="";
  ratings.forEach(r=>{
    reviews.innerHTML+=`⭐${r.stars} - ${r.user}<br>${r.comment||""}<hr>`;
  });
}
</script>

</body>
</html>
