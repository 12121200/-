// Форматирование цены
function formatPriceInput(value) {
    value = value.replace(/[^\d]/g, "");
    return value.replace(/\B(?=(\d{3})+(?!\d))/g, " ");
}

function parseFormattedPrice(value) {
    return Number(value.replace(/\s+/g, ""));
}

function formatPrice(num) {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
}

let users = JSON.parse(localStorage.getItem('users')) || {};
let currentUser = JSON.parse(localStorage.getItem('currentUser')) || null;
let ads = JSON.parse(localStorage.getItem('ads')) || [];
let messages = JSON.parse(localStorage.getItem('messages')) || [];

let cropImage = null, cropScale = 1, cropOffsetX = 0, cropOffsetY = 0, isDragging = false, dragStartX = 0, dragStartY = 0;
let currentDetailImages = [], currentDetailIndex = 0;

let viewerImages = [], viewerIndex = 0, viewerScale = 1, viewerDragging = false, viewerStartX = 0, viewerStartY = 0, viewerOffsetX = 0, viewerOffsetY = 0;

let currentDialogAdId = null, deleteAdId = null, deleteDialogAdId = null;

let tempImages = [];

let viewMode = 'list', sortMode = 'newest', priceSortDirection = 'asc';

const cities = [
    "Москва","Санкт-Петербург","Новосибирск","Екатеринбург","Казань","Нижний Новгород","Челябинск","Самара","Омск",
    "Ростов-на-Дону","Уфа","Красноярск","Пермь","Волгоград","Воронеж","Саратов","Краснодар","Тюмень","Иркутск",
    "Хабаровск","Ярославль","Тольятти","Барнаул","Ижевск","Ульяновск","Владивосток","Кемерово","Томск","Курск",
    "Белгород","Липецк","Брянск","Рязань","Тверь","Калуга","Сочи"
];

function saveUsers(){localStorage.setItem('users',JSON.stringify(users));}
function saveCurrentUser(){localStorage.setItem('currentUser',JSON.stringify(currentUser));}
function saveAds(){localStorage.setItem('ads',JSON.stringify(ads));}
function saveMessages(){localStorage.setItem('messages',JSON.stringify(messages));}

/* theme */
function applyTheme(theme){
    document.body.setAttribute('data-theme',theme);
    localStorage.setItem('themeMode',theme);
    const btn=document.getElementById('themeToggle');
    if(!btn) return;
    btn.classList.add('active');
    if(theme==='light')btn.textContent='Темная';
    else btn.textContent='Светлая';
}

function toggleLightDark(){
    const current=document.body.getAttribute('data-theme');
    if(current==='light')applyTheme('dark');
    else applyTheme('light');
}

/* navigation */
function navigate(page){
    document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
    const el = document.getElementById(page+'-page');
    if(el) el.classList.add('active');
    if(page==='main')renderAds();
    if(page==='profile')renderProfile();
    if(page==='messages')renderDialogs();
}

function toggleFilters(){
    const panel=document.getElementById('filtersPanel');
    if(!panel) return;
    panel.style.display=panel.style.display==='block'?'none':'block';
}

/* auth */
function showLoginModal(){document.getElementById('loginModal').style.display='flex';}
function showRegisterModal(){document.getElementById('registerModal').style.display='flex';}

function isValidEmail(email){
    const allowed=['@gmail.com','@mail.ru','@yandex.ru','@outlook.com','@hotmail.com'];
    return allowed.some(d=>email.endsWith(d));
}

function isValidPhone(phone){
    const p=phone.replace(/\s+/g,'');
    if(!(p.startsWith('+7')||p.startsWith('8')))return false;
    const digits=p.replace(/[^\d]/g,'');
    return digits.length>=11&&digits.length<=12;
}

function submitRegister(){
    const login=document.getElementById('regLogin').value.trim();
    const email=document.getElementById('regEmail').value.trim();
    const phone=document.getElementById('regPhone').value.trim();
    const password=document.getElementById('regPassword').value.trim();
    if(!login||!password)return alert("Введите логин и пароль");
    if(users[login])return alert("Логин уже существует");
    if(email&&!isValidEmail(email))return alert("Неверный формат email или домен");
    if(phone&&!isValidPhone(phone))return alert("Телефон должен начинаться с +7 или 8 и содержать достаточно цифр");
    users[login]={login,password,displayName:login,avatar:null,email:email||null,phone:phone||null};
    saveUsers();
    alert("Аккаунт создан. Теперь войдите.");
    document.getElementById('registerModal').style.display='none';
}

function findUserByIdentifier(id){
    id=id.trim();if(!id)return null;
    if(users[id])return users[id];
    for(const key in users){
        const u=users[key];
        if(u.email&&u.email===id)return u;
        if(u.phone&&u.phone===id)return u;
    }
    return null;
}

