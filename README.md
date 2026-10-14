<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>教師控制台 - ClassPoll</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
</head>
<body class="bg-slate-100 font-sans min-h-screen text-slate-800">
    <nav class="bg-indigo-600 text-white shadow-md">
        <div class="max-w-6xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa-solid fa-chalkboard-user text-2xl"></i>
                <span class="font-bold text-xl tracking-wide">ClassPoll 教師控制台</span>
            </div>
            <div class="flex items-center gap-3">
                <div id="authStatus" class="text-sm bg-indigo-700 px-3 py-1 rounded-full flex items-center gap-2">
                    <span id="statusDot" class="w-2 h-2 rounded-full bg-amber-400"></span>
                    <span id="statusText">檢查登入狀態中…</span>
                </div>
                <button id="logoutBtn" onclick="handleLogout()" class="hidden bg-indigo-800 hover:bg-indigo-900 text-xs px-3 py-1 rounded font-bold text-white transition">登出</button>
            </div>
        </div>
    </nav>

    <div class="max-w-4xl mx-auto p-4 md:p-6 space-y-6">
        <div id="authCard" class="bg-white p-6 rounded-2xl shadow-md border border-indigo-100">
            <h3 class="text-lg font-bold mb-4 text-indigo-900 border-b pb-2 flex items-center gap-2">
                <i class="fa-solid fa-lock text-indigo-600"></i> 教師身分驗證
            </h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                    <label for="authEmail" class="block text-xs font-semibold text-slate-600 mb-1">電子郵件 (Email)</label>
                    <input type="email" id="authEmail" autocomplete="username" placeholder="teacher@school.edu" class="w-full px-3 py-2 border rounded-lg text-sm focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
                <div>
                    <label for="authPassword" class="block text-xs font-semibold text-slate-600 mb-1">密碼 (Password)</label>
                    <input type="password" id="authPassword" autocomplete="current-password" placeholder="至少 6 個字元" class="w-full px-3 py-2 border rounded-lg text-sm focus:ring-2 focus:ring-indigo-500 outline-none">
                </div>
            </div>
            <div class="flex gap-3 mt-4">
                <button id="loginBtn" onclick="handleLogin()" class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 rounded-lg text-sm transition shadow">登入帳號</button>
                <button id="signupBtn" onclick="handleSignUp()" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold py-2 rounded-lg text-sm transition">註冊新帳號</button>
            </div>
            <p class="text-xs text-slate-500 mt-3">若註冊後要求 Email 驗證，請先完成信箱驗證再登入。</p>
        </div>

        <div id="mainDashboard" class="opacity-50 pointer-events-none space-y-6">
            <div class="bg-indigo-900 text-white p-6 rounded-2xl shadow-md flex flex-wrap justify-between items-center gap-4">
                <div>
                    <span class="text-indigo-300 text-sm font-medium">當前教室代碼</span>
                    <div class="flex items-center gap-2 mt-1">
                        <input type="text" id="teacherRoomCodeInput" value="888" maxlength="30" class="bg-indigo-800 text-amber-300 font-mono font-black text-2xl px-3 py-1 rounded border border-indigo-600 w-32 uppercase text-center focus:outline-none">
                        <button onclick="switchRoom()" class="bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold text-xs px-4 py-2 rounded shadow transition">切換 / 建立教室</button>
                    </div>
                </div>
                <div class="flex items-center gap-3 flex-wrap">
                    <button onclick="generateBotAnswers()" class="bg-indigo-700 hover:bg-indigo-600 text-indigo-100 text-sm px-4 py-2 rounded-lg transition border border-indigo-500">
                        <i class="fa-solid fa-robot mr-1"></i> 模擬回答（測試提示）
                    </button>
                    <button onclick="clearCurrentAnswers()" class="bg-rose-600 hover:bg-rose-700 text-white text-sm px-4 py-2 rounded-lg transition">
                        <i class="fa-solid fa-trash mr-1"></i> 清除回答紀錄
                    </button>
                </div>
            </div>

            <div class="bg-white p-6 rounded-2xl shadow-md">
                <h3 class="text-lg font-bold mb-4 text-slate-800 border-b pb-2"><i class="fa-solid fa-paper-plane text-indigo-600 mr-2"></i>發布新題目</h3>
                <div class="space-y-4">
                    <div>
                        <label for="questionInput" class="block text-sm font-medium text-slate-700 mb-1">題目內容</label>
                        <input type="text" id="questionInput" value="請問下列何者是台灣最高的山？" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                        <div><label for="optA" class="block text-xs text-slate-500 mb-1">選項 A (索引 0)</label><input type="text" id="optA" value="玉山" class="w-full px-3 py-2 border rounded-lg text-sm"></div>
                        <div><label for="optB" class="block text-xs text-slate-500 mb-1">選項 B (索引 1)</label><input type="text" id="optB" value="雪山" class="w-full px-3 py-2 border rounded-lg text-sm"></div>
                        <div><label for="optC" class="block text-xs text-slate-500 mb-1">選項 C (索引 2)</label><input type="text" id="optC" value="陽明山" class="w-full px-3 py-2 border rounded-lg text-sm"></div>
                        <div><label for="optD" class="block text-xs text-slate-500 mb-1">選項 D (索引 3)</label><input type="text" id="optD" value="阿里山" class="w-full px-3 py-2 border rounded-lg text-sm"></div>
                    </div>
                    <button onclick="publishQuestion()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 rounded-lg shadow-md transition">即時發布題目至 Supabase</button>
                </div>
            </div>

            <div class="bg-white p-6 rounded-2xl shadow-md">
                <div class="flex justify-between items-center mb-4 border-b pb-2 gap-3">
                    <h3 class="text-lg font-bold text-slate-800"><i class="fa-solid fa-chart-column text-indigo-600 mr-2"></i>即時統計圖表</h3>
                    <span class="text-sm bg-slate-100 text-slate-600 px-3 py-1 rounded-full font-bold whitespace-nowrap">已回答人數: <span id="totalRespondedCount" class="text-indigo-600 text-base">0</span> 人</span>
                </div>
                <div id="activeQuestionLabel" class="text-sm text-slate-500 mb-3">目前沒有已發布的題目</div>
                <div class="w-full max-w-md mx-auto my-4"><canvas id="pollChart"></canvas></div>
                <div class="mt-6">
                    <h4 class="text-sm font-semibold text-slate-600 mb-2">學生提交明細（Supabase Realtime）：</h4>
                    <div id="studentResponsesList" class="max-h-40 overflow-y-auto space-y-1 bg-slate-50 p-3 rounded-lg border text-xs text-slate-600"><div class="text-slate-400 text-center">尚無回答數據</div></div>
                </div>
            </div>
        </div>
    </div>

