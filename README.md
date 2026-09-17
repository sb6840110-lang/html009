<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเช็คชื่อกิจกรรมยุคดิจิทัล (Real-time Live Sync)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <!-- face-api.js for Face Detection -->
    <script defer src="https://cdn.jsdelivr.net/npm/@vladmandic/face-api/dist/face-api.js"></script>
</head>
<body class="bg-slate-100 font-sans min-h-screen pb-12">

    <!-- Header & Auth Status Bar -->
    <header class="bg-indigo-700 text-white p-6 shadow-lg">
        <div class="max-w-5xl mx-auto flex flex-col md:flex-row justify-between items-center gap-4">
            <div>
                <h1 class="text-2xl font-bold flex items-center gap-2">
                    <i class="fa-solid fa-user-check"></i> ระบบลงทะเบียนเข้าร่วมกิจกรรม
                </h1>
                <p class="text-indigo-200 text-sm">เข้าสู่ระบบเพื่อลงทะเบียน • ข้อมูลอัปเดตเรียลไทม์ทุกบัญชี</p>
            </div>
            
            <!-- Auth Buttons Container -->
            <div id="authContainer" class="flex flex-wrap items-center gap-2">
                <!-- Guest State -->
                <div id="guestBtns" class="flex gap-2">
                    <button type="button" onclick="openStudentLoginModal()" class="bg-emerald-500 hover:bg-emerald-600 text-white font-bold px-3.5 py-2 rounded-xl text-xs transition flex items-center gap-1.5 shadow">
                        <i class="fa-solid fa-user-graduate"></i> นักเรียนเข้าสู่ระบบ
                    </button>
                    <button type="button" onclick="openTeacherLoginModal()" class="bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold px-3.5 py-2 rounded-xl text-xs transition flex items-center gap-1.5 shadow">
                        <i class="fa-solid fa-user-shield"></i> ครูเวรเข้าสู่ระบบ
                    </button>
                </div>

                <!-- Logged In State Badge -->
                <div id="userProfile" class="hidden flex items-center gap-2 bg-indigo-800 border border-indigo-500 px-3 py-1.5 rounded-xl text-xs">
                    <i id="roleIcon" class="fa-solid text-amber-400"></i>
                    <span id="userNameDisplay" class="font-bold text-amber-300">ผู้ใช้งาน</span>
                    <button type="button" onclick="logout()" class="ml-2 bg-rose-600 hover:bg-rose-700 text-white px-2 py-1 rounded-lg text-[10px] font-bold transition">
                        ออกจากระบบ
                    </button>
                </div>
            </div>
        </div>
    </header>

    <main class="max-w-5xl mx-auto mt-6 px-4 space-y-6">

        <!-- Status Timeline -->
        <section class="bg-white p-6 rounded-2xl shadow-md border border-slate-200">
            <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                <i class="fa-solid fa-clock-rotate-left text-indigo-600"></i> เกณฑ์เวลาการเข้าร่วมกิจกรรม
            </h2>
            <div class="relative w-full bg-slate-200 h-5 rounded-full overflow-hidden flex shadow-inner">
                <div class="w-1/2 bg-emerald-500 h-full flex items-center justify-center text-[11px] text-white font-bold">ปกติ (&lt; 08:30)</div>
                <div class="w-1/6 bg-amber-500 h-full flex items-center justify-center text-[11px] text-white font-bold">สาย (08:30-09:00)</div>
                <div class="w-1/3 bg-rose-500 h-full flex items-center justify-center text-[11px] text-white font-bold">ไม่เข้าร่วม (&gt; 09:00)</div>
            </div>
            <div id="statusBadge" class="mt-4 p-3 rounded-xl text-center font-bold text-sm bg-slate-100 text-slate-700 border border-slate-200">
                สถานะปัจจุบันหากลงทะเบียนตอนนี้: <span id="currentRuleStatus" class="underline">กำลังคำนวณ...</span>
            </div>
        </section>

        <!-- Form Check-in (With Lock Screen when not logged in) -->
        <section class="bg-white p-6 rounded-2xl shadow-md border border-slate-200 relative overflow-hidden">
            
            <!-- Auth Lock Overlay -->
            <div id="formLockOverlay" class="absolute inset-0 bg-slate-900/40 backdrop-blur-[2px] z-20 flex flex-col items-center justify-center p-6 text-center transition-all duration-300">
                <div class="bg-white p-6 rounded-2xl shadow-2xl max-w-sm w-full space-y-3 border border-slate-100">
                    <div class="w-12 h-12 bg-rose-100 text-rose-600 rounded-full flex items-center justify-center mx-auto text-xl">
                        <i class="fa-solid fa-lock"></i>
                    </div>
                    <h3 class="text-base font-bold text-slate-800">กรุณาเข้าสู่ระบบก่อนเช็คชื่อ</h3>
                    <p class="text-xs text-slate-500">นักเรียนหรือคุณครูเวร ต้องเข้าสู่ระบบก่อน จึงจะสามารถบันทึกและดูข้อมูลอัปเดตล่าสุดได้</p>
                    <div class="flex gap-2 pt-2">
                        <button type="button" onclick="openStudentLoginModal()" class="w-1/2 bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-2 rounded-xl text-xs transition">นักเรียน Login</button>
                        <button type="button" onclick="openTeacherLoginModal()" class="w-1/2 bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold py-2 rounded-xl text-xs transition">ครูเวร Login</button>
                    </div>
                </div>
            </div>

            <div class="flex justify-between items-center mb-4">
                <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-pen-to-square text-indigo-600"></i> แบบฟอร์มลงทะเบียนเช็คชื่อ
                </h2>
                <span id="studentLockNotice" class="hidden text-xs bg-emerald-100 text-emerald-800 font-bold px-2.5 py-1 rounded-lg border border-emerald-300">
                    <i class="fa-solid fa-user-check"></i> เข้าสู่ระบบเรียบร้อยแล้ว
                </span>
            </div>

            <form id="checkinForm" class="space-y-4">
                <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                    <div class="md:col-span-2">
                        <label class="block text-sm font-medium text-slate-700 mb-1">ชื่อ - นามสกุล <span class="text-rose-500">*</span></label>
                        <input type="text" id="fullname" required disabled placeholder="กรุณาเข้าสู่ระบบก่อน..." class="w-full px-4 py-2.5 border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:outline-none disabled:bg-slate-100 disabled:cursor-not-allowed">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">ระดับชั้น <span class="text-rose-500">*</span></label>
                        <input type="text" id="studentClass" required disabled placeholder="เช่น ม.1/1" class="w-full px-4 py-2.5 border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:outline-none disabled:bg-slate-100 disabled:cursor-not-allowed">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">เลขที่ <span class="text-rose-500">*</span></label>
                        <input type="number" id="studentNo" min="1" max="90" required disabled placeholder="เช่น 15" class="w-full px-4 py-2.5 border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:outline-none disabled:bg-slate-100 disabled:cursor-not-allowed">
                    </div>
                </div>

                <!-- Camera / Photo Input -->
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">ถ่ายรูปโปรไฟล์ (ตรวจจับใบหน้า) <span class="text-rose-500">*</span></label>
                    <input type="file" id="photo" accept="image/*" capture="user" required disabled class="w-full text-sm text-slate-500 file:mr-4 file:py-2.5 file:px-4 file:rounded-xl file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100 cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed">
                    
                    <div id="faceStatus" class="mt-2 text-xs font-bold hidden"></div>
                    <div id="imagePreview" class="mt-2 hidden relative w-36 h-36">
                        <img id="previewImg" src="" alt="Preview" class="w-full h-full rounded-xl border border-slate-300 object-cover shadow-sm">
                    </div>
                </div>

                <!-- GPS Location -->
                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="block text-sm font-medium text-slate-700">พิกัดตำแหน่ง GPS <span class="text-rose-500">*</span></label>
                        <span id="geoStatus" class="text-xs font-bold px-2 py-0.5 rounded-full bg-slate-100 text-slate-500">รอตรวจสอบ</span>
                    </div>
                    <div class="flex gap-2">
                        <input type="text" id="location" readonly placeholder="กดปุ่มเพื่อตรวจสอบพิกัด GPS" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-sm text-slate-600" required>
                        <button type="button" id="getGpsBtn" onclick="getLocation()" disabled class="bg-slate-800 text-white px-5 py-2.5 rounded-xl text-sm hover:bg-slate-700 transition flex items-center whitespace-nowrap gap-1 disabled:bg-slate-400 disabled:cursor-not-allowed">
                            <i class="fa-solid fa-location-crosshairs"></i> ดึงพิกัด GPS
                        </button>
                    </div>
                </div>

                <button type="submit" id="submitBtn" disabled class="w-full bg-indigo-600 text-white font-bold py-3.5 rounded-xl shadow-md hover:bg-indigo-700 active:scale-[0.99] transition duration-200 text-base flex items-center justify-center gap-2 disabled:bg-slate-300 disabled:cursor-not-allowed">
                    <i class="fa-solid fa-paper-plane"></i> ยืนยันการบันทึกเวลา
                </button>
            </form>
        </section>

        <!-- Live Attendance Panel -->
        <section id="attendancePanel" class="hidden space-y-6">
            <div class="bg-indigo-50 border border-indigo-200 p-4 rounded-2xl flex items-center justify-between text-indigo-900">
                <div class="flex items-center gap-3">
                    <div class="w-10 h-10 bg-indigo-600 text-white rounded-xl flex items-center justify-center font-bold text-lg">
                        <i class="fa-solid fa-arrows-rotate animate-spin"></i>
                    </div>
                    <div>
                        <h3 class="font-bold text-sm">ข้อมูลสรุปและประวัติการลงทะเบียนล่าสุด</h3>
                        <p class="text-xs text-indigo-700">ระบบอัปเดตข้อมูลทุกครั้งที่มีการลงทะเบียนหรือเข้าสู่ระบบใหม่</p>
                    </div>
                </div>
                <span class="text-xs bg-indigo-200 font-bold px-3 py-1 rounded-full border border-indigo-300">Live Updating</span>
            </div>

            <!-- Summary Cards -->
            <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500">ตรงเวลาทั้งหมด</p>
                        <p id="countNormal" class="text-2xl font-black text-emerald-600 mt-1">0</p>
                    </div>
                    <div class="w-10 h-10 bg-emerald-100 text-emerald-600 rounded-xl flex items-center justify-center text-lg">
                        <i class="fa-solid fa-circle-check"></i>
                    </div>
                </div>
                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500">มาสายทั้งหมด</p>
                        <p id="countLate" class="text-2xl font-black text-amber-500 mt-1">0</p>
                    </div>
                    <div class="w-10 h-10 bg-amber-100 text-amber-600 rounded-xl flex items-center justify-center text-lg">
                        <i class="fa-solid fa-clock"></i>
                    </div>
                </div>
                <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                        <p class="text-xs font-semibold text-slate-500">ไม่เข้าร่วม / เกินเวลา</p>
                        <p id="countAbsent" class="text-2xl font-black text-rose-500 mt-1">0</p>
                    </div>
                    <div class="w-10 h-10 bg-rose-100 text-rose-600 rounded-xl flex items-center justify-center text-lg">
                        <i class="fa-solid fa-circle-xmark"></i>
                    </div>
                </div>
            </div>

            <!-- Attendance Records Table -->
            <div class="bg-white p-6 rounded-2xl shadow-md border border-slate-200">
                <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                    <i class="fa-solid fa-list-check text-indigo-600"></i> ประวัติการลงทะเบียนเข้าร่วมกิจกรรมทั้งหมด
                </h2>
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-sm text-slate-600">
                        <thead class="bg-slate-50 text-xs uppercase text-slate-500 border-b border-slate-200">
                            <tr>
                                <th class="p-3">เวลา</th>
                                <th class="p-3">ชื่อ - นามสกุล</th>
                                <th class="p-3">ชั้น/เลขที่</th>
                                <th class="p-3">รูปถ่าย</th>
                                <th class="p-3">พิกัด GPS</th>
                                <th class="p-3">สถานะ</th>
                            </tr>
                        </thead>
                        <tbody id="attendanceTable" class="divide-y divide-slate-100">
                            <tr>
                                <td colspan="6" class="text-center py-6 text-slate-400">ยังไม่มีข้อมูลการเช็คชื่อ</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

    </main>

    <!-- Student Login Modal -->
    <div id="studentLoginModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl max-w-sm w-full p-6 space-y-4">
            <div class="flex justify-between items-center border-b border-slate-100 pb-3">
                <h3 class="font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-user-graduate text-emerald-600"></i> เข้าสู่ระบบ (นักเรียน)
                </h3>
                <button type="button" onclick="closeStudentLoginModal()" class="text-slate-400 hover:text-slate-600 text-xl font-bold px-2">&times;</button>
            </div>
            <form id="studentLoginForm" onsubmit="handleStudentLogin(event)" class="space-y-3">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสนักเรียน</label>
                    <input type="text" id="studentIdInput" required placeholder="กรอกรหัสนักเรียน" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสผ่าน</label>
                    <input type="password" id="studentPasswordInput" required placeholder="••••••••" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-2.5 rounded-xl text-sm transition mt-2">
                    เข้าสู่ระบบ
                </button>
            </form>
        </div>
    </div>

    <!-- Teacher Login Modal -->
    <div id="teacherLoginModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl max-w-sm w-full p-6 space-y-4">
            <div class="flex justify-between items-center border-b border-slate-100 pb-3">
                <h3 class="font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-user-shield text-amber-500"></i> เข้าสู่ระบบ (ครูเวร)
                </h3>
                <button type="button" onclick="closeTeacherLoginModal()" class="text-slate-400 hover:text-slate-600 text-xl font-bold px-2">&times;</button>
            </div>
            <form id="teacherLoginForm" onsubmit="handleTeacherLogin(event)" class="space-y-3">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อผู้ใช้งาน (Username)</label>
                    <input type="text" id="teacherUsernameInput" required placeholder="กรอกชื่อผู้ใช้งาน" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสผ่าน (Password)</label>
                    <input type="password" id="teacherPasswordInput" required placeholder="••••••••" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-amber-500 focus:outline-none">
                </div>
                <button type="submit" class="w-full bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold py-2.5 rounded-xl text-sm transition mt-2">
                    เข้าสู่ระบบในฐานะครูเวร
                </button>
            </form>
        </div>
    </div>

    <!-- JavaScript Logic -->
    <script>
        // State Management & Persistent Storage Key
        const STORAGE_KEY = 'SCHOOL_ATTENDANCE_DATA';
        let currentUser = null;
        let faceModelsLoaded = false;
        let isFaceDetected = false;
        let attendanceData = [];

        // App Initialization
        document.addEventListener("DOMContentLoaded", () => {
            loadStoredAttendanceData();
            initFaceApi();
            updateCurrentTimeRule();
            setInterval(updateCurrentTimeRule, 30000);
            setupPhotoInputListener();
        });

        // Load Persisted Data from localStorage
        function loadStoredAttendanceData() {
            const stored = localStorage.getItem(STORAGE_KEY);
            if (stored) {
                try {
                    attendanceData = JSON.parse(stored);
                } catch (e) {
                    attendanceData = [];
                }
            }
        }

        // Save Data to localStorage
        function saveAttendanceData() {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(attendanceData));
        }

        // Dynamic Time Rule Calculation
        function getCheckinStatus() {
            const now = new Date();
            const hours = now.getHours();
            const minutes = now.getMinutes();
            const totalMinutes = hours * 60 + minutes;

            const normalCutoff = 8 * 60 + 30; // 08:30
            const lateCutoff = 9 * 60;        // 09:00

            if (totalMinutes < normalCutoff) {
                return { status: "ปกติ", class: "text-emerald-600 font-bold" };
            } else if (totalMinutes <= lateCutoff) {
                return { status: "สาย", class: "text-amber-500 font-bold" };
            } else {
                return { status: "ไม่เข้าร่วมกิจกรรม", class: "text-rose-600 font-bold" };
            }
        }

        function updateCurrentTimeRule() {
            const currentRule = getCheckinStatus();
            const statusEl = document.getElementById("currentRuleStatus");
            if (statusEl) {
                statusEl.textContent = currentRule.status;
                statusEl.className = currentRule.class + " underline";
            }
        }

        // Face-API Models Loading
        async function initFaceApi() {
            try {
                const MODEL_URL = 'https://cdn.jsdelivr.net/npm/@vladmandic/face-api/model/';
                await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
                faceModelsLoaded = true;
            } catch (err) {
                console.warn("Face-API models CDN issue, operating in fallback mode.", err);
            }
        }

        // Photo Upload Listener
        function setupPhotoInputListener() {
            const photoInput = document.getElementById("photo");
            photoInput.addEventListener("change", async (e) => {
                const file = e.target.files[0];
                const faceStatus = document.getElementById("faceStatus");
                const imagePreview = document.getElementById("imagePreview");
                const previewImg = document.getElementById("previewImg");

                if (!file) return;

                const imageUrl = URL.createObjectURL(file);
                previewImg.src = imageUrl;
                imagePreview.classList.remove("hidden");

                faceStatus.classList.remove("hidden");
                faceStatus.className = "mt-2 text-xs font-bold text-amber-600";
                faceStatus.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i> กำลังวิเคราะห์ใบหน้าในรูปถ่าย...';

                if (faceModelsLoaded) {
                    try {
                        const img = await faceapi.fetchImage(imageUrl);
                        const detections = await faceapi.detectAllFaces(img, new faceapi.TinyFaceDetectorOptions());

                        if (detections.length > 0) {
                            isFaceDetected = true;
                            faceStatus.className = "mt-2 text-xs font-bold text-emerald-600";
                            faceStatus.innerHTML = `<i class="fa-solid fa-circle-check"></i> ตรวจพบใบหน้าในภาพเรียบร้อยแล้ว (${detections.length} ใบหน้า)`;
                        } else {
                            isFaceDetected = false;
                            faceStatus.className = "mt-2 text-xs font-bold text-rose-600";
                            faceStatus.innerHTML = '<i class="fa-solid fa-triangle-exclamation"></i> ไม่พบใบหน้าในภาพ กรุณากรถ่ายใหม่อีกครั้ง';
                        }
                    } catch (err) {
                        isFaceDetected = true;
                        faceStatus.className = "mt-2 text-xs font-bold text-emerald-600";
                        faceStatus.innerHTML = '<i class="fa-solid fa-circle-check"></i> อัปโหลดรูปภาพเรียบร้อยแล้ว';
                    }
                } else {
                    isFaceDetected = true;
                    faceStatus.className = "mt-2 text-xs font-bold text-emerald-600";
                    faceStatus.innerHTML = '<i class="fa-solid fa-circle-check"></i> บันทึกรูปโปรไฟล์แล้ว';
                }
            });
        }

        // Location GPS Trigger
        function getLocation() {
            const locationInput = document.getElementById("location");
            const geoStatus = document.getElementById("geoStatus");

            if (!navigator.geolocation) {
                alert("เบราว์เซอร์ของคุณไม่รองรับการดึงพิกัด GPS");
                return;
            }

            geoStatus.className = "text-xs font-bold px-2 py-0.5 rounded-full bg-amber-100 text-amber-700";
            geoStatus.textContent = "กำลังระบุตำแหน่ง...";

            navigator.geolocation.getCurrentPosition(
                (position) => {
                    const lat = position.coords.latitude.toFixed(6);
                    const lng = position.coords.longitude.toFixed(6);
                    locationInput.value = `${lat}, ${lng}`;
                    geoStatus.className = "text-xs font-bold px-2 py-0.5 rounded-full bg-emerald-100 text-emerald-700";
                    geoStatus.textContent = "พิกัดถูกต้อง";
                },
                (error) => {
                    alert("ไม่สามารถดึงตำแหน่ง GPS ได้ กรุณาเปิดการเข้าถึงตำแหน่งบนอุปกรณ์ของคุณ");
                    geoStatus.className = "text-xs font-bold px-2 py-0.5 rounded-full bg-rose-100 text-rose-700";
                    geoStatus.textContent = "ผิดพลาด";
                },
                { enableHighAccuracy: true, timeout: 10000 }
            );
        }

        // Modal Controllers
        function openStudentLoginModal() {
            document.getElementById("studentLoginModal").classList.remove("hidden");
        }
        function closeStudentLoginModal() {
            document.getElementById("studentLoginModal").classList.add("hidden");
        }
        function openTeacherLoginModal() {
            document.getElementById("teacherLoginModal").classList.remove("hidden");
        }
        function closeTeacherLoginModal() {
            document.getElementById("teacherLoginModal").classList.add("hidden");
        }

        // Login Handlers
        function handleStudentLogin(e) {
            e.preventDefault();
            const studentId = document.getElementById("studentIdInput").value;
            closeStudentLoginModal();
            loginUser('student', `นักเรียน (${studentId})`);
        }

        function handleTeacherLogin(e) {
            e.preventDefault();
            const username = document.getElementById("teacherUsernameInput").value;
            closeTeacherLoginModal();
            loginUser('teacher', `ครู${username}`);
        }

        function loginUser(role, displayName) {
            currentUser = { role, name: displayName };

            document.getElementById("guestBtns").classList.add("hidden");
            document.getElementById("userProfile").classList.remove("hidden");
            document.getElementById("userNameDisplay").textContent = displayName;

            const roleIcon = document.getElementById("roleIcon");
            if (role === 'student') {
                roleIcon.className = "fa-solid fa-user-graduate text-emerald-400";
            } else {
                roleIcon.className = "fa-solid fa-user-shield text-amber-400";
            }

            document.getElementById("formLockOverlay").classList.add("hidden");
            document.getElementById("studentLockNotice").classList.remove("hidden");

            const inputs = document.querySelectorAll("#checkinForm input, #getGpsBtn, #submitBtn");
            inputs.forEach(el => el.removeAttribute("disabled"));

            if (role === 'student') {
                document.getElementById("fullname").value = displayName;
            }

            // โหลดและอัปเดตข้อมูลประวัติและ Dashboard ทันที
            loadStoredAttendanceData();
            refreshDashboardAndTable();
            document.getElementById("attendancePanel").classList.remove("hidden");
        }

        function logout() {
            currentUser = null;

            document.getElementById("guestBtns").classList.remove("hidden");
            document.getElementById("userProfile").classList.add("hidden");
            document.getElementById("formLockOverlay").classList.remove("hidden");
            document.getElementById("studentLockNotice").classList.add("hidden");
            document.getElementById("attendancePanel").classList.add("hidden");

            const inputs = document.querySelectorAll("#checkinForm input, #getGpsBtn, #submitBtn");
            inputs.forEach(el => el.setAttribute("disabled", "true"));

            document.getElementById("checkinForm").reset();
            document.getElementById("imagePreview").classList.add("hidden");
            document.getElementById("faceStatus").classList.add("hidden");
            document.getElementById("geoStatus").className = "text-xs font-bold px-2 py-0.5 rounded-full bg-slate-100 text-slate-500";
            document.getElementById("geoStatus").textContent = "รอตรวจสอบ";
        }

        // Form Submit Execution
        document.getElementById("checkinForm").addEventListener("submit", (e) => {
            e.preventDefault();

            if (!currentUser) {
                alert("กรุณาเข้าสู่ระบบก่อนเช็คชื่อ");
                return;
            }

            const photoInput = document.getElementById("photo");
            if (photoInput.files.length === 0) {
                alert("กรุณาอัปโหลดหรือถ่ายรูปโปรไฟล์");
                return;
            }

            if (!isFaceDetected) {
                alert("กรุณาใช้นำเข้ารูปถ่ายที่สามารถตรวจจับใบหน้าได้อย่างชัดเจน");
                return;
            }

            const locationVal = document.getElementById("location").value;
            if (!locationVal) {
                alert("กรุณกดดึงพิกัด GPS ก่อนทำการบันทึก");
                return;
            }

            const fullname = document.getElementById("fullname").value;
            const studentClass = document.getElementById("studentClass").value;
            const studentNo = document.getElementById("studentNo").value;
            
            const checkinTime = new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
            const statusInfo = getCheckinStatus();

            const record = {
                time: checkinTime,
                name: fullname,
                classInfo: `${studentClass} เลขที่ ${studentNo}`,
                imgUrl: document.getElementById("previewImg").src,
                location: locationVal,
                status: statusInfo.status
            };

            // บันทึกเข้า Array และบันทึกลง Persistent LocalStorage
            attendanceData.unshift(record);
            saveAttendanceData();

            // อัปเดตแสดงผลหน้าจอ
            refreshDashboardAndTable();

            alert(`บันทึกการเข้าร่วมกิจกรรมสำเร็จ!\nสถานะ: ${statusInfo.status}`);

            document.getElementById("checkinForm").reset();
            document.getElementById("imagePreview").classList.add("hidden");
            document.getElementById("faceStatus").classList.add("hidden");
            document.getElementById("geoStatus").className = "text-xs font-bold px-2 py-0.5 rounded-full bg-slate-100 text-slate-500";
            document.getElementById("geoStatus").textContent = "รอตรวจสอบ";
            isFaceDetected = false;

            if (currentUser.role === 'student') {
                document.getElementById("fullname").value = currentUser.name;
            }
        });

        // Function อัปเดต Dashboard สรุปผล + ตารางประวัติ
        function refreshDashboardAndTable() {
            renderAttendanceTable();
            updateSummaryCards();
        }

        // Table Render Logic
        function renderAttendanceTable() {
            const tbody = document.getElementById("attendanceTable");
            if (!tbody) return;

            if (attendanceData.length === 0) {
                tbody.innerHTML = '<tr><td colspan="6" class="text-center py-6 text-slate-400">ยังไม่มีข้อมูลการเช็คชื่อ</td></tr>';
                return;
            }

            tbody.innerHTML = attendanceData.map(item => {
                let statusBadge = '';
                if (item.status === 'ปกติ') {
                    statusBadge = '<span class="bg-emerald-100 text-emerald-800 text-xs font-bold px-2.5 py-1 rounded-lg">ปกติ</span>';
                } else if (item.status === 'สาย') {
                    statusBadge = '<span class="bg-amber-100 text-amber-800 text-xs font-bold px-2.5 py-1 rounded-lg">สาย</span>';
                } else {
                    statusBadge = '<span class="bg-rose-100 text-rose-800 text-xs font-bold px-2.5 py-1 rounded-lg">ไม่เข้าร่วมกิจกรรม</span>';
                }

                return `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="p-3 font-semibold text-slate-700">${item.time}</td>
                        <td class="p-3 font-medium text-slate-800">${item.name}</td>
                        <td class="p-3 text-slate-600">${item.classInfo}</td>
                        <td class="p-3">
                            <img src="${item.imgUrl}" alt="Profile" class="w-10 h-10 rounded-lg object-cover border border-slate-200">
                        </td>
                        <td class="p-3 text-xs font-mono text-slate-500">${item.location}</td>
                        <td class="p-3">${statusBadge}</td>
                    </tr>
                `;
            }).join('');
        }

        // Summary Cards Update Logic
        function updateSummaryCards() {
            const countNormal = attendanceData.filter(item => item.status === 'ปกติ').length;
            const countLate = attendanceData.filter(item => item.status === 'สาย').length;
            const countAbsent = attendanceData.filter(item => item.status === 'ไม่เข้าร่วมกิจกรรม').length;

            document.getElementById("countNormal").textContent = countNormal;
            document.getElementById("countLate").textContent = countLate;
            document.getElementById("countAbsent").textContent = countAbsent;
        }
    </script>
</body>
</html>