function submitLogin(){
    const identifier=document.getElementById('loginIdentifier').value.trim();
    const password=document.getElementById('loginPassword').value.trim();
    const user=findUserByIdentifier(identifier);
    if(!user)return alert("Аккаунт не найден");
    if(user.password!==password)return alert("Неверный пароль");
    currentUser=user;
    saveCurrentUser();
    document.getElementById('loginModal').style.display='none';
    document.getElementById('loginBtn').style.display='none';
    document.getElementById('registerBtn').style.display='none';
    document.getElementById('userPanel').style.display='flex';
    document.getElementById('usernameDisplay').textContent=currentUser.displayName;
    document.getElementById('headerAvatar').src=currentUser.avatar||"https://via.placeholder.com/40?text=👤";
    navigate('main');
}

function openLogoutModal(){document.getElementById('logoutModal').style.display='flex';}
function closeLogoutModal(){document.getElementById('logoutModal').style.display='none';}

function confirmLogout(){
    currentUser=null;
    localStorage.removeItem('currentUser');
    document.getElementById('loginBtn').style.display='inline-block';
    document.getElementById('registerBtn').style.display='inline-block';
    document.getElementById('userPanel').style.display='none';
    document.getElementById('logoutModal').style.display='none';
    navigate('main');
}

/* view/sort */
function setViewMode(mode){
    viewMode=mode;
    document.getElementById('btnViewList').classList.toggle('filters-toggle-btn-active',mode==='list');
    document.getElementById('btnViewGrid').classList.toggle('filters-toggle-btn-active',mode==='grid');
    renderAds();
}

function setSortMode(mode){
    sortMode=mode;
    document.getElementById('btnSortNewest').classList.toggle('filters-toggle-btn-active',mode==='newest');
    renderAds();
}

function togglePriceSort(){
    if(priceSortDirection==='asc')priceSortDirection='desc';
    else priceSortDirection='asc';
    document.getElementById('priceSortLabel').textContent=priceSortDirection==='asc'?'Цена ↑':'Цена ↓';
    sortMode='price';
    renderAds();
}

/* ads render */
function renderAds(list=null){
    const grid=document.getElementById('adsGrid');
    if(!grid) return;
    let data=list||ads.slice();

    if(sortMode==='price'){
        if(priceSortDirection==='asc')data.sort((a,b)=>a.price-b.price);
        else data.sort((a,b)=>b.price-a.price);
    }
    if(sortMode==='newest')data.sort((a,b)=>b.id-a.id);

    grid.style.gridTemplateColumns=viewMode==='grid'?'repeat(4,minmax(0,1fr))':'repeat(1,minmax(0,1fr))';

    grid.innerHTML=data.map(ad=>{
        const firstImg=(ad.images&&ad.images.length)?ad.images[ad.mainImageIndex||0]:"https://via.placeholder.com/600x400?text=Фото";
        const cityText=ad.city||'';
        const conditionText=ad.condition==='new'?'Новое':(ad.condition==='used'?'Б/у':'');
        const fullDesc=ad.description||'';
        const shortDesc=fullDesc.length>200?fullDesc.slice(0,200)+'…':fullDesc;
        const sellerName=ad.authorName||ad.authorLogin||'';

        if(viewMode==='list'){
            return `
<div class="card card-list" onclick="openAdDetail(${ad.id})">
  <div class="card-img"><img src="${firstImg}" alt=""></div>
  <div class="card-list-body">
    <div class="card-list-row">
      <div class="card-list-main">
        <div class="card-list-desc">${shortDesc}</div>
        <div class="card-meta">${cityText}</div>
        <div class="card-meta">${conditionText}</div>
      </div>
      <div class="card-list-side">
        <div class="card-list-title">${ad.title}</div>
        <div class="card-list-price">${formatPrice(ad.price)} ₽</div>
        <div class="card-list-author">${sellerName}</div>
        <button class="btn btn-primary" style="margin-top:6px;font-size:14px;" onclick="event.stopPropagation();openChatFromAd(${ad.id})">Написать</button>
      </div>
    </div>
  </div>
</div>`;
        } else {
            return `
<div class="card" onclick="openAdDetail(${ad.id})">
  <div class="card-img">
    <img src="${firstImg}" alt="">
  </div>
  <div class="card-body" style="padding:12px;">
    <div class="card-title" style="font-size:16px;font-weight:600;margin-bottom:6px;">
      ${ad.title}
    </div>
    <div class="card-price" style="font-size:15px;font-weight:700;color:#00aa5b;margin-top:4px;">
      ${formatPrice(ad.price)} ₽
    </div>
  </div>
</div>`;
        }
    }).join('');
}

