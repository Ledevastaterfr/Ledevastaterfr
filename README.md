<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>SIGP-GN Web Secure</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <style>
    :root {
      --primary: #003366;
      --light: #f0f0f0;
      --accent: #0055aa;
      --bg-list: #eef;
      --centered: flex;
    }
    * { box-sizing:border-box; margin:0; padding:0; }
    body, html { height:100%; font-family: Arial, sans-serif; }
    .hidden { display:none; }
    header { background:var(--primary); color:#fff; padding:1rem; text-align:center; }
    main { padding:1rem; }
    .center { display: var(--centered); flex-direction:column; align-items:center; justify-content:center; }
    form { background:#fff; padding:1rem; border:1px solid #ccc; border-radius:4px; width:100%; max-width:400px; margin:1rem auto; }
    form input, form textarea, form select {
      width:100%; margin:.5rem 0; padding:.5rem; font-size:1rem; border:1px solid #ccc; border-radius:4px;
    }
    form button { width:100%; padding:.75rem; background:var(--primary); color:#fff; border:none; border-radius:4px; font-size:1rem; cursor:pointer; }
    nav { display:flex; flex-wrap:wrap; background:var(--light); }
    nav button { flex:1 1 auto; padding:.75rem; border:none; background:none; cursor:pointer; transition:.2s; }
    nav button.active, nav button:hover { background:var(--accent); color:#fff; }
    .tab { display:none; }
    .tab.active { display:block; }
    ul { list-style:none; margin:1rem 0; max-width:600px; }
    ul li { background:var(--bg-list); margin:.3rem 0; padding:.6rem; border-radius:4px; display:flex; justify-content:space-between; }
    ul li button { background:red; color:#fff; border:none; border-radius:4px; cursor:pointer; padding:.3rem .6rem; }
  </style>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.0/jszip.min.js"></script>
</head>
<body>
  <header>SIGP-GN Web Secure</header>

  <!-- Écran de login -->
  <main id="login" class="center">
    <form id="loginForm">
      <h2>Connexion</h2>
      <input type="text" id="uName" placeholder="Identifiant" required>
      <input type="password" id="uPass" placeholder="Mot de passe" required>
      <button type="submit">Se connecter</button>
      <p><a href="#" id="toRegister">Créer un compte</a> | <a href="#" id="toReset">Mot de passe oublié</a></p>
    </form>
    <form id="regForm" class="hidden">
      <h2>Créer un compte</h2>
      <input type="text" id="rName" placeholder="Identifiant" required>
      <input type="password" id="rPass" placeholder="Mot de passe" required>
      <input type="password" id="rPass2" placeholder="Confirmer mot de passe" required>
      <button type="submit">Créer</button>
      <p><a href="#" id="toLogin1">Retour</a></p>
    </form>
    <form id="resetForm" class="hidden">
      <h2>Réinitialiser mot de passe</h2>
      <input type="text" id="resetName" placeholder="Identifiant" required>
      <button type="submit">Envoyer lien</button>
      <p><a href="#" id="toLogin2">Retour</a></p>
    </form>
  </main>

  <!-- Interface principale -->
  <nav id="tabs" class="hidden"></nav>
  <main id="content" class="hidden">
    <!-- Les modules seront injectés dynamiquement -->
  </main>

  <script>
  // Stockage des utilisateurs (demo)
  const USERS = JSON.parse(localStorage.getItem('users')||'[]');

  // Modules
  const MODULES = [
    { id:'fpr',    title:'FPR',       fields:['Nom','Prénom','DOB','Motif'],           listKey:'fpr',    listLabels:['nom','prenom','dob','motif']},
    { id:'siv',    title:'SIV',       fields:['Immatriculation','Propriétaire','Infos'], listKey:'siv',    listLabels:['imm','prop','infos']},
    { id:'aud',    title:'Auditions', fields:['Nom','Prénom','DOB','Contenu'],       listKey:'aud',    listLabels:['nom','prenom','dob','content']},
    { id:'garde',  title:'Garde à vue',fields:['Nom','Prénom','DOB','Motif','NoAgent'],listKey:'garde',  listLabels:['nom','prenom','dob','motif','agent']},
    { id:'urg',    title:'Urgences',  fields:['Num','Localisation','Message'],        listKey:'urg',    listLabels:['num','loc','msg']},
    { id:'proc',   title:'Procureur', fields:['Type','Statut'],                      listKey:'proc',   listLabels:['type','statut'], autoRef:true},
    { id:'just',   title:'Justice',   fields:['Réf','Description','Date'],            listKey:'just',   listLabels:['ref','desc','date']}
  ];

  // Authentification
  const loginForm = document.getElementById('loginForm'),
        regForm   = document.getElementById('regForm'),
        resetForm = document.getElementById('resetForm'),
        loginPage = document.getElementById('login'),
        tabsNav   = document.getElementById('tabs'),
        content   = document.getElementById('content');

  document.getElementById('toRegister').onclick = _=> toggleForms('reg');
  document.getElementById('toReset').onclick    = _=> toggleForms('reset');
  document.getElementById('toLogin1').onclick   = _=> toggleForms('login');
  document.getElementById('toLogin2').onclick   = _=> toggleForms('login');

  function toggleForms(f){
    loginForm.parentElement.classList.toggle('hidden', f!=='login');
    regForm.classList.toggle('hidden', f!=='reg');
    resetForm.classList.toggle('hidden', f!=='reset');
  }

  loginForm.onsubmit = e=>{
    e.preventDefault();
    const u=uName.value, p=uPass.value;
    if(USERS.find(x=>x.name===u && x.pass===p)){
      initApp();
    } else alert('Identifiant ou mot de passe invalide');
  };

  regForm.onsubmit = e=>{
    e.preventDefault();
    const u=rName.value, p1=rPass.value, p2=rPass2.value;
    if(p1!==p2){ alert('Les mots de passe ne correspondent pas'); return; }
    USERS.push({name:u,pass:p1});
    localStorage.setItem('users', JSON.stringify(USERS));
    alert('Compte créé. Connecte-toi.');
    toggleForms('login');
  };

  resetForm.onsubmit = e=>{
    e.preventDefault();
    alert('Un lien de réinitialisation a été envoyé (simulation).');
    toggleForms('login');
  };

  // Initialisation de l’app
  function initApp(){
    loginPage.classList.add('hidden');
    tabsNav.classList.remove('hidden');
    content.classList.remove('hidden');
    buildTabs();
    activate('fpr');
  }

  function buildTabs(){
    MODULES.forEach(m=>{
      // onglet
      const btn=document.createElement('button');
      btn.id='tab-'+m.id; btn.textContent=m.title;
      btn.onclick=_=>activate(m.id);
      tabsNav.appendChild(btn);
      // section
      const sec=document.createElement('section');
      sec.id=m.id; sec.className='tab';
      // Formulaire générique
      let html=`<h2>${m.title}</h2><form id="${m.id}Form">`;
      if(m.autoRef) html+=`<input readonly value="10585${Math.floor(1000+Math.random()*9000)}" id="${m.id}Ref"><br>`;
      m.fields.forEach(f=> html+=`<input placeholder="${f}" id="${m.id+f.replace(/\\s/g,'')}" required><br>`);
      html+=`<button>Enregistrer</button></form><ul id="${m.listKey}List"></ul>`;
      sec.innerHTML=html;
      content.appendChild(sec);
    });
    // Export
    const exp=document.createElement('button');
    exp.textContent='Exporter tous (ZIP)';
    exp.onclick=exportAll;
    tabsNav.appendChild(exp);
  }

  function activate(id){
    MODULES.forEach(m=>{
      document.getElementById(m.id).classList.toggle('active', m.id===id);
      document.getElementById('tab-'+m.id).classList.toggle('active', m.id===id);
    });
    renderModule(id);
  }

  function renderModule(id){
    const m=MODULES.find(x=>x.id===id);
    const key=m.listKey;
    const listId=key+'List';
    // setup submit
    const form=document.getElementById(id+'Form');
    form.onsubmit=e=>{
      e.preventDefault();
      const obj={date:new Date().toLocaleString()};
      if(m.autoRef) obj.ref=document.getElementById(id+'Ref').value;
      m.fields.forEach(f=>{
        const prop=f.replace(/\s/g,'').toLowerCase();
        obj[prop]=document.getElementById(id+f.replace(/\s/g,'')).value;
      });
      // phone/motif d'urgence
      saveData(key,obj);
      form.reset();
      renderList(key,listId,obj=>formatItem(m,obj));
    };
    // affichage
    renderList(key,listId,obj=>formatItem(m,obj));
  }

  function saveData(key,data){
    const arr=JSON.parse(localStorage.getItem(key)||'[]');
    arr.push(data);
    localStorage.setItem(key, JSON.stringify(arr));
  }

  function renderList(key,listId,fmt){
    const ul=document.getElementById(listId);
    const arr=JSON.parse(localStorage.getItem(key)||'[]');
    ul.innerHTML=arr.map((o,i)=>`<li>${fmt(o)}<button onclick="deleteItem('${key}',${i})">X</button></li>`).join('');
  }

  function deleteItem(key,i){
    const arr=JSON.parse(localStorage.getItem(key)||'[]');
    arr.splice(i,1);
    localStorage.setItem(key, JSON.stringify(arr));
    renderList(key,key+'List',o=>o.date+' …');  // re-render
  }

  function formatItem(m,obj){
    return m.autoRef
      ? `[${obj.ref}] ${m.fields.map(f=>obj[f.replace(/\s/g,'').toLowerCase()]).join(' – ')} (${obj.date})`
      : `${m.fields.map(f=>obj[f.replace(/\s/g,'').toLowerCase()]).join(' – ')} (${obj.date})`;
  }

  // Export ZIP
  async function exportAll(){
    const zip=new JSZip();
    MODULES.forEach(m=>{
      const arr=JSON.parse(localStorage.getItem(m.listKey)||'[]');
      zip.folder(m.title).file(m.listKey+'.json',JSON.stringify(arr,null,2));
    });
    const blob=await zip.generateAsync({type:'blob'});
    const a=document.createElement('a');
    a.href=URL.createObjectURL(blob);
    a.download='SIGP-GN.zip';
    a.click();
  }
  </script>
</body>
</html>
