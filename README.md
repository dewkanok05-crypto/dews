<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prite Helper - Sheets & Arduino</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap');
        body { font-family: 'Kanit', sans-serif; }
        .glass { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); }
    </style>
</head>
<body class="bg-gray-100 min-h-screen">

    <div class="max-w-4xl mx-auto p-4 sm:p-6">
        <!-- Header -->
        <header class="text-center mb-8">
            <h1 class="text-3xl font-bold text-blue-600 mb-2">
                <i class="fas fa-robot mr-2"></i>Prite Helper
            </h1>
            <p class="text-gray-600">ผู้ช่วยจัดการ Sheets และโค้ด Arduino</p>
        </header>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            
            <!-- Section 1: Speech to Text for Sheets -->
            <div class="glass p-6 rounded-2xl shadow-lg border border-white">
                <h2 class="text-xl font-semibold mb-4 text-green-600 flex items-center">
                    <i class="fas fa-microphone-alt mr-2"></i>ผู้ช่วยเสียงสำหรับ Sheets
                </h2>
                <div class="space-y-4">
                    <div class="relative">
                        <textarea id="stt-output" rows="4" class="w-full p-3 rounded-xl border border-gray-200 focus:ring-2 focus:ring-green-400 outline-none" placeholder="ผลลัพธ์จากการพูดจะปรากฏที่นี่..."></textarea>
                        <button id="copy-btn" class="absolute top-2 right-2 text-gray-400 hover:text-green-600">
                            <i class="fas fa-copy"></i>
                        </button>
                    </div>
                    
                    <div class="flex gap-2">
                        <button id="start-stt" class="flex-1 bg-green-500 hover:bg-green-600 text-white font-medium py-2 px-4 rounded-xl transition-all flex items-center justify-center">
                            <i class="fas fa-play mr-2"></i>เริ่มพูด
                        </button>
                        <button id="stop-stt" class="bg-red-500 hover:bg-red-600 text-white py-2 px-4 rounded-xl hidden">
                            <i class="fas fa-stop"></i>
                        </button>
                    </div>

                    <div class="pt-4 border-t border-gray-100">
                        <label class="block text-sm font-medium text-gray-700 mb-2">ทดสอบเสียงอ่าน (TTS):</label>
                        <div class="flex gap-2">
                            <input type="text" id="tts-input" class="flex-1 p-2 rounded-lg border border-gray-200" placeholder="พิมพ์ข้อความให้อ่าน...">
                            <button id="play-tts" class="bg-blue-500 hover:bg-blue-600 text-white p-2 rounded-lg">
                                <i class="fas fa-volume-up"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Section 2: Arduino Chatbot -->
            <div class="glass p-6 rounded-2xl shadow-lg border border-white flex flex-col h-[500px]">
                <h2 class="text-xl font-semibold mb-2 text-blue-600 flex items-center">
                    <i class="fas fa-microchip mr-2"></i>Arduino Sensor Bot
                </h2>
                
                <div class="mb-2">
                    <input type="password" id="api-key" class="w-full p-2 text-xs border rounded-lg" placeholder="ใส่ Gemini API Key ที่นี่ (ถ้ามี)">
                </div>

                <div id="chat-box" class="flex-1 overflow-y-auto mb-4 p-3 bg-gray-50 rounded-xl space-y-3 text-sm">
                    <div class="bg-blue-100 p-2 rounded-lg text-blue-800">
                        สวัสดีครับคุณไปร์ท! อยากให้ช่วยเขียนโค้ดคุม Sensor ตัวไหน (Ultrasonic, DHT11, LDR ฯลฯ) บอกได้เลยครับ
                    </div>
                </div>

                <div class="flex gap-2">
                    <input type="text" id="user-msg" class="flex-1 p-2 border rounded-xl focus:ring-2 focus:ring-blue-400 outline-none" placeholder="ถามเรื่อง Sensor...">
                    <button id="send-msg" class="bg-blue-600 hover:bg-blue-700 text-white px-4 rounded-xl">
                        <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
            </div>

        </div>

        <!-- System Message -->
        <div id="status-msg" class="fixed bottom-4 right-4 bg-gray-800 text-white px-4 py-2 rounded-lg text-sm opacity-0 transition-opacity"></div>
    </div>

    <script>
        // --- API & Utility ---
        const apiKeyInput = document.getElementById('api-key');
        const statusMsg = document.getElementById('status-msg');

        function showStatus(text) {
            statusMsg.innerText = text;
            statusMsg.style.opacity = '1';
            setTimeout(() => statusMsg.style.opacity = '0', 2000);
        }

        // --- Speech to Text (STT) ---
        const recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (recognition) {
            const rec = new recognition();
            rec.lang = 'th-TH';
            rec.continuous = true;
            rec.interimResults = true;

            const sttOutput = document.getElementById('stt-output');
            const startBtn = document.getElementById('start-stt');
            const stopBtn = document.getElementById('stop-stt');

            rec.onresult = (event) => {
                let text = '';
                for (let i = event.resultIndex; i < event.results.length; i++) {
                    text += event.results[i][0].transcript;
                }
                sttOutput.value = text;
            };

            startBtn.onclick = () => {
                rec.start();
                startBtn.classList.add('hidden');
                stopBtn.classList.remove('hidden');
                showStatus('กำลังบันทึกเสียง...');
            };

            stopBtn.onclick = () => {
                rec.stop();
                startBtn.classList.remove('hidden');
                stopBtn.classList.add('hidden');
                showStatus('หยุดบันทึก');
            };
        } else {
            document.getElementById('start-stt').disabled = true;
            document.getElementById('start-stt').innerText = 'เบราว์เซอร์ไม่รองรับเสียง';
        }

        // --- Text to Speech (TTS) ---
        document.getElementById('play-tts').onclick = () => {
            const text = document.getElementById('tts-input').value || document.getElementById('stt-output').value;
            if (!text) return;
            const uttr = new SpeechSynthesisUtterance(text);
            uttr.lang = 'th-TH';
            window.speechSynthesis.speak(uttr);
        };

        // --- Copy Logic ---
        document.getElementById('copy-btn').onclick = () => {
            const text = document.getElementById('stt-output').value;
            if (!text) return;
            const tempInput = document.createElement("textarea");
            tempInput.value = text;
            document.body.appendChild(tempInput);
            tempInput.select();
            document.execCommand("copy");
            document.body.removeChild(tempInput);
            showStatus('คัดลอกลงคลิปบอร์ดแล้ว');
        };

        // --- Chatbot Logic (Gemini API Integration) ---
        const chatBox = document.getElementById('chat-box');
        const userMsgInput = document.getElementById('user-msg');
        const sendBtn = document.getElementById('send-msg');

        async function callGemini(prompt) {
            const apiKey = apiKeyInput.value.trim();
            if (!apiKey) {
                return "กรุณาใส่ API Key ของ Gemini ก่อนเพื่อใช้งานระบบแชทครับ";
            }

            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = {
                contents: [{ 
                    parts: [{ 
                        text: `คุณคือ 'น้องไปร์ท' ผู้เชี่ยวชาญ Arduino ตอบคำถามด้วยภาษาไทยที่สุภาพและเป็นกันเอง อธิบายการต่อวงจรและเขียนโค้ดสำหรับ Sensor ต่างๆ คำถามคือ: ${prompt}` 
                    }] 
                }]
            };

            try {
                let response;
                let retries = 0;
                const backoffs = [1000, 2000, 4000, 8000, 16000];

                while (retries < 5) {
                    response = await fetch(url, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (response.ok) break;
                    await new Promise(r => setTimeout(r, backoffs[retries++]));
                }

                const data = await response.json();
                return data.candidates?.[0]?.content?.parts?.[0]?.text || "ขออภัยครับ เกิดข้อผิดพลาดในการประมวลผล";
            } catch (err) {
                return "ไม่สามารถเชื่อมต่อกับ AI ได้ในขณะนี้";
            }
        }

        async function handleChat() {
            const msg = userMsgInput.value.trim();
            if (!msg) return;

            // User Message
            const userDiv = document.createElement('div');
            userDiv.className = 'bg-gray-200 p-2 rounded-lg ml-8 text-gray-800';
            userDiv.innerText = msg;
            chatBox.appendChild(userDiv);
            userMsgInput.value = '';
            chatBox.scrollTop = chatBox.scrollHeight;

            // Loading
            const loadingDiv = document.createElement('div');
            loadingDiv.className = 'bg-blue-50 p-2 rounded-lg mr-8 text-blue-500 italic';
            loadingDiv.innerText = 'กำลังคิดคำตอบ...';
            chatBox.appendChild(loadingDiv);

            // AI Response
            const aiResponse = await callGemini(msg);
            chatBox.removeChild(loadingDiv);
            
            const aiDiv = document.createElement('div');
            aiDiv.className = 'bg-blue-100 p-2 rounded-lg mr-8 text-blue-800 whitespace-pre-wrap';
            aiDiv.innerText = aiResponse;
            chatBox.appendChild(aiDiv);
            chatBox.scrollTop = chatBox.scrollHeight;
        }

        sendBtn.onclick = handleChat;
        userMsgInput.onkeypress = (e) => { if (e.key === 'Enter') handleChat(); };

    </script>
</body>
</html># dews
00