/* detail */
function openAdDetail(id){
    const ad=ads.find(a=>a.id===id);
    if(!ad)return;

    const images=ad.images&&ad.images.length?ad.images:["https://via.placeholder.com/600x400?text=Фото"];
    currentDetailImages=images;
    currentDetailIndex=ad.mainImageIndex||0;
    const mainImg=images[currentDetailIndex];

    const sellerName=ad.authorName||ad.authorLogin||'Продавец';
    const cityText=ad.city||'';
    const conditionText=ad.condition==='new'?'Новое':(ad.condition==='used'?'Б/у':'');

    let sellerBlock='';
    if(!currentUser){
        sellerBlock=`
<div class="detail-seller">
  <div class="detail-seller-row">
    <div class="detail-seller-main">
      <div><strong>Продавец:</strong> ${sellerName}</div>
      <div style="margin-top:4px;color:#6b7280;">${cityText}</div>
      <div style="margin-top:4px;color:#6b7280;">${conditionText}</div>
    </div>
    <div class="detail-seller-side">
      <button class="btn btn-primary" onclick="showLoginModal()">Войти, чтобы написать</button>
    </div>
  </div>
</div>`;
    } else {
        const canWrite=currentUser.login!==ad.authorLogin;
        sellerBlock=`
<div class="detail-seller">
  <div class="detail-seller-row">
    <div class="detail-seller-main">
      <div><strong>Продавец:</strong> ${sellerName}</div>
      <div style="margin-top:4px;color:#6b7280;">${cityText}</div>
      <div style="margin-top:4px;color:#6b7280;">${conditionText}</div>
    </div>
    <div class="detail-seller-side">
      ${canWrite
        ? `<button class="btn btn-primary" onclick="openChatFromAd(${ad.id})">Написать</button>`
        : `<span style="color:#9ca3af;">Это ваше объявление</span>`}
    </div>
  </div>
</div>`;
    }

    document.getElementById('adDetailContent').innerHTML=`
<div class="detail-image-wrapper">
  <button class="detail-arrow detail-arrow-left" onclick="prevDetailImage()">‹</button>
  <img class="detail-main-img" id="detailMainImg" src="${mainImg}" onclick="openImageViewerFromDetail(${ad.id})">
  <button class="detail-arrow detail-arrow-right" onclick="nextDetailImage()">›</button>
</div>
<div class="detail-gallery">
  ${images.map((img,i)=>`<div class="detail-thumb" onclick="setDetailImageIndex(${i})" style="background-image:url('${img}')"></div>`).join('')}
</div>
<div class="detail-title">${ad.title}</div>
<div class="detail-price">${formatPrice(ad.price)} ₽</div>
<div class="detail-desc">${ad.description||''}</div>
${sellerBlock}`;
    navigate('ad-detail');
}

function setDetailImageIndex(i){
    currentDetailIndex=i;
    const img=document.getElementById('detailMainImg');
    if(img&&currentDetailImages[i])img.src=currentDetailImages[i];
}

function prevDetailImage(){
    if(!currentDetailImages.length)return;
    currentDetailIndex=(currentDetailIndex-1+currentDetailImages.length)%currentDetailImages.length;
    setDetailImageIndex(currentDetailIndex);
}

function nextDetailImage(){
    if(!currentDetailImages.length)return;
    currentDetailIndex=(currentDetailIndex+1)%currentDetailImages.length;
    setDetailImageIndex(currentDetailIndex);
}

/* image viewer */
function openImageViewerFromDetail(adId){
    const ad=ads.find(a=>a.id===adId);
    if(!ad)return;
    viewerImages=ad.images&&ad.images.length?ad.images:["https://via.placeholder.com/600x400?text=Фото"];
    viewerIndex=currentDetailIndex||0;
    viewerScale=1;viewerOffsetX=0;viewerOffsetY=0;
    const img=document.getElementById('imageViewerImg');
    img.src=viewerImages[viewerIndex];
    img.style.transform=`translate(0px,0px) scale(1)`;
    document.getElementById('imageViewerZoom').value=1;
    document.getElementById('imageViewerModal').style.display='flex';
}

function viewerPrev(){
    if(!viewerImages.length)return;
    viewerIndex=(viewerIndex-1+viewerImages.length)%viewerImages.length;
    const img=document.getElementById('imageViewerImg');
    img.src=viewerImages[viewerIndex];
    resetViewerTransform();
}

function viewerNext(){
    if(!viewerImages.length)return;
    viewerIndex=(viewerIndex+1)%viewerImages.length;
    const img=document.getElementById('imageViewerImg');
    img.src=viewerImages[viewerIndex];
    resetViewerTransform();
}

function closeImageViewer(){
    document.getElementById('imageViewerModal').style.display='none';
}

function resetViewerTransform(){
    viewerScale=1;viewerOffsetX=0;viewerOffsetY=0;
    const img=document.getElementById('imageViewerImg');
    img.style.transform=`translate(0px,0px) scale(1)`;
    document.getElementById('imageViewerZoom').value=1;
}

