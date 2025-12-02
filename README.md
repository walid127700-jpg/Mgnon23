<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>Magnon Chat - Master Advanced</title>
<style>
body { background:#121212; color:#fff; font-family:sans-serif; margin:0; padding:0; transition:background 2s; }
#chatBox { width:90%; max-width:900px; margin:20px auto; background:#1e1e1e; padding:10px; border-radius:10px; transition:background 1s; }
#messages { height:300px; overflow-y:auto; background:#2a2a2a; padding:10px; border-radius:10px; margin-bottom:10px; }
.message { opacity:0; animation:fadeIn 0.5s forwards; }
@keyframes fadeIn { to {opacity:1;} }
#inputBox { display:flex; margin-bottom:10px; }
#inputBox input { flex:1; padding:10px; border-radius:5px; border:none; margin-right:5px; }
#inputBox button { padding:10px; border:none; border-radius:5px; background:#00c853; color:#fff; cursor:pointer; transition:0.3s; }
#inputBox button:hover { transform:scale(1.05); background:#00e676; }
#micTimerBox { display:none; background:#111; color:#fff; padding:10px; border-radius:10px; margin-top:10px; font-size:14px; }
#micNotification { display:none; position:fixed; top:20px; left:50%; transform:translateX(-50%); background:#00c853; color:#fff; padding:15px 30px; border-radius:10px; font-size:16px; font-weight:bold; box-shadow:0 0 10px #0005; z-index:9999; }
#userCoins { margin-top:10px; font-size:16px; }
#vipBadge { color:#ffd700; font-weight:bold; margin-left:5px; }
#historyBox { background:#222; padding:10px; border-radius:10px; margin-top:10px; max-height:150px; overflow-y:auto; }
button.actionBtn { margin:5px; padding:5px 10px; border:none; border-radius:5px; cursor:pointer; transition:0.3s; }
button.actionBtn:hover { transform:scale(1.05); background:#00e676; }
.roomControlBox { background:#222; padding:10px; border-radius:10px; margin-top:10px; }
.roomControlBox h3 { margin-top:0; }
.roomControlBox ul { max-height:100px; overflow-y:auto; padding-left:20px; }
#masterPanel { background:#333; padding:10px; border-radius:10px; margin-top:10px; }
#masterPanel h3 { margin-top:0; }
#masterPanel div { margin:5px 0; }
#filterBox { margin-top:10px; }
#filterBox input, #filterBox select { margin-right:5px; padding:5px; border-radius:5px; border:none; }
.theme1 { background:#263238 !important; }
.theme2 { background:#1a237e !important; }
.theme3 { background:#004d40 !important; }
</style>
</head>
<body>

<div id="chatBox">
    <h2>Magnon Chat - Master Advanced</h2>
    <div id="messages"></div>

    <div id="inputBox">
        <input type="text" id="msgInput" placeholder="اكتب رسالتك هنا...">
        <button onclick="sendMessage()">إرسال</button>
    </div>

    <button class="actionBtn" onclick="startMic(currentUser)">تشغيل المايك</button>
    <button class="actionBtn" onclick="stopMic(currentUser)">إيقاف المايك</button>

    <div id="micTimerBox">
        ⏱ مدة استخدام المايك: <span id="micUsed">0 دقيقة</span><br>
        ⌛ الوقت المتبقي: <span id="micLeft">60 دقيقة</span>
    </div>

    <div id="userCoins">
        Gold: <span id="coinsAmount">0</span> 🪙 <span id="vipBadge"></span>
    </div>

    <button class="actionBtn" onclick="buyVIP(currentUser)">شراء رتبة VIP - 50 Gold</button>
    <button class="actionBtn" onclick="buyTheme(currentUser)">شراء ثيم خاص - 20 Gold</button>
    <button class="actionBtn" onclick="sendGift(currentUser)">إرسال هدية - 10 Gold</button>

    <div id="historyBox">
        <strong>سجل المكافآت:</strong>
        <ul id="historyList"></ul>
    </div>

    <!-- فلتر الرسائل -->
    <div id="filterBox">
        <input type="text" id="filterUser" placeholder="فلتر حسب المستخدم">
        <input type="date" id="filterDate">
        <button onclick="applyFilter()">تطبيق الفلتر</button>
        <button onclick="clearFilter()">مسح الفلتر</button>
    </div>

    <!-- لوحة تحكم غرف -->
    <div id="roomControlRoom1" class="roomControlBox">
        <h3>لوحة تحكم الغرفة: Room1</h3>
        <div><strong>سجل الدخول:</strong><ul id="loginHistoryRoom1"></ul></div>
        <div><strong>سجل الخروج:</strong><ul id="logoutHistoryRoom1"></ul></div>
    </div>

    <div id="roomControlRoom2" class="roomControlBox">
        <h3>لوحة تحكم الغرفة: Room2</h3>
        <div><strong>سجل الدخول:</strong><ul id="loginHistoryRoom2"></ul></div>
        <div><strong>سجل الخروج:</strong><ul id="logoutHistoryRoom2"></ul></div>
    </div>

    <!-- لوحة الماستر التفاعلية -->
    <div id="masterPanel">
        <h3>لوحة الماستر التفاعلية</h3>
        <div>عدد المستخدمين الحاليين في Room1: <span id="room1Users">0</span></div>
        <div>عدد الرسائل في Room1: <span id="room1Msgs">0</span></div>
        <div>إجمالي Gold المستخدمين في Room1: <span id="room1Gold">0</span></div>
        <div>عدد المستخدمين الحاليين في Room2: <span id="room2Users">0</span></div>
        <div>عدد الرسائل في Room2: <span id="room2Msgs">0</span></div>
        <div>إجمالي Gold المستخدمين في Room2: <span id="room2Gold">0</span></div>
    </div>
</div>

<div id="micNotification">🎉 اكتملت الساعة! حصلت على +3 Gold 🪙</div>

<audio id="soundMessage" src="sounds/message.mp3"></audio>
<audio id="soundGift" src="sounds/gift.mp3"></audio>
<audio id="micCompleteSound" src="sounds/complete.mp3"></audio>

<script>
// ===== بيانات المستخدمين =====
let users = {
    "user1": { coins:0, XP:0, VIP:false, theme:false, micStartTime:0, lastMicUse:0, history:[], messagesSent:0 },
    "user2": { coins:0, XP:0, VIP:false, theme:false, micStartTime:0, lastMicUse:0, history:[], messagesSent:0 }
};
let currentUser="user1";

// ===== بيانات الغرف =====
let rooms = {
    "Room1": { loginHistory:[], logoutHistory:[], messages:0 },
    "Room2": { loginHistory:[], logoutHistory:[], messages:0 }
};

// ===== وظائف الشات =====
function sendMessage(){
    let msg=document.getElementById("msgInput").value;
    if(msg.trim()==="") return;
    let messages=document.getElementById("messages");
    let div=document.createElement("div");
    div.classList.add("message");
    div.innerHTML=`<strong>${currentUser}:</strong> ${msg}`;
    div.dataset.user=currentUser;
    div.dataset.date=new Date().toISOString().split("T")[0];
    messages.appendChild(div);
    document.getElementById("msgInput").value="";
    messages.scrollTop=messages.scrollHeight;
    users[currentUser].XP+=1;
    users[currentUser].messagesSent+=1;
    users[currentUser].history.push(`أرسلت رسالة: +1 XP`);
    rooms["Room1"].messages+=1;
    document.getElementById("soundMessage").play();
    updateHistory(currentUser);
    updateMasterPanel();
}

// ===== الفلتر =====
function applyFilter(){
    let userFilter=document.getElementById("filterUser").value.toLowerCase();
    let dateFilter=document.getElementById("filterDate").value;
    document.querySelectorAll("#messages .message").forEach(msg=>{
        let user=msg.dataset.user.toLowerCase();
        let date=msg.dataset.date;
        if((!userFilter || user.includes(userFilter)) && (!dateFilter || date===dateFilter)){ msg.style.display="block"; }
        else{ msg.style.display="none"; }
    });
}
function clearFilter(){ document.querySelectorAll("#messages .message").forEach(msg=>msg.style.display="block"); document.getElementById("filterUser").value=""; document.getElementById("filterDate").value=""; }

// باقي الكود (الميك، VIP، الثيم، الهدايا، السجلات، لوحة الماستر) كما في النسخة السابقة، مع تحديثMasterPanel() بعد كل حدث
</script>

</body>
</html>