<script>
/*
 * 請將以下兩個佔位值換成 Supabase 專案的 Project URL 與 Publishable/anon key。
 * 不可在前端使用 service_role 或 Secret key。
 */
const SUPABASE_URL = "https://yqrytizwuwgxpkqyyymd.supabase.co";
const SUPABASE_KEY = "sb_publishable_jydU8c5DZMBHuKLFuK2JOg_ZjmncLwU";

let _supabase;
let currentUser = null;
let currentRoom = "888";
let currentClassroomId = null;
let chartInstance = null;
let activeQuestion = null;
let votesList = [];
let votesSubscription = null;
let authBusy = false;

function escapeHTML(value) {
    return String(value ?? "").replace(/[&<>"']/g, ch => ({
        "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;"
    })[ch]);
}
function getOptions(question) {
    if (!question || question.options == null) return [];
    if (Array.isArray(question.options)) return question.options;
    try {
        const parsed = JSON.parse(question.options);
        return Array.isArray(parsed) ? parsed : [];
    } catch (e) {
        console.error("題目選項格式不正確：", e);
        return [];
    }
}
function setAuthBusy(busy) {
    authBusy = busy;
    document.getElementById("loginBtn").disabled = busy;
    document.getElementById("signupBtn").disabled = busy;
    document.getElementById("loginBtn").classList.toggle("opacity-50", busy);
    document.getElementById("signupBtn").classList.toggle("opacity-50", busy);
}
async function handleLogin() {
    if (authBusy) return;
    const email = document.getElementById("authEmail").value.trim();
    const password = document.getElementById("authPassword").value;
    if (!email || !password) return alert("請輸入電子郵件與密碼！");
    setAuthBusy(true);
    try {
        const { error } = await _supabase.auth.signInWithPassword({ email, password });
        if (error) alert("登入失敗：" + error.message);
    } catch (e) {
        alert("登入時發生錯誤：" + e.message);
    } finally {
        setAuthBusy(false);
    }
}
async function handleSignUp() {
    if (authBusy) return;
    const email = document.getElementById("authEmail").value.trim();
    const password = document.getElementById("authPassword").value;
    if (!email || !password) return alert("請輸入電子郵件與密碼！");
    if (password.length < 6) return alert("密碼至少需要 6 個字元。");
    setAuthBusy(true);
    try {
        const { data, error } = await _supabase.auth.signUp({ email, password });
        if (error) return alert("註冊失敗：" + error.message);
        if (!data.session) {
            alert("註冊申請已送出！請先查看信箱並完成 Email 驗證，再回到此頁登入。");
        } else {
            alert("註冊成功，已登入教師控制台。");
        }
    } catch (e) {
        alert("註冊時發生錯誤：" + e.message);
    } finally {
        setAuthBusy(false);
    }
}
async function handleLogout() {
    const { error } = await _supabase.auth.signOut();
    if (error) return alert("登出失敗：" + error.message);
    currentUser = null;
    currentClassroomId = null;
    activeQuestion = null;
    votesList = [];
    if (votesSubscription) {
        await _supabase.removeChannel(votesSubscription);
        votesSubscription = null;
    }
    checkAuthUI();
    updateDashboard();
}
function checkAuthUI() {
    const loggedIn = !!currentUser && !currentUser.is_anonymous;
    document.getElementById("authCard").classList.toggle("hidden", loggedIn);
    document.getElementById("mainDashboard").classList.toggle("opacity-50", !loggedIn);
    document.getElementById("mainDashboard").classList.toggle("pointer-events-none", !loggedIn);
    document.getElementById("logoutBtn").classList.toggle("hidden", !currentUser);
    document.getElementById("statusDot").className = "w-2 h-2 rounded-full " + (loggedIn ? "bg-emerald-400 animate-pulse" : "bg-amber-400");
    document.getElementById("statusText").innerText = loggedIn ? "已登入：" + (currentUser.email || "教師帳號") : (currentUser ? "目前是匿名帳號，請使用教師帳號登入" : "未登入");
}
async function removeVoteSubscription() {
    if (votesSubscription) {
        await _supabase.removeChannel(votesSubscription);
        votesSubscription = null;
    }
}
async function getOrCreateClassroom(roomCode) {
    if (!currentUser || currentUser.is_anonymous) {
        alert("請先使用教師 Email 帳號登入。");
        return null;
    }
    const { data: classroom, error } = await _supabase
        .from("classrooms").select("id, room_code, teacher_id, is_active")
        .eq("room_code", roomCode).maybeSingle();

    if (error) {
        console.error("查詢教室錯誤：", error);
        alert("查詢教室失敗：" + error.message);
        return null;
    }
    if (classroom) {
        if (classroom.teacher_id !== currentUser.id) {
            alert("這個教室代碼已被其他教師使用，請輸入其他代碼。");
            return null;
        }
        if (!classroom.is_active) {
            const { error: activeError } = await _supabase.from("classrooms")
                .update({ is_active: true }).eq("id", classroom.id);
            if (activeError) {
                alert("重新啟用教室失敗：" + activeError.message);
                return null;
            }
        }
        return classroom.id;
    }
    const { data: newClassroom, error: createError } = await _supabase
        .from("classrooms")
        .insert([{ room_code: roomCode, teacher_id: currentUser.id, is_active: true }])
        .select("id").single();

    if (createError) {
        console.error("建立教室錯誤：", createError);
        alert("建立教室失敗：" + createError.message + "\n請確認 classrooms 的 RLS INSERT/SELECT 政策與 teacher_id 欄位設定。");
        return null;
    }
    return newClassroom.id;
}
async function switchRoom() {
    if (!currentUser || currentUser.is_anonymous) return alert("請先登入教師帳號！");
    const newRoom = document.getElementById("teacherRoomCodeInput").value.trim().toUpperCase();
    if (!newRoom) return alert("請輸入有效的教室代碼！");
    if (newRoom.length > 30) return alert("教室代碼最多 30 個字元。");

    await removeVoteSubscription();
    currentRoom = newRoom;
    currentClassroomId = null;
    activeQuestion = null;
    votesList = [];
    updateDashboard();

    const id = await getOrCreateClassroom(currentRoom);
    if (!id) return;
    currentClassroomId = id;
    await loadCurrentQuestionAndVotes();
    alert("已成功切換至教室代碼：" + currentRoom);
}
async function loadCurrentQuestionAndVotes() {
    if (!currentUser || currentUser.is_anonymous) return;
    const classroomId = await getOrCreateClassroom(currentRoom);
    if (!classroomId) {
        currentClassroomId = null;
        activeQuestion = null;
        votesList = [];
        updateDashboard();
        return;
    }
    currentClassroomId = classroomId;
    const { data: qData, error: qError } = await _supabase
        .from("questions").select("*")
        .eq("classroom_id", currentClassroomId)
        .eq("status", "published")
        .order("created_at", { ascending: false }).limit(1).maybeSingle();

    if (qError) {
        console.error("讀取題目失敗：", qError);
        alert("讀取題目失敗：" + qError.message);
        return;
    }
    activeQuestion = qData || null;
    if (activeQuestion) {
        await fetchVotes(activeQuestion.id);
        subscribeVotes(activeQuestion.id);
    } else {
        await removeVoteSubscription();
        votesList = [];
        updateDashboard();
    }
}
async function fetchVotes(questionId) {
    if (!currentUser || !currentClassroomId || !activeQuestion || activeQuestion.id !== questionId) return;
    const { data, error } = await _supabase.from("votes").select("id, question_id, student_id, selected_option, created_at").eq("question_id", questionId);
    if (error) {
        console.error("讀取投票資料失敗：", error);
        document.getElementById("studentResponsesList").innerHTML = '<div class="text-rose-600 text-center">讀取回答失敗，請檢查 votes 的 SELECT RLS 政策。</div>';
        return;
    }
    votesList = data || [];
    updateDashboard();
}
function subscribeVotes(questionId) {
    removeVoteSubscription().then(() => {
        if (!activeQuestion || activeQuestion.id !== questionId) return;
        votesSubscription = _supabase.channel("teacher-votes-" + questionId)
            .on("postgres_changes", {
                event: "*", schema: "public", table: "votes",
                filter: "question_id=eq." + questionId
            }, () => { fetchVotes(questionId); })
            .subscribe(status => {
                if (status === "SUBSCRIBED") console.log("Realtime 已連線，題目 ID：", questionId);
                if (status === "CHANNEL_ERROR" || status === "TIMED_OUT") console.warn("Realtime 訂閱狀態：", status);
            });
    });
}
async function publishQuestion() {
    if (!currentUser || currentUser.is_anonymous) return alert("請先完成教師身分登入！");
    const questionText = document.getElementById("questionInput").value.trim();
    const options = ["optA", "optB", "optC", "optD"]
        .map(id => document.getElementById(id).value.trim()).filter(Boolean);
    if (!questionText) return alert("請輸入題目內容！");
    if (options.length < 2) return alert("請至少輸入兩個選項！");
    if (new Set(options).size !== options.length) return alert("選項內容請勿重複。");

    if (!currentClassroomId) {
        currentClassroomId = await getOrCreateClassroom(currentRoom);
        if (!currentClassroomId) return;
    }
    const { error: closeError } = await _supabase.from("questions")
        .update({ status: "closed" })
        .eq("classroom_id", currentClassroomId).eq("status", "published");
    if (closeError) return alert("關閉舊題目失敗：" + closeError.message);

    const { data, error } = await _supabase.from("questions").insert([{
        classroom_id: currentClassroomId,
        question_text: questionText,
        options,
        status: "published"
    }]).select().single();
    if (error) return alert("發布題目失敗：" + error.message);

    activeQuestion = data;
    votesList = [];
    updateDashboard();
    subscribeVotes(activeQuestion.id);
    alert("題目已成功發布至教室 [" + currentRoom + "]！");
}
function updateDashboard() {
    document.getElementById("totalRespondedCount").innerText = String(votesList.length);
    const listEl = document.getElementById("studentResponsesList");
    const labelEl = document.getElementById("activeQuestionLabel");
    if (!activeQuestion) {
        labelEl.innerText = "目前沒有已發布的題目";
        listEl.innerHTML = '<div class="text-slate-400 text-center">尚無回答數據</div>';
        if (chartInstance) {
            chartInstance.data.labels = ["尚無題目"];
            chartInstance.data.datasets[0].data = [0];
            chartInstance.update();
        }
        return;
    }
    labelEl.innerText = "目前題目：" + activeQuestion.question_text;
    const options = getOptions(activeQuestion);
    if (votesList.length === 0) {
        listEl.innerHTML = '<div class="text-slate-400 text-center">尚無回答數據</div>';
    } else {
        listEl.innerHTML = votesList.map(v => {
            const optionText = options[Number(v.selected_option)] !== undefined
                ? options[Number(v.selected_option)] : "未知選項";
            return '<div class="flex justify-between gap-3 border-b pb-1">' +
                '<span class="font-medium break-all">' + escapeHTML(v.student_id) + '</span>' +
                '<span class="text-indigo-600 font-bold">' + escapeHTML(optionText) + '</span></div>';
        }).join("");
    }
    if (chartInstance) {
        chartInstance.data.labels = options;
        chartInstance.data.datasets[0].data = options.map((_, i) =>
            votesList.filter(v => Number(v.selected_option) === i).length
        );
        chartInstance.update();
    }
}
function initChart() {
    const ctx = document.getElementById("pollChart").getContext("2d");
    chartInstance = new Chart(ctx, {
        type: "bar",
        data: {
            labels: ["選項 A", "選項 B"],
            datasets: [{
                label: "得票數",
                data: [0, 0],
                backgroundColor: "rgba(99, 102, 241, 0.75)",
                borderColor: "rgba(99, 102, 241, 1)",
                borderWidth: 1.5,
                borderRadius: 6
            }]
        },
        options: {
            responsive: true,
            scales: { y: { beginAtZero: true, ticks: { stepSize: 1, precision: 0 } } }
        }
    });
}
async function generateBotAnswers() {
    alert("此按鈕不會將假資料寫入正式投票表。請用學生端開啟匿名登入並提交測試答案，避免違反 votes 的學生身分 RLS 規則。");
}
async function clearCurrentAnswers() {
    if (!activeQuestion) return alert("目前沒有已發布的題目。");
    if (!confirm("確定要清除本題所有學生的回答紀錄嗎？此操作無法復原。")) return;
    const { error } = await _supabase.from("votes").delete().eq("question_id", activeQuestion.id);
    if (error) {
        alert("清除紀錄失敗：" + error.message + "\n請確認已設定允許教師刪除自己教室投票的 DELETE RLS 政策。");
        return;
    }
    await fetchVotes(activeQuestion.id);
}
function validateConfig() {
    if (SUPABASE_URL.includes("YOUR_PROJECT_ID") || SUPABASE_KEY.includes("YOUR_PUBLISHABLE_ANON_KEY")) {
        document.getElementById("statusText").innerText = "請先設定 Supabase 連線資訊";
        alert("請先編輯 HTML，填入真正的 Supabase Project URL 和 Publishable/anon key。");
        return false;
    }
    return true;
}
window.addEventListener("load", async () => {
    if (!validateConfig()) return;
    try {
        _supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
        initChart();
        checkAuthUI();

        // 避免在 onAuthStateChange callback 內直接 await 資料庫操作，造成 Auth 鎖定。
        _supabase.auth.onAuthStateChange((event, session) => {
            currentUser = session ? session.user : null;
            checkAuthUI();
            if (currentUser && !currentUser.is_anonymous) {
                setTimeout(() => loadCurrentQuestionAndVotes(), 0);
            } else {
                currentClassroomId = null;
                activeQuestion = null;
                votesList = [];
                removeVoteSubscription();
                updateDashboard();
            }
        });

        const { data, error } = await _supabase.auth.getSession();
        if (error) throw error;
        currentUser = data.session ? data.session.user : null;
        checkAuthUI();
        if (currentUser && !currentUser.is_anonymous) await loadCurrentQuestionAndVotes();
    } catch (e) {
        console.error("初始化失敗：", e);
        alert("初始化 Supabase 失敗：" + e.message);
    }
});
</script>
</body>
</html>