const viewerImgEl=document.getElementById('imageViewerImg');
if(viewerImgEl){
    viewerImgEl.addEventListener('mousedown',e=>{
        viewerDragging=true;
        viewerStartX=e.clientX;viewerStartY=e.clientY;
        viewerImgEl.style.cursor='grabbing';
    });
    window.addEventListener('mouseup',()=>{
        viewerDragging=false;
        viewerImgEl.style.cursor='grab';
    });
    window.addEventListener('mousemove',e=>{
        if(!viewerDragging)return;
        const dx=e.clientX-viewerStartX;
        const dy=e.clientY-viewerStartY;
        viewerStartX=e.clientX;viewerStartY=e.clientY;
        viewerOffsetX+=dx;viewerOffsetY+=dy;
        viewerImgEl.style.transform=`translate(${viewerOffsetX}px,${viewerOffsetY}px) scale(${viewerScale})`;
    });
    viewerImgEl.addEventListener('wheel',e=>{
        e.preventDefault();
        const delta=e.deltaY<0?1.05:0.95;
        viewerScale*=delta;
        if(viewerScale<0.5)viewerScale=0.5;
        if(viewerScale>3)viewerScale=3;
        document.getElementById('imageViewerZoom').value=viewerScale;
        viewerImgEl.style.transform=`translate(${viewerOffsetX}px,${viewerOffsetY}px) scale(${viewerScale})`;
    });
    document.getElementById('imageViewerZoom').addEventListener('input',e=>{
        viewerScale=Number(e.target.value);
        viewerImgEl.style.transform=`translate(${viewerOffsetX}px,${viewerOffsetY}px) scale(${viewerScale})`;
    });
}

/* create ads */
function prepareImagesPreview(){
    const input=document.getElementById('adImages');
    const preview=document.getElementById('imagesPreview');
    preview.innerHTML='';tempImages=[];
    if(!input.files.length)return;
    for(let i=0;i<input.files.length;i++){
        const url=URL.createObjectURL(input.files[i]);
        tempImages.push(url);
    }
    tempImages.forEach((url,index)=>{
        preview.innerHTML+=`
<div style="width:70px;height:70px;border-radius:8px;border:2px solid #e5e7eb;overflow:hidden;position:relative;cursor:pointer;" onclick="setMainImage(${index})">
  <img src="${url}" style="width:100%;height:100%;object-fit:cover;">
  <div id="mainMark${index}" style="position:absolute;bottom:2px;left:2px;right:2px;background:rgba(0,0,0,0.5);color:#fff;font-size:11px;text-align:center;border-radius:4px;display:${index===0?'block':'none'};">Главная</div>
</div>`;
    });
    document.getElementById('mainImageIndex').value=0;
}

function setMainImage(index){
    document.getElementById('mainImageIndex').value=index;
    tempImages.forEach((_,i)=>{
        const mark=document.getElementById('mainMark'+i);
        if(mark)mark.style.display=i===index?'block':'none';
    });
}

async function saveAd(e){
    e.preventDefault();
    if(!currentUser) return alert("Войдите в аккаунт");

    const fileInput = document.getElementById('adImages');
    const editId = document.getElementById('editId').value;
    let title = document.getElementById('adTitle').value.trim();
    let desc = document.getElementById('adDesc').value.trim();
    const price = parseFormattedPrice(document.getElementById('adPrice').value);
    const cityInput = document.getElementById('adCityInput').value.trim();
    const condition = document.getElementById('adCondition').value;
    const mainIndex = Number(document.getElementById('mainImageIndex').value || 0);

    if (!cityInput) return alert("Выберите город из списка");
    if (!cities.includes(cityInput)) return alert("Выберите город из списка, а не вводите произвольный текст");
    if (!price || price <= 0) return alert("Введите корректную цену");
    if (String(price).length > 10) return alert("Цена не может быть длиннее 10 цифр");
    if (desc.length > 330) desc = desc.slice(0, 330);

    let images = [];

    // Редактирование
    if (editId) {
        const ad = ads.find(a => a.id == editId);
        if (!ad || ad.authorLogin !== currentUser.login)
            return alert("Это не ваше объявление");

        if (!fileInput.files.length) {
            images = ad.images ? ad.images.slice() : [];
        } else {
            const formData = new FormData();
            for (let f of fileInput.files) formData.append('images', f);
            const uploadRes = await fetch('/api/upload', {
                method: 'POST',
                body: formData
            });
            const uploadData = await uploadRes.json();
            images = uploadData.files || [];
        }

        ad.title = title;
        ad.description = desc;
        ad.price = price;
        ad.city = cityInput;
        ad.condition = condition;
        ad.images = images;
        ad.mainImageIndex = images.length
            ? (mainIndex >= 0 && mainIndex < images.length ? mainIndex : 0)
            : 0;

        saveAds();
        renderAds();
        navigate('main');
        alert("Объявление обновлено");
        return;
    }

    // Создание
    if (!fileInput.files.length)
        return alert("Добавьте хотя бы одну фотографию");

    const formData = new FormData();
    for (let f of fileInput.files) formData.append('images', f);
    const uploadRes = await fetch('/api/upload', {
        method: 'POST',
        body: formData
    });
    const uploadData = await uploadRes.json();
    images = uploadData.files || [];

    ads.unshift({
        id: Date.now(),
        title,
        description: desc,
        price,
        city: cityInput,
        condition,
        authorLogin: currentUser.login,
        authorName: currentUser.displayName,
        images,
        mainImageIndex: images.length
            ? (mainIndex >= 0 && mainIndex < images.length ? mainIndex : 0)
            : 0
    });

    saveAds();
    renderAds();
    navigate('main');
    alert("Объявление создано");

    document.getElementById('formTitle').textContent = "Новое объявление";
    document.getElementById('editId').value = "";
    e.target.reset();
    document.getElementById('imagesPreview').innerHTML = '';
    tempImages = images.slice();
}

