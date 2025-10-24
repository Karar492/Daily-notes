<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>حافظة الأشياء اليومية</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box;}
  body{font-family:'Arial',sans-serif;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);min-height:100vh;display:flex;align-items:center;justify-content:center;padding:20px;}
  .language-screen, .container{background:#fff;border-radius:20px;box-shadow:0 15px 35px rgba(0,0,0,0.1);padding:40px;text-align:center;max-width:850px;width:100%;}
  .language-screen select{padding:12px 15px;border-radius:10px;font-size:16px;margin-bottom:20px;}
  .language-screen button{padding:12px 20px;font-size:16px;border:none;border-radius:10px;background:#4facfe;color:white;cursor:pointer;}
  .container{display:none;flex-direction:column;}
  .header{background:linear-gradient(135deg,#4facfe 0%,#00f2fe 100%);color:white;padding:30px 20px;border-radius:15px 15px 0 0;}
  .header h1{font-size:2em;margin-bottom:10px;}
  .header p{opacity:0.9;}
  .add-section{padding:20px;border-bottom:1px solid #e9ecef;}
  .input-group{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:10px;}
  .input-field,.category-select{flex:1;padding:12px;border:2px solid #e9ecef;border-radius:10px;font-size:15px;}
  .input-field:focus,.category-select:focus{border-color:#4facfe;outline:none;}
  .add-btn{width:100%;padding:12px;background:#4facfe;color:white;border:none;border-radius:10px;cursor:pointer;margin-top:10px;}
  .items-section{padding:20px;}
  .category-filter{display:flex;gap:5px;flex-wrap:wrap;margin-bottom:15px;}
  .filter-btn{padding:5px 15px;border-radius:20px;border:1px solid #e9ecef;cursor:pointer;}
  .filter-btn.active{background:#4facfe;color:white;border-color:#4facfe;}
  .items-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(250px,1fr));gap:15px;}
  .item-card{background:#fff;padding:15px;border-radius:10px;box-shadow:0 3px 10px rgba(0,0,0,0.1);position:relative;}
  .item-title{font-weight:bold;margin-bottom:5px;}
  .item-footer{display:flex;justify-content:space-between;font-size:12px;margin-top:10px;}
  .delete-btn{background:#e74c3c;color:white;border:none;padding:5px 10px;border-radius:5px;cursor:pointer;}
  .empty-state{text-align:center;color:#777;padding:40px;}
</style>
</head>
<body>

<div class="language-screen">
  <h2>اختر اللغة / Select Language / Sprache wählen / 言語を選択</h2>
  <select id="languageSelect">
    <option value="ar">العربية</option>
    <option value="en">English</option>
    <option value="de">Deutsch</option>
    <option value="ja">日本語</option>
  </select>
  <br>
  <button onclick="startApp()">ابدأ / Start / Starten / 開始</button>
</div>

<div class="container">
  <div class="header">
    <h1 id="headerTitle"></h1>
    <p id="headerSubtitle"></p>
  </div>
  <div class="add-section">
    <div class="input-group">
      <input type="text" id="itemTitle" class="input-field">
      <select id="itemCategory" class="category-select"></select>
    </div>
    <input type="text" id="itemDescription" class="input-field">
    <button class="add-btn" id="addButton"></button>
  </div>
  <div class="items-section">
    <div class="category-filter" id="categoryFilter"></div>
    <div class="items-grid" id="itemsGrid"></div>
  </div>
</div>

<script>
let items = JSON.parse(localStorage.getItem('dailyItems')) || [];
let currentFilter = 'all';
let currentLang = 'ar';

const translations = {
  ar:{headerTitle:'حافظة الأشياء اليومية',headerSubtitle:'احفظ ملاحظاتك وأفكارك اليومية بسهولة',placeholders:{title:'عنوان الشيء',description:'تفاصيل إضافية'},addButton:'إضافة للحافظة',categories:{personal:'شخصي',work:'عمل',shopping:'تسوق',health:'صحة',other:'أخرى'},filters:{all:'الكل',personal:'شخصي',work:'عمل',shopping:'تسوق',health:'صحة',other:'أخرى'},deleteConfirm:'هل أنت متأكد من حذف هذا العنصر؟',empty:{all:'لا توجد عناصر محفوظة',category:'لا توجد عناصر بهذا التصنيف',add:'ابدأ بإضافة العناصر التي تحتاجها',other:'جرب تصنيفاً آخر أو أضف عنصراً جديداً'}},
  en:{headerTitle:'Daily Notes',headerSubtitle:'Save your daily items easily',placeholders:{title:'Item title',description:'Additional details'},addButton:'Add to Notes',categories:{personal:'Personal',work:'Work',shopping:'Shopping',health:'Health',other:'Other'},filters:{all:'All',personal:'Personal',work:'Work',shopping:'Shopping',health:'Health',other:'Other'},deleteConfirm:'Are you sure you want to delete this item?',empty:{all:'No items saved',category:'No items in this category',add:'Start adding items you need',other:'Try another category or add a new item'}},
  de:{headerTitle:'Tägliche Notizen',headerSubtitle:'Speichern Sie Ihre täglichen Notizen einfach',placeholders:{title:'Titel',description:'Zusätzliche Details'},addButton:'Zur Notizen hinzufügen',categories:{personal:'Persönlich',work:'Arbeit',shopping:'Einkaufen',health:'Gesundheit',other:'Andere'},filters:{all:'Alle',personal:'Persönlich',work:'Arbeit',shopping:'Einkaufen',health:'Gesundheit',other:'Andere'},deleteConfirm:'Möchten Sie diesen Eintrag wirklich löschen?',empty:{all:'Keine Einträge vorhanden',category:'Keine Einträge in dieser Kategorie',add:'Fügen Sie die benötigten Einträge hinzu',other:'Probieren Sie eine andere Kategorie oder fügen Sie einen neuen Eintrag hinzu'}},
  ja:{headerTitle:'日々のメモ',headerSubtitle:'日々のアイテムを簡単に保存',placeholders:{title:'アイテム名',description:'詳細（任意）'},addButton:'メモに追加',categories:{personal:'個人',work:'仕事',shopping:'買い物',health:'健康',other:'その他'},filters:{all:'すべて',personal:'個人',work:'仕事',shopping:'買い物',health:'健康',other:'その他'},deleteConfirm:'この項目を削除してもよろしいですか？',empty:{all:'保存された項目はありません',category:'このカテゴリーには項目がありません',add:'必要な項目を追加してください',other:'別のカテゴリーを試すか、新しい項目を追加してください'}}
};

function startApp(){
  currentLang = document.getElementById('languageSelect').value;
  document.documentElement.lang = currentLang;
  document.documentElement.dir = (currentLang==='ar')?'rtl':'ltr';
  document.querySelector('.language-screen').style.display='none';
  document.querySelector('.container').style.display='flex';
  applyLanguage();
  renderItems();
}

function applyLanguage(){
  const t = translations[currentLang];
  document.getElementById('headerTitle').textContent = t.headerTitle;
  document.getElementById('headerSubtitle').textContent = t.headerSubtitle;
  document.getElementById('itemTitle').placeholder = t.placeholders.title;
  document.getElementById('itemDescription').placeholder = t.placeholders.description;
  document.getElementById('addButton').textContent = t.addButton;

  const catSelect = document.getElementById('itemCategory');
  catSelect.innerHTML = '';
  for(let key in t.categories){
    const opt = document.createElement('option');
    opt.value = key;
    opt.textContent = t.categories[key];
    catSelect.appendChild(opt);
  }

  const filterDiv = document.getElementById('categoryFilter');
  filterDiv.innerHTML='';
  for(let key in t.filters){
    const btn = document.createElement('button');
    btn.className = 'filter-btn'+(key==='all'?' active':'');
    btn.dataset.filter = key;
    btn.textContent = t.filters[key];
    btn.addEventListener('click',()=>{ currentFilter=key; setActiveFilter(); renderItems(); });
    filterDiv.appendChild(btn);
  }
}

function setActiveFilter(){
  document.querySelectorAll('.filter-btn').forEach(btn=>btn.classList.toggle('active',btn.dataset.filter===currentFilter));
}

function addItem(){
  const title = document.getElementById('itemTitle').value.trim();
  const category = document.getElementById('itemCategory').value;
  const description = document.getElementById('itemDescription').value.trim();
  if(!title){ alert(translations[currentLang].deleteConfirm); return; }
  items.push({id:Date.now(),title,category,description,date:new Date().toLocaleDateString()});
  localStorage.setItem('dailyItems',JSON.stringify(items));
  renderItems();
  document.getElementById('itemTitle').value='';
  document.getElementById('itemDescription').value='';
}

function deleteItem(id){
  if(confirm(translations[currentLang].deleteConfirm)){
    items=items.filter(i=>i.id!==id);
    localStorage.setItem('dailyItems',JSON.stringify(items));
    renderItems();
  }
}

function renderItems(){
  const grid=document.getElementById('itemsGrid');
  const t = translations[currentLang];
  const filtered = currentFilter==='all'?items:items.filter(i=>i.category===currentFilter);
  if(filtered.length===0){
    grid.innerHTML=`<div class="empty-state"><p>${currentFilter==='all'?t.empty.all:t.empty.category}</p></div>`;
    return;
  }
  grid.innerHTML = filtered.map(i=>`
    <div class="item-card ${i.category}">
      <div class="item-title">${i.title}</div>
      ${i.description?`<div class="item-description">${i.description}</div>`:''}
      <div class="item-footer">
        <span class="item-category">${t.categories[i.category]}</span>
        <span class="item-date">${i.date}</span>
        <button class="delete-btn" onclick="deleteItem(${i.id})">${currentLang==='ar'?'حذف':'Delete'}</button>
      </div>
    </div>
  `).join('');
}

document.getElementById('itemTitle').addEventListener('keypress',e=>{if(e.key==='Enter') addItem();});
document.getElementById('itemDescription').addEventListener('keypress',e=>{if(e.key==='Enter') addItem();});
</script>

</body>
</html>
