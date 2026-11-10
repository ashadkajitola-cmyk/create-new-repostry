<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EduAI Portal & Wallet System</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { background: #f0f2f5; color: #333; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        .container { width: 100%; max-width: 480px; background: #fff; min-height: 100vh; box-shadow: 0 0 20px rgba(0,0,0,0.1); display: flex; flex-direction: column; position: relative; overflow-x: hidden; }
        header { background: #1a73e8; color: #fff; padding: 15px 20px; display: flex; justify-content: space-between; align-items: center; }
        header h2 { font-size: 1.2rem; }
        .btn { background: #1a73e8; color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; font-weight: 600; font-size: 0.95rem; }
        .btn:active { opacity: 0.8; }
        .btn-danger { background: #d93025; }
        .btn-success { background: #188038; }
        .btn-secondary { background: #5f6368; }
        .screen { padding: 20px; flex: 1; display: none; flex-direction: column; gap: 15px; overflow-y: auto; }
        .screen.active { display: flex; }
        input, select { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 6px; font-size: 1rem; outline: none; }
        input:focus { border-color: #1a73e8; }
        .card { background: #f8f9fa; border: 1px solid #dadce0; border-radius: 8px; padding: 15px; display: flex; flex-direction: column; gap: 10px; }
        .wallet-box { background: linear-gradient(135deg, #1a73e8, #4285f4); color: white; padding: 20px; border-radius: 12px; text-align: center; }
        .wallet-box h3 { font-size: 2rem; margin-top: 5px; }
        .codes-info { font-size: 0.9rem; background: rgba(255,255,255,0.2); padding: 8px; border-radius: 6px; margin-top: 10px; display: flex; justify-content: space-around; }
        .nav-bottom { display: flex; background: #fff; border-top: 1px solid #dadce0; position: sticky; bottom: 0; width: 100%; }
        .nav-tab { flex: 1; text-align: center; padding: 12px 5px; font-size: 0.8rem; color: #5f6368; cursor: pointer; border: none; background: none; }
        .nav-tab.active { color: #1a73e8; font-weight: bold; border-top: 2px solid #1a73e8; }
        .green-num { color: #188038; font-weight: bold; }
        .red-num { color: #d93025; font-weight: bold; }
        .history-item { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #eee; padding-bottom: 8px; margin-bottom: 8px; }
        .chat-box { flex: 1; border: 1px solid #ccc; border-radius: 8px; padding: 10px; height: 300px; overflow-y: auto; background: #fff; display: flex; flex-direction: column; gap: 10px; }
        .msg { padding: 10px; border-radius: 8px; max-width: 80%; }
        .msg.user { background: #e8f0fe; align-self: flex-end; }
        .msg.bot { background: #f1f3f4; align-self: flex-start; }
    </style>
</head>
<body>

<div class="container">
    <!-- Screen 1: Auth (Login / Register) -->
    <div id="authScreen" class="screen active">
        <h2>EduAI Portal Login / Register</h2>
        <p style="color: #666; font-size: 0.9rem;">Apna Mobile Number dalein. Agar pehle se account hai toh PIN automatically set rahega.</p>
        <div class="card">
            <label>Mobile Number</label>
            <input type="tel" id="authPhone" placeholder="Mobile Number dalein" maxlength="10">
            <div id="nameFieldContainer" style="display:none;">
                <label>Aapka Naam</label>
                <input type="text" id="authName" placeholder="Poora Naam">
            </div>
            <div id="pinFieldContainer" style="display:none;">
                <label>3-Digit PIN Set Karein</label>
                <input type="password" id="authPin" placeholder="3-digit PIN" maxlength="3">
            </div>
            <button class="btn" id="authBtn" onclick="handleAuth()">Aage Badhein</button>
        </div>
    </div>

    <!-- Main App Screen (With Navigation Tabs) -->
    <div id="mainAppScreen" class="container" style="display:none; max-width: 100%; box-shadow: none;">
        <header>
            <h2 id="headerTitle">EduAI Hub</h2>
            <button class="btn btn-danger" style="padding: 5px 10px; font-size: 0.8rem;" onclick="logout()">Logout</button>
        </header>

        <!-- Tab 1: Home / Education Q&A -->
        <div id="tabHome" class="screen active" style="padding-bottom: 60px;">
            <div class="wallet-box">
                <p>Aapka Wallet Balance</p>
                <h3 id="walletBalance">₹25</h3>
                <div class="codes-info">
                    <span>PIN: <strong id="displayPin">***</strong></span>
                    <span>5-Digit Code: <strong id="displayCode">-----</strong></span>
                </div>
            </div>

            <div class="card">
                <h4>AI Education Search & Voice Assistant</h4>
                <p style="font-size: 0.85rem; color: #d93025;" id="rechargeWarningMsg">⚠️ Sawaal poochne ke liye active recharge hona zaroori hai!</p>
                <div style="display: flex; gap: 5px;">
                    <input type="text" id="aiQuery" placeholder="Kuch bhi sawaal poochein...">
                    <button class="btn" onclick="askAI()">Search</button>
                    <button class="btn btn-secondary" onclick="startVoiceInput()" title="Voice input">🎤</button>
                </div>
                <div class="chat-box" id="chatBox">
                    <div class="msg bot">Hello! Main aapka AI Education assistant hoon. Koi bhi sawaal pooch sakte hain.</div>
                </div>
            </div>
        </div>

        <!-- Tab 2: Recharge Store -->
        <div id="tabRecharge" class="screen" style="padding-bottom: 60px;">
            <h3>Recharge Store</h3>
            <div id="adminRechargePanel" style="display:none;" class="card">
                <h4>Admin Panel: Naya Recharge Banayein</h4>
                <input type="text" id="newPlanName" placeholder="Plan Naam ( jaise: 1 Month Unlimited )">
                <input type="number" id="newPlanPrice" placeholder="Amount ( ₹ )">
                <input type="number" id="newPlanValidity" placeholder="Validity ( Din me )">
                <button class="btn btn-success" onclick="adminCreateRecharge()">Recharge Plan Banayein</button>
            </div>

            <div id="rechargePlansList" style="display: flex; flex-direction: column; gap: 10px;">
                <!-- Dynamic Plans -->
            </div>

            <div class="card" style="margin-top: 15px;">
                <h4>13-Digit Code Activate Karein</h4>
                <input type="text" id="activationCodeInput" placeholder="13-Digit Code Dalein" maxlength="13">
                <button class="btn btn-success" onclick="activateRechargeCode()">Active Karein</button>
            </div>
        </div>

        <!-- Tab 3: Send Money -->
        <div id="tabSendMoney" class="screen" style="padding-bottom: 60px;">
            <h3>Pise Bhejein (Send Money)</h3>
            <div class="card">
                <label>Receiver ka 5-Digit Code</label>
                <input type="text" id="sendCodeInput" placeholder="5-digit code dalein" maxlength="5" onkeyup="checkReceiverCode(this.value)">
                <p id="receiverNameDisplay" style="font-weight: bold; color: #1a73e8;"></p>
                
                <label>Amount (₹)</label>
                <input type="number" id="sendAmountInput" placeholder="Kitna bhejna hai" oninput="formatAmountGreen(this)">
                
                <label>Aapka 3-Digit PIN</label>
                <input type="password" id="sendPinInput" placeholder="3-digit PIN" maxlength="3">
                
                <button class="btn btn-success" onclick="sendMoneyAction()">Payment Karein</button>
            </div>
        </div>

        <!-- Tab 4: History -->
        <div id="tabHistory" class="screen" style="padding-bottom: 60px;">
            <h3>Transaction History</h3>
            <div class="card" id="historyListContainer">
                <!-- History Items -->
            </div>
        </div>

        <!-- Bottom Navigation -->
        <div class="nav-bottom">
            <button class="nav-tab active" onclick="switchTab('tabHome', this)">Home</button>
            <button class="nav-tab" onclick="switchTab('tabRecharge', this)">Recharge</button>
            <button class="nav-tab" onclick="switchTab('tabSendMoney', this)">Send Money</button>
            <button class="nav-tab" onclick="switchTab('tabHistory', this)">History</button>
        </div>
    </div>
</div>

<script>
    // Database Simulation using localStorage
    let db = JSON.parse(localStorage.getItem('edu_wallet_db')) || {
        users: {},
        recharges: [
            { id: 1, name: "Basic Education Plan", price: 99, validity: 30 },
            { id: 2, name: "Pro Unlimited Plan", price: 299, validity: 90 }
        ],
        history: []
    };

    let currentUserPhone = localStorage.getItem('edu_current_user') || null;

    // Check on load
    window.onload = function() {
        if (currentUserPhone && db.users[currentUserPhone]) {
            showMainApp();
        } else {
            // Setup default admin if not exists
            if(!db.users["7355055313"]) {
                db.users["7355055313"] = {
                    name: "Admin (Ashad)",
                    phone: "7355055313",
                    pin: "123",
                    code: "10001",
                    balance: 500000,
                    activeRecharge: true,
                    validityDate: "2099-12-31"
                };
                saveDB();
            }
        }
    };

    // Phone input handler for Auth
    const phoneInput = document.getElementById('authPhone');
    phoneInput.addEventListener('input', function() {
        let phone = this.value.trim();
        let nameContainer = document.getElementById('nameFieldContainer');
        let pinContainer = document.getElementById('pinFieldContainer');
        let authBtn = document.getElementById('authBtn');

        if(phone.length === 10) {
            if(db.users[phone]) {
                // User already exists, PIN is already set automatically!
                nameContainer.style.display = 'none';
                pinContainer.style.display = 'none';
                authBtn.innerText = "Login Karein (PIN Automatically Set)";
            } else {
                // New User Registration
                nameContainer.style.display = 'block';
                pinContainer.style.display = 'block';
                authBtn.innerText = "Account Banayein (+₹25 Free Bonus)";
            }
        } else {
            nameContainer.style.display = 'none';
            pinContainer.style.display = 'none';
        }
    });

    function handleAuth() {
        let phone = document.getElementById('authPhone').value.trim();
        if(phone.length !== 10) {
            alert("Kripya 10 ank ka valid mobile number dalein.");
            return;
        }

        if(db.users[phone]) {
            // Existing user login
            currentUserPhone = phone;
            localStorage.setItem('edu_current_user', phone);
            showMainApp();
        } else {
            // New user registration
            let name = document.getElementById('authName').value.trim();
            let pin = document.getElementById('authPin').value.trim();
            if(!name || pin.length !== 3) {
                alert("Kripya poora naam aur 3-digit ka PIN zaroor dalein.");
                return;
            }

            let fiveDigitCode = Math.floor(10000 + Math.random() * 90000).toString();
            
            // Check admin bonus
            let bonus = (phone === "7355055313") ? 500000 : 25;

            db.users[phone] = {
                name: name,
                phone: phone,
                pin: pin,
                code: fiveDigitCode,
                balance: bonus,
                activeRecharge: (phone === "7355055313"),
                validityDate: (phone === "7355055313") ? "2099-12-31" : null
            };

            // History entry for bonus
            db.history.unshift({
                type: 'credit',
                amount: bonus,
                name: "Welcome Bonus",
                targetPhone: phone,
                date: new Date().toLocaleString()
            });

            saveDB();
            currentUserPhone = phone;
            localStorage.setItem('edu_current_user', phone);
            showMainApp();
        }
    }

    function showMainApp() {
        document.getElementById('authScreen').classList.remove('active');
        document.getElementById('mainAppScreen').style.display = 'flex';
        
        let user = db.users[currentUserPhone];
        document.getElementById('headerTitle').innerText = `Welcome, ${user.name}`;
        document.getElementById('walletBalance').innerText = `₹${user.balance}`;
        document.getElementById('displayPin').innerText = user.pin;
        document.getElementById('displayCode').innerText = user.code;

        // Check Admin panel
        if(currentUserPhone === "7355055313") {
            document.getElementById('adminRechargePanel').style.display = 'block';
        }

        // Check recharge warning
        if(user.activeRecharge) {
            document.getElementById('rechargeWarningMsg').style.display = 'none';
        } else {
            document.getElementById('rechargeWarningMsg').style.display = 'block';
        }

        renderRecharges();
        renderHistory();
    }

    function logout() {
        currentUserPhone = null;
        localStorage.removeItem('edu_current_user');
        document.getElementById('mainAppScreen').style.display = 'none';
        document.getElementById('authScreen').classList.add('active');
        document.getElementById('authPhone').value = '';
    }

    function switchTab(tabId, btnElem) {
        document.querySelectorAll('#mainAppScreen .screen').forEach(s => s.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        document.querySelectorAll('.nav-tab').forEach(t => t.classList.remove('active'));
        btnElem.classList.add('active');
    }

    // AI Education & Voice Assistant Logic
    function askAI() {
        let user = db.users[currentUserPhone];
        if(!user.activeRecharge) {
            alert("Aapka recharge active nahi hai! Sawaal poochne ke liye pehle Recharge Store se recharge karein.");
            switchTab('tabRecharge', document.querySelectorAll('.nav-tab')[1]);
            return;
        }

        let queryBox = document.getElementById('aiQuery');
        let question = queryBox.value.trim();
        if(!question) return;

        appendChat(question, 'user');
        queryBox.value = '';

        // Generate smart educational answer
        setTimeout(() => {
            let answer = getSmartEducationAnswer(question);
            appendChat(answer, 'bot');
            speakText(answer);
        }, 500);
    }

    function getSmartEducationAnswer(q) {
        q = q.toLowerCase();
        if(q.includes('math') || q.includes('ganit')) return "Mathematics numbers, structures, space aur change ka study hai. Jaise Pythagoras theorem: a² + b² = c².";
        if(q.includes('science') || q.includes('vigyan')) return "Science physical aur natural world ka observation aur experiment ke zariye systematic study hai.";
        if(q.includes('history') || q.includes('itihas')) return "History past events ki study hai, jo humein purani galtiyon se sikhane me madad karti hai.";
        return `Aapne sawaal poocha: "${q}". Education ke mutabiq, iska behtareen jawab ye hai ki har concept ko step-by-step samajhna zaroori hai. Kripya apna specific topic batayein!`;
    }

    function appendChat(text, sender) {
        let chatBox = document.getElementById('chatBox');
        let div = document.createElement('div');
        div.className = `msg ${sender}`;
        div.innerText = text;
        chatBox.appendChild(div);
        chatBox.scrollTop = chatBox.scrollHeight;
    }

    function startVoiceInput() {
        let user = db.users[currentUserPhone];
        if(!user.activeRecharge) {
            alert("Recharge active hona zaroori hai!");
            return;
        }
        if (!('webkitSpeechRecognition' in window) && !('SpeechRecognition' in window)) {
            alert("Aapka browser Speech Recognition support nahi karta.");
            return;
        }
        let SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        let recognition = new SpeechRecognition();
        recognition.lang = 'hi-IN';
        recognition.onresult = function(event) {
            let speechText = event.results[0][0].transcript;
            document.getElementById('aiQuery').value = speechText;
            askAI();
        };
        recognition.start();
    }

    function speakText(text) {
        if ('speechSynthesis' in window) {
            let utterance = new SpeechSynthesisUtterance(text);
            utterance.lang = 'hi-IN';
            window.speechSynthesis.speak(utterance);
        }
    }

    // Recharge Store Functions
    function renderRecharges() {
        let listContainer = document.getElementById('rechargePlansList');
        listContainer.innerHTML = '';
        db.recharges.forEach(plan => {
            let div = document.createElement('div');
            div.className = 'card';
            div.innerHTML = `
                <h4>${plan.name}</h4>
                <p>Price: <strong>₹${plan.price}</strong> | Validity: <strong>${plan.validity} Days</strong></p>
                <div style="display:flex; gap:10px;">
                    <button class="btn btn-success" onclick="buyRecharge(${plan.id})">Pay ₹${plan.price}</button>
                    ${currentUserPhone === "7355055313" ? `<button class="btn btn-danger" onclick="deleteRecharge(${plan.id})">Delete</button>` : ''}
                </div>
            `;
            listContainer.appendChild(div);
        });
    }

    function adminCreateRecharge() {
        let name = document.getElementById('newPlanName').value.trim();
        let price = parseFloat(document.getElementById('newPlanPrice').value);
        let validity = parseInt(document.getElementById('newPlanValidity').value);

        if(!name || !price || !validity) {
            alert("Sabhi fields bharein.");
            return;
        }

        db.recharges.push({ id: Date.now(), name, price, validity });
        saveDB();
        renderRecharges();
        alert("Naya recharge plan safalपूर्वक ban gaya!");
        document.getElementById('newPlanName').value = '';
        document.getElementById('newPlanPrice').value = '';
        document.getElementById('newPlanValidity').value = '';
    }

    function deleteRecharge(id) {
        db.recharges = db.recharges.filter(p => p.id !== id);
        saveDB();
        renderRecharges();
    }

    function buyRecharge(planId) {
        let plan = db.recharges.find(p => p.id === planId);
        let user = db.users[currentUserPhone];

        let pin = prompt("Apna 3-digit PIN dalein payment confirm karne ke liye:");
        if(pin !== user.pin) {
            alert("Galat PIN!");
            return;
        }

        if(user.balance < plan.price) {
            alert("Wallet me balance kam hai! Kripya paise add karwayein.");
            return;
        }

        user.balance -= plan.price;
        
        // Admin (7355055313) ko paise milenge
        if(currentUserPhone !== "7355055313" && db.users["7355055313"]) {
            db.users["7355055313"].balance += plan.price;
        }

        // Generate 13-digit code
        let thirteenCode = Math.floor(1000000000000 + Math.random() * 9000000000000).toString();
        
        // Add to history
        db.history.unshift({
            type: 'debit',
            amount: plan.price,
            name: `Recharge Purchase (${plan.name}) - Code: ${thirteenCode}`,
            targetPhone: currentUserPhone,
            date: new Date().toLocaleString()
        });

        saveDB();
        showMainApp();
        alert(`Payment successful!\nAapka 13-digit recharge code ye raha: ${thirteenCode}\nIsse neeche 'Active Karein' box me dalein.`);
    }

    function activateRechargeCode() {
        let code = document.getElementById('activationCodeInput').value.trim();
        if(code.length !== 13) {
            alert("Kripya sahi 13-digit code dalein.");
            return;
        }

        let user = db.users[currentUserPhone];
        user.activeRecharge = true;
        saveDB();
        showMainApp();
        alert("🎉 Badhai ho! Recharge successfully active ho chuka hai. Ab aap AI Education search aur voice assistant use kar sakte hain.");
        document.getElementById('activationCodeInput').value = '';
    }

    // Send Money Functions
    function checkReceiverCode(code) {
        let display = document.getElementById('receiverNameDisplay');
        if(code.length === 5) {
            let foundUser = Object.values(db.users).find(u => u.code === code);
            if(foundUser) {
                if(foundUser.phone === currentUserPhone) {
                    display.innerText = "⚠️ Aap khud ko paise nahi bhej sakte!";
                } else {
                    display.innerText = `Receiver Name: ${foundUser.name}`;
                }
            } else {
                display.innerText = "❌ Is code ka koi user nahi mila.";
            }
        } else {
            display.innerText = "";
        }
    }

    function formatAmountGreen(elem) {
        elem.style.color = "#188038";
        elem.style.fontWeight = "bold";
    }

    function sendMoneyAction() {
        let rCode = document.getElementById('sendCodeInput').value.trim();
        let amount = parseFloat(document.getElementById('sendAmountInput').value);
        let pin = document.getElementById('sendPinInput').value.trim();
        let sender = db.users[currentUserPhone];

        if(pin !== sender.pin) {
            alert("Galat PIN!");
            return;
        }

        let receiver = Object.values(db.users).find(u => u.code === rCode);
        if(!receiver) {
            alert("Galat 5-digit receiver code!");
            return;
        }

        if(receiver.phone === sender.phone) {
            alert("Aap khud ko paise nahi bhej sakte!");
            return;
        }

        if(isNaN(amount) || amount <= 0) {
            alert("Sahi amount dalein.");
            return;
        }

        if(sender.balance < amount) {
            alert("Aapke wallet me utne paise nahi hain!");
            return;
        }

        sender.balance -= amount;
        receiver.balance += amount;

        // History logs
        db.history.unshift({
            type: 'debit',
            amount: amount,
            name: `Sent to ${receiver.name} (${receiver.code})`,
            targetPhone: sender.phone,
            date: new Date().toLocaleString()
        });

        db.history.unshift({
            type: 'credit',
            amount: amount,
            name: `Received from ${sender.name} (${sender.code})`,
            targetPhone: receiver.phone,
            date: new Date().toLocaleString()
        });

        saveDB();
        showMainApp();
        alert(`₹${amount} safalपूर्वक ${receiver.name} ko bhej diye gaye hain!`);
        
        document.getElementById('sendCodeInput').value = '';
        document.getElementById('sendAmountInput').value = '';
        document.getElementById('sendPinInput').value = '';
        document.getElementById('receiverNameDisplay').innerText = '';
    }

    function renderHistory() {
        let container = document.getElementById('historyListContainer');
        container.innerHTML = '<h4>Recent Transactions</h4>';
        
        let userHistory = db.history.filter(h => h.targetPhone === currentUserPhone);
        if(userHistory.length === 0) {
            container.innerHTML += '<p style="color:#666;">Koi transaction history nahi hai.</p>';
            return;
        }

        userHistory.forEach(item => {
            let div = document.createElement('div');
            div.className = 'history-item';
            let isCredit = item.type === 'credit';
            div.innerHTML = `
                <div>
                    <p style="font-size:0.9rem; font-weight:600;">${item.name}</p>
                    <small style="color:#666;">${item.date}</small>
                </div>
                <div class="${isCredit ? 'green-num' : 'red-num'}">
                    ${isCredit ? '+' : '-'}₹${item.amount}
                </div>
            `;
            container.appendChild(div);
        });
    }

    function saveDB() {
        localStorage.setItem('edu_wallet_db', JSON.stringify(db));
    }
</script>

</body>
</html>