/* profile */
function renderProfile(){
    if(!currentUser)return;
    document.getElementById('profileName').textContent=currentUser.displayName;
    const avatarEl=document.getElementById('avatarPreview');
    if(currentUser.avatar&&currentUser.avatar.startsWith('data:image')){
        avatarEl.style.backgroundImage=`url('${currentUser.avatar}')`;
        avatarEl.textContent='';
    }else{
        avatarEl.style.backgroundImage=`url('${currentUser.avatar||"https://via.placeholder.com/120?text=👤"}')`;
        avatarEl.textContent='';
    }

    document.getElementById('newDisplayName').value=currentUser.displayName||'';
    document.getElementById('emailMask').textContent=currentUser.email?"Нажмите, чтобы показать email":"Email не указан";
    document.getElementById('emailValue').textContent=currentUser.email||'';
    document.getElementById('phoneMask').textContent=currentUser.phone?"Нажмите, чтобы показать телефон":"Телефон не указан";
    document.getElementById('phoneValue').textContent=currentUser.phone||'';

    const myAds=ads.filter(a=>a.authorLogin===currentUser.login);
    document.getElementById('profileAds').innerHTML=myAds.map(ad=>{
        const thumb=ad.images&&ad.images.length?ad.images[ad.mainImageIndex||0]:"https://via.placeholder.com/80x80?text=Фото";
        return `
<div>
  <div class="profile-ad-thumb" style="background-image:url('${thumb}')"></div>
  <div style="flex:1;">
    <div style="font-weight:600;font-size:15px;">${ad.title}</div>
    <div style="color:#00aa5b;font-weight:700;font-size:16px;">${formatPrice(ad.price)} ₽</div>
    <div style="margin-top:4px;color:#6b7280;font-size:14px;">${ad.city||''}</div>
    <div style="margin-top:6px;">
      <button class="btn btn-primary" onclick="editAd(${ad.id})">Редактировать</button>
      <button class="btn btn-danger" onclick="openDeleteModal(${ad.id})">Удалить</button>
    </div>
  </div>
</div>`;
    }).join('');
}

function toggleHiddenField(type){
    const maskEl=document.getElementById(type==='email'?'emailMask':'phoneMask');
    const valEl=document.getElementById(type==='email'?'emailValue':'phoneValue');
    if(!valEl.textContent)return;
    if(valEl.style.display==='inline'){
        valEl.style.display='none';
        maskEl.style.display='inline';
    }else{
        valEl.style.display='inline';
        maskEl.style.display='none';
    }
}

// эмодзи-аватарка отключена
function confirmEmojiAvatar(emoji){
    alert("Установка эмодзи как аватарки отключена.");
}
function setEmojiAvatar(emoji){
    // пусто
}

function editAd(id){
    const ad=ads.find(a=>a.id===id);
    if(!ad||ad.authorLogin!==currentUser.login)return alert("Это не ваше объявление");

    document.getElementById('editId').value=id;
    document.getElementById('adTitle').value=ad.title;
    document.getElementById('adDesc').value=ad.description;
    document.getElementById('adPrice').value=formatPriceInput(String(ad.price));
    document.getElementById('adCityInput').value=ad.city||'';
    document.getElementById('adCondition').value=ad.condition||'';
    document.getElementById('formTitle').textContent="Редактировать объявление";

    tempImages=ad.images?ad.images.slice():[];
    const preview=document.getElementById('imagesPreview');
    preview.innerHTML='';
    tempImages.forEach((url,index)=>{
        preview.innerHTML+=`
<div style="width:70px;height:70px;border-radius:8px;border:2px solid #e5e7eb;overflow:hidden;position:relative;cursor:pointer;" onclick="setMainImage(${index})">
  <img src="${url}" style="width:100%;height:100%;object-fit:cover;">
  <div id="mainMark${index}" style="position:absolute;bottom:2px;left:2px;right:2px;background:rgba(0,0,0,0.5);color:#fff;font-size:11px;text-align:center;border-radius:4px;display:${index===(ad.mainImageIndex||0)?'block':'none'};">Главная</div>
</div>`;
    });
    document.getElementById('mainImageIndex').value=ad.mainImageIndex||0;
    navigate('create');
}

/* delete ad */
function openDeleteModal(id){
    deleteAdId=id;
    document.getElementById('deleteModal').style.display='flex';
}

function closeDeleteModal(){
    deleteAdId=null;
    document.getElementById('deleteModal').style.display='none';
}

function confirmDelete(){
    if(!deleteAdId)return;
    const ad=ads.find(a=>a.id===deleteAdId);
    if(!ad||ad.authorLogin!==currentUser.login){
        alert("Это не ваше объявление");
        closeDeleteModal();
        return;
    }
    ads=ads.filter(a=>a.id!==deleteAdId);
    saveAds();
    renderAds();
    renderProfile();
    closeDeleteModal();
}

/* avatar crop */
function changeAvatar(){
    if(!currentUser)return alert("Войдите в аккаунт");
    const input=document.createElement('input');
    input.type='file';input.accept='image/*';
    input.onchange=function(){
        if(!this.files[0])return;
        cropImage=new Image();
        cropImage.src=URL.createObjectURL(this.files[0]);
        cropImage.onload=()=>{
            const canvas=document.getElementById('cropCanvas');
            cropScale=1;
            cropOffsetX=(canvas.width-cropImage.width*cropScale)/2;
            cropOffsetY=(canvas.height-cropImage.height*cropScale)/2;
            drawCrop();
            document.getElementById('cropModal').style.display='flex';
            document.getElementById('cropSlider').value=cropScale;
        };
    };
    input.click();
}

function drawCrop(){
    if(!cropImage)return;
    const canvas=document.getElementById('cropCanvas');
    const ctx=canvas.getContext('2d');
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.save();
    ctx.beginPath();
    ctx.arc(canvas.width/2,canvas.height/2,canvas.width/2,0,Math.PI*2);
    ctx.clip();
    ctx.drawImage(
        cropImage,0,0,cropImage.width,cropImage.height,
        cropOffsetX,cropOffsetY,
        cropImage.width*cropScale,
        cropImage.height*cropScale
    );
    ctx.restore();
}

const cropCanvas=document.getElementById('cropCanvas');
if(cropCanvas){
    cropCanvas.addEventListener('mousedown',e=>{
        if(!cropImage)return;
        isDragging=true;
        dragStartX=e.clientX;dragStartY=e.clientY;
        cropCanvas.style.cursor='grabbing';
    });
    window.addEventListener('mouseup',()=>{
        isDragging=false;
        cropCanvas.style.cursor='grab';
    });
    window.addEventListener('mousemove',e=>{
        if(!isDragging||!cropImage)return;
        const dx=e.clientX-dragStartX;
        const dy=e.clientY-dragStartY;
        dragStartX=e.clientX;dragStartY=e.clientY;
        cropOffsetX+=dx;cropOffsetY+=dy;
        const canvas=cropCanvas;
        const maxX=canvas.width*2,maxY=canvas.height*2;
        if(cropOffsetX>maxX)cropOffsetX=maxX;
        if(cropOffsetX<-maxX)cropOffsetX=-maxX;
        if(cropOffsetY>maxY)cropOffsetY=maxY;
        if(cropOffsetY<-maxY)cropOffsetY=-maxY;
        drawCrop();
    });
    cropCanvas.addEventListener('wheel',e=>{
        if(!cropImage)return;
        e.preventDefault();
        const oldScale=cropScale;
        const delta=e.deltaY<0?1.05:0.95;
        cropScale*=delta;
        const canvas=cropCanvas;
        const cx=canvas.width/2,cy=canvas.height/2;
        cropOffsetX=cx-(cx-cropOffsetX)*(cropScale/oldScale);
        cropOffsetY=cy-(cy-cropOffsetY)*(cropScale/oldScale);
        drawCrop();
        document.getElementById('cropSlider').value=cropScale;
    });
    document.getElementById('cropSlider').addEventListener('input',e=>{
        if(!cropImage)return;
        const oldScale=cropScale;
        cropScale=Number(e.target.value);
        const canvas=cropCanvas;
        const cx=canvas.width/2,cy=canvas.height/2;
        cropOffsetX=cx-(cx-cropOffsetX)*(cropScale/oldScale);
        cropOffsetY=cy-(cy-cropOffsetY)*(cropScale/oldScale);
        drawCrop();
    });
}

function applyCrop(){
    const canvas=document.getElementById('cropCanvas');
    const url=canvas.toDataURL('image/png');
    currentUser.avatar=url;
    users[currentUser.login]=currentUser;
    saveUsers();saveCurrentUser();
    document.getElementById('avatarPreview').style.backgroundImage=`url('${url}')`;
    document.getElementById('avatarPreview').textContent='';
    document.getElementById('headerAvatar').src=url;
    document.getElementById('cropModal').style.display='none';
}

/* profile save */
function saveProfileChanges(){
    if(!currentUser)return alert("Войдите в аккаунт");
    const newName=document.getElementById('newDisplayName').value.trim();
    const email=document.getElementById('emailValue').textContent.trim();
    const phone=document.getElementById('phoneValue').textContent.trim();
    if(email&&!isValidEmail(email))return alert("Неверный формат email или домен");
    if(phone&&!isValidPhone(phone))return alert("Телефон должен начинаться с +7 или 8 и содержать достаточно цифр");
    if(newName)currentUser.displayName=newName;
    currentUser.email=email||null;
    currentUser.phone=phone||null;
    users[currentUser.login]=currentUser;
    saveUsers();saveCurrentUser();
    document.getElementById('profileName').textContent=currentUser.displayName;
    document.getElementById('usernameDisplay').textContent=currentUser.displayName;
    ads=ads.map(ad=>{
        if(ad.authorLogin===currentUser.login){
            return {...ad,authorName:currentUser.displayName};
        }
        return ad;
    });
    saveAds();
    renderAds();
    renderProfile();
    alert("Изменения профиля сохранены");
}

/* search */
function searchAdsAdvanced(){
    const qTitle=document.getElementById('searchInput').value.toLowerCase().trim();
    const title=document.getElementById('filterTitle').value.toLowerCase().trim();
    const cityInput=document.getElementById('filterCityInput').value.toLowerCase().trim();
    const condition=document.getElementById('filterCondition').value;
    const priceFrom=Number(document.getElementById('filterPriceFrom').value||0);
    const priceTo=Number(document.getElementById('filterPriceTo').value||0);

    let results=ads.filter(ad=>{
        const t=(ad.title||'').toLowerCase();
        const d=(ad.description||'').toLowerCase();
        const c=(ad.city||'').toLowerCase();
        const cond=ad.condition||'';

        if(qTitle&&!t.includes(qTitle)&&!d.includes(qTitle))return false;
        if(title&&!t.includes(title))return false;
        if(cityInput&&c!==cityInput)return false;
        if(condition&&cond!==condition)return false;
        if(priceFrom&&ad.price<priceFrom)return false;
        if(priceTo&&ad.price>priceTo)return false;
        return true;
    });

    renderAds(results);
}

/* city select */
function filterCityList(inputId,dropdownId){
    const input=document.getElementById(inputId);
    const dropdown=document.getElementById(dropdownId);
    const value=input.value.toLowerCase();
    const filtered=cities.filter(c=>c.toLowerCase().includes(value));
    if(!filtered.length){
        dropdown.style.display='none';
        dropdown.innerHTML='';
        return;
    }
    dropdown.style.display='block';
    dropdown.innerHTML=filtered.map(c=>`<div class="city-select-item" onclick="selectCity('${inputId}','${dropdownId}','${c}')">${c}</div>`).join('');
}

function selectCity(inputId,dropdownId,city){
    const input=document.getElementById(inputId);
    const dropdown=document.getElementById(dropdownId);
    input.value=city;
    dropdown.style.display='none';
}

/* chat/messages */
function openChatFromAd(adId){
    if(!currentUser){
        alert("Чтобы написать продавцу, нужно войти или зарегистрироваться.");
        showLoginModal();
        return;
    }
    const ad=ads.find(a=>a.id===adId);
    if(!ad)return;
    currentDialogAdId=adId;
    navigate('messages');
    renderDialogs();
    openDialog(adId);
}

function renderDialogs(){
    const list=document.getElementById('dialogsList');
    const chatArea=document.getElementById('chatArea');
    if(!currentUser){
        list.innerHTML='<div style="color:#9ca3af;font-size:13px;">Чтобы видеть диалоги, войдите в аккаунт.</div>';
        chatArea.innerHTML='<div style="color:#9ca3af;font-size:13px;">Нет выбранного диалога.</div>';
        return;
    }

    const myDialogsMap={};
    messages.forEach(m=>{
        if(m.sellerLogin===currentUser.login||m.buyerLogin===currentUser.login){
            const key=m.adId+'|'+m.sellerLogin+'|'+m.buyerLogin;
            if(!myDialogsMap[key])myDialogsMap[key]=[];
            myDialogsMap[key].push(m);
        }
    });

    const dialogKeys=Object.keys(myDialogsMap);
    if(!dialogKeys.length){
        list.innerHTML='<div style="color:#9ca3af;font-size:13px;">Диалогов пока нет.</div>';
        chatArea.innerHTML='<div style="color:#9ca3af;font-size:13px;">Выберите диалог слева, чтобы открыть чат.</div>';
        return;
    }

    list.innerHTML=dialogKeys.map(key=>{
        const msgs=myDialogsMap[key];
        const first=msgs[0];
        const ad=ads.find(a=>a.id===first.adId);
        const thumb=(ad&&ad.images&&ad.images.length)?ad.images[ad.mainImageIndex||0]:"https://via.placeholder.com/80x80?text=Фото";
        const title=ad?ad.title:'Объявление';
        const lastMsg=msgs[msgs.length-1];
        const otherUser = currentUser.login===first.sellerLogin ? first.buyerLogin : first.sellerLogin;
        return `
<div class="dialog-item" onclick="openDialog(${first.adId})">
  <div class="dialog-thumb" style="background-image:url('${thumb}')"></div>
  <div>
    <div class="dialog-info-title">${title}</div>
    <div class="dialog-info-meta">Собеседник: ${otherUser}</div>
    <div class="dialog-info-meta">Последнее: ${lastMsg.text}</div>
    <div class="dialog-delete-btn" onclick="event.stopPropagation();openDeleteDialogModal(${first.adId})">Удалить диалог</div>
  </div>
</div>`;
    }).join('');
}

function openDialog(adId){
    currentDialogAdId=adId;
    const chatArea=document.getElementById('chatArea');
    const ad=ads.find(a=>a.id===adId);
    if(!ad){
        chatArea.innerHTML='<div style="color:#9ca3af;font-size:13px;">Объявление не найдено.</div>';
        return;
    }

    const dialogMessages=messages.filter(m=>m.adId===adId && (m.sellerLogin===currentUser.login || m.buyerLogin===currentUser.login));
    const thumb=(ad.images&&ad.images.length)?ad.images[ad.mainImageIndex||0]:"https://via.placeholder.com/80x80?text=Фото";
    const sellerName=ad.authorName||ad.authorLogin||'Продавец';

    chatArea.innerHTML=`
<div class="chat-header">
  <div class="chat-header-img" style="background-image:url('${thumb}')"></div>
  <div>
    <div class="chat-header-title">${ad.title}</div>
    <div class="chat-header-meta">${sellerName}, ${ad.city||''}</div>
  </div>
</div>
<div id="chatMessages" class="chat-messages">
  ${dialogMessages.map(m=>`
    <div class="chat-message-row">
      <span class="chat-message-from">${m.from===currentUser.login?'Вы':m.from}:</span>
      <span>${m.text}</span>
    </div>`).join('')}
</div>
<div style="margin-top:8px;">
  <input id="chatInput" class="chat-input" placeholder="Напишите сообщение..." onkeyup="if(event.key==='Enter') sendMessage()">
  <button class="btn btn-primary" style="margin-top:6px;" onclick="sendMessage()">Отправить</button>
</div>`;
}

function sendMessage(){
    if(!currentUser||!currentDialogAdId)return;
    const input=document.getElementById('chatInput');
    if(!input)return;
    const text=input.value.trim();
    if(!text)return;
    const ad=ads.find(a=>a.id===currentDialogAdId);
    if(!ad)return;

    const sellerLogin=ad.authorLogin;
    const buyerLogin=currentUser.login===sellerLogin ? null : currentUser.login;
    const from=currentUser.login;

    messages.push({
        id:Date.now(),
        adId:currentDialogAdId,
        sellerLogin,
        buyerLogin:buyerLogin||'buyer',
        from,
        text
    });
    saveMessages();
    input.value='';
    openDialog(currentDialogAdId);
}

function openDeleteDialogModal(adId){
    deleteDialogAdId=adId;
    document.getElementById('deleteDialogModal').style.display='flex';
}

function closeDeleteDialogModal(){
    deleteDialogAdId=null;
    document.getElementById('deleteDialogModal').style.display='none';
}

function confirmDeleteDialog(){
    if(!deleteDialogAdId)return;
    messages=messages.filter(m=>m.adId!==deleteDialogAdId || (m.sellerLogin!==currentUser.login && m.buyerLogin!==currentUser.login));
    saveMessages();
    renderDialogs();
    document.getElementById('chatArea').innerHTML='<div style="color:#9ca3af;font-size:13px;">Выберите диалог слева, чтобы открыть чат.</div>';
    closeDeleteDialogModal();
}

/* init */
window.addEventListener('DOMContentLoaded',()=>{
    const savedTheme = localStorage.getItem('themeMode') || 'light';
    applyTheme(savedTheme);

    if(currentUser){
        document.getElementById('loginBtn').style.display='none';
        document.getElementById('registerBtn').style.display='none';
        document.getElementById('userPanel').style.display='flex';
        document.getElementById('usernameDisplay').textContent=currentUser.displayName;
        document.getElementById('headerAvatar').src=currentUser.avatar||"https://via.placeholder.com/40?text=👤";
    }

    renderAds();

    const priceInput = document.getElementById('adPrice');
    if(priceInput){
        priceInput.addEventListener('input',()=>{
            const cursorPos = priceInput.selectionStart;
            const raw = priceInput.value;
            const formatted = formatPriceInput(raw);
            priceInput.value = formatted;
            priceInput.selectionStart = priceInput.selectionEnd = cursorPos;
        });
    }
});
