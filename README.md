<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Калькулятор / Calculator</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; background-color: var(--tg-theme-secondary-bg-color, #f0f0f0); color: var(--tg-theme-text-color, #222222); margin: 0; display: flex; flex-direction: column; height: 100vh; user-select: none; }
        .tabs { display: flex; background-color: var(--tg-theme-bg-color, #ffffff); padding: 10px; gap: 10px; justify-content: center; border-bottom: 1px solid var(--tg-theme-hint-color, #cccccc); }
        .tab-btn { background-color: transparent; color: var(--tg-theme-hint-color, #999999); border: none; padding: 8px 12px; font-size: 14px; font-weight: bold; border-radius: 10px; cursor: pointer; }
        .tab-btn.active { background-color: var(--tg-theme-button-color, #3390ec); color: var(--tg-theme-button-text-color, #ffffff); }
        
        #display-container { flex-grow: 1; display: flex; flex-direction: column; align-items: flex-end; justify-content: flex-end; padding: 20px; background-color: var(--tg-theme-secondary-bg-color, #f0f0f0); }
        #history { font-size: 20px; color: var(--tg-theme-hint-color, #999999); min-height: 24px; margin-bottom: 5px; }
        #display { font-size: 48px; text-align: right; word-wrap: break-word; }
        
        .keypad { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; padding: 15px; background-color: var(--tg-theme-bg-color, #ffffff); border-top-left-radius: 20px; border-top-right-radius: 20px; padding-bottom: 20px; }
        button { background-color: var(--tg-theme-secondary-bg-color, #e4e4e5); color: var(--tg-theme-text-color, #000000); border: none; border-radius: 15px; padding: 15px; font-size: 24px; cursor: pointer; transition: 0.1s; }
        button:active { opacity: 0.6; }
        button.operator { color: var(--tg-theme-button-color, #3390ec); font-weight: bold; }
        button.action { background-color: #ff3b30; color: white; }
        
        #eng-row { display: none; grid-column: span 4; display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; margin-bottom: 8px;}
        .eng-btn { font-size: 16px; background-color: var(--tg-theme-hint-color, #cccccc); color: var(--tg-theme-bg-color, #ffffff); }
        
        #converter-view { display: none; padding: 20px; flex-grow: 1; background-color: var(--tg-theme-bg-color, #ffffff); }
        .conv-row { display: flex; gap: 10px; margin-bottom: 15px; }
        .conv-input { flex: 1; width: 50%; padding: 15px; border: 1px solid var(--tg-theme-hint-color, #ccc); border-radius: 10px; background: var(--tg-theme-secondary-bg-color, #f0f0f0); color: var(--tg-theme-text-color, #000); font-size: 18px; box-sizing: border-box; }
        .conv-select { width: 100%; padding: 15px; margin-bottom: 20px; border: 1px solid var(--tg-theme-hint-color, #ccc); border-radius: 10px; background: var(--tg-theme-secondary-bg-color, #e4e4e5); color: var(--tg-theme-text-color, #000); font-size: 16px; font-weight: bold; cursor: pointer; appearance: none; outline: none; }
        .conv-unit-select { flex: 1; width: 50%; padding: 10px; border: 1px solid var(--tg-theme-hint-color, #ccc); border-radius: 10px; background: var(--tg-theme-bg-color, #fff); color: var(--tg-theme-text-color, #000); font-size: 16px; cursor: pointer; outline: none; }
        
        .bottom-actions { display: flex; gap: 10px; padding: 15px; background-color: var(--tg-theme-bg-color, #ffffff); }
        .action-btn { flex: 1; font-size: 14px; padding: 15px; border-radius: 12px; font-weight: bold; color: white; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <div class="tabs">
        <button id="tab-std" class="tab-btn active" onclick="switchMode('standard', this)"></button>
        <button id="tab-eng" class="tab-btn" onclick="switchMode('engineer', this)"></button>
        <button id="tab-cnv" class="tab-btn" onclick="switchMode('converter', this)"></button>
    </div>

    <!-- Режим: Калькулятор -->
    <div id="calc-view" style="display: flex; flex-direction: column; flex-grow: 1;">
        <div id="display-container">
            <div id="history"></div>
            <div id="display">0</div>
        </div>
        <div class="keypad">
            <div id="eng-row" style="display: none;">
                <button class="eng-btn" onclick="calcAction('Math.sin(')">sin</button>
                <button class="eng-btn" onclick="calcAction('Math.cos(')">cos</button>
                <button class="eng-btn" onclick="calcAction('Math.sqrt(')">√</button>
                <button class="eng-btn" onclick="calcAction('**')">x²</button>
            </div>
            <button class="action" onclick="clearDisplay()">C</button>
            <button onclick="calcAction('(')">(</button>
            <button onclick="calcAction(')')">)</button>
            <button class="operator" onclick="calcAction('/')">÷</button>
            <button onclick="calcAction('7')">7</button><button onclick="calcAction('8')">8</button><button onclick="calcAction('9')">9</button><button class="operator" onclick="calcAction('*')">×</button>
            <button onclick="calcAction('4')">4</button><button onclick="calcAction('5')">5</button><button onclick="calcAction('6')">6</button><button class="operator" onclick="calcAction('-')">-</button>
            <button onclick="calcAction('1')">1</button><button onclick="calcAction('2')">2</button><button onclick="calcAction('3')">3</button><button class="operator" onclick="calcAction('+')">+</button>
            <button onclick="calcAction('0')" style="grid-column: span 2;">0</button><button onclick="calcAction('.')">.</button>
            <button class="operator" onclick="calculate()" style="background-color: var(--tg-theme-button-color, #3390ec); color: white;">=</button>
        </div>
    </div>

    <!-- Режим: Продвинутый Конвертер -->
    <div id="converter-view">
        <select id="conv-category" class="conv-select" onchange="updateUnitOptions()"></select>
        
        <div class="conv-row">
            <input type="number" id="conv-input1" class="conv-input" placeholder="0" oninput="doConvert(1)">
            <select id="conv-unit1" class="conv-unit-select" onchange="doConvert(1)"></select>
        </div>
        
        <div style="text-align: center; color: var(--tg-theme-hint-color); margin-bottom: 10px;">=</div>
        
        <div class="conv-row">
            <input type="number" id="conv-input2" class="conv-input" placeholder="0" oninput="doConvert(2)">
            <select id="conv-unit2" class="conv-unit-select" onchange="doConvert(2)"></select>
        </div>
    </div>

    <div class="bottom-actions">
        <button id="share-btn" class="action-btn" style="background-color: #34c759;" onclick="shareResult()"></button>
        <button id="desktop-btn" class="action-btn" style="background-color: var(--tg-theme-button-color, #3390ec);" onclick="addToHomeScreen()"></button>
    </div>

    <script>
        let tg = window.Telegram.WebApp;
        tg.expand(); tg.ready();

        let lang = (tg.initDataUnsafe?.user?.language_code === 'ru') ? 'ru' : 'en';

        const uiText = {
            ru: { tabStd: "🔢 Простой", tabEng: "📐 Инженерный", tabCnv: "🔄 Конвертер", deskBtn: "📱 На рабочий стол", shareBtn: "📤 Поделиться", errText: "Ошибка", shareTextTitle: "Смотри, что у меня вышло:\n" },
            en: { tabStd: "🔢 Standard", tabEng: "📐 Scientific", tabCnv: "🔄 Converter", deskBtn: "📱 Home Screen", shareBtn: "📤 Share", errText: "Error", shareTextTitle: "Look what I got:\n" }
        };

        const convData = {
            length: {
                name: { ru: "📏 Длина и расстояние", en: "📏 Length & Distance" },
                units: {
                    m: { name: { ru: "Метры (m)", en: "Meters (m)" }, rate: 1 },
                    km: { name: { ru: "Километры (km)", en: "Kilometers (km)" }, rate: 1000 },
                    cm: { name: { ru: "Сантиметры (cm)", en: "Centimeters (cm)" }, rate: 0.01 },
                    mm: { name: { ru: "Миллиметры (mm)", en: "Millimeters (mm)" }, rate: 0.001 },
                    mi: { name: { ru: "Мили (mi)", en: "Miles (mi)" }, rate: 1609.34 },
                    ft: { name: { ru: "Футы (ft)", en: "Feet (ft)" }, rate: 0.3048 },
                    in: { name: { ru: "Дюймы (in)", en: "Inches (in)" }, rate: 0.0254 }
                }
            },
            weight: {
                name: { ru: "⚖️ Вес и масса", en: "⚖️ Weight & Mass" },
                units: {
                    kg: { name: { ru: "Килограммы (kg)", en: "Kilograms (kg)" }, rate: 1 },
                    g: { name: { ru: "Граммы (g)", en: "Grams (g)" }, rate: 0.001 },
                    mg: { name: { ru: "Миллиграммы (mg)", en: "Milligrams (mg)" }, rate: 0.000001 },
                    lb: { name: { ru: "Фунты (lb)", en: "Pounds (lb)" }, rate: 0.453592 },
                    oz: { name: { ru: "Унции (oz)", en: "Ounces (oz)" }, rate: 0.0283495 }
                }
            },
            temp: {
                name: { ru: "🌡️ Температура", en: "🌡️ Temperature" },
                units: {
                    c: { name: { ru: "Цельсий (°C)", en: "Celsius (°C)" } },
                    f: { name: { ru: "Фаренгейт (°F)", en: "Fahrenheit (°F)" } },
                    k: { name: { ru: "Кельвин (K)", en: "Kelvin (K)" } }
                }
            }
        };

        document.getElementById('tab-std').innerText = uiText[lang].tabStd;
        document.getElementById('tab-eng').innerText = uiText[lang].tabEng;
        document.getElementById('tab-cnv').innerText = uiText[lang].tabCnv;
        document.getElementById('desktop-btn').innerText = uiText[lang].deskBtn;
        document.getElementById('share-btn').innerText = uiText[lang].shareBtn;

        let display = document.getElementById('display'), history = document.getElementById('history');
        let currentExpr = '0', lastEquation = "";

        function switchMode(mode, btnElement) {
            tg.HapticFeedback.impactOccurred('light');
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active')); btnElement.classList.add('active');
            document.getElementById('calc-view').style.display = 'none'; document.getElementById('converter-view').style.display = 'none'; document.getElementById('eng-row').style.display = 'none';
            if (mode === 'standard') { document.getElementById('calc-view').style.display = 'flex'; } 
            else if (mode === 'engineer') { document.getElementById('calc-view').style.display = 'flex'; document.getElementById('eng-row').style.display = 'grid'; } 
            else if (mode === 'converter') { document.getElementById('converter-view').style.display = 'block'; initConverter(); }
        }

        function formatExpression(expr) { return expr.replace(/\*/g, ' × ').replace(/\//g, ' ÷ ').replace(/Math.sin\(/g, 'sin(').replace(/Math.cos\(/g, 'cos(').replace(/Math.sqrt\(/g, '√('); }
        function calcAction(val) { tg.HapticFeedback.impactOccurred('light'); if (history.innerText !== "") history.innerText = ""; if (currentExpr === '0' && val !== '.') currentExpr = ''; currentExpr += val; display.innerText = formatExpression(currentExpr); }
        function clearDisplay() { tg.HapticFeedback.impactOccurred('medium'); currentExpr = '0'; history.innerText = ""; lastEquation = ""; display.innerText = "0"; }

        function calculate() {
            try {
                let formattedExpr = formatExpression(currentExpr);
                let result = eval(currentExpr);
                let finalResult = String(Math.round(result * 10000000) / 10000000);
                history.innerText = formattedExpr + " ="; display.innerText = finalResult;
                lastEquation = formattedExpr + " = " + finalResult; currentExpr = finalResult; tg.HapticFeedback.notificationOccurred('success');
            } catch(e) { display.innerText = uiText[lang].errText; currentExpr = '0'; tg.HapticFeedback.notificationOccurred('error'); }
        }

        let catSelect = document.getElementById('conv-category');
        let u1Select = document.getElementById('conv-unit1');
        let u2Select = document.getElementById('conv-unit2');
        let i1Input = document.getElementById('conv-input1');
        let i2Input = document.getElementById('conv-input2');
        let convInitialized = false;

        function initConverter() {
            if (convInitialized) return;
            for (let key in convData) {
                let opt = document.createElement('option');
                opt.value = key; opt.innerText = convData[key].name[lang];
                catSelect.appendChild(opt);
            }
            updateUnitOptions();
            convInitialized = true;
        }

        function updateUnitOptions() {
            let cat = catSelect.value;
            let units = convData[cat].units;
            u1Select.innerHTML = ''; u2Select.innerHTML = '';
            
            for (let key in units) {
                let opt1 = document.createElement('option');
                let opt2 = document.createElement('option');
                opt1.value = key; opt1.innerText = units[key].name[lang];
                opt2.value = key; opt2.innerText = units[key].name[lang];
                u1Select.appendChild(opt1); u2Select.appendChild(opt2);
            }
            if(u2Select.options.length > 1) u2Select.selectedIndex = 1;
            doConvert(1);
        }

        function doConvert(sourceObj) {
            let cat = catSelect.value;
            let val = parseFloat(sourceObj === 1 ? i1Input.value : i2Input.value);
            let from = sourceObj === 1 ? u1Select.value : u2Select.value;
            let to = sourceObj === 1 ? u2Select.value : u1Select.value;
            let targetInput = sourceObj === 1 ? i2Input : i1Input;

            if (isNaN(val)) { targetInput.value = ''; return; }

            let result = 0;
            if (cat === 'temp') {
                let c = val;
                if (from === 'f') c = (val - 32) * 5/9;
                if (from === 'k') c = val - 273.15;
                if (to === 'f') result = c * 9/5 + 32;
                else if (to === 'k') result = c + 273.15;
                else result = c;
            } else {
                let rateFrom = convData[cat].units[from].rate;
                let rateTo = convData[cat].units[to].rate;
                result = val * (rateFrom / rateTo);
            }
            
            targetInput.value = Math.round(result * 100000) / 100000;
        }

        function addToHomeScreen() { tg.HapticFeedback.impactOccurred('heavy'); if (tg.addToHomeScreen) tg.addToHomeScreen(); }
        
        function shareResult() {
            tg.HapticFeedback.impactOccurred('heavy');
            let botUsername = "calculator_number1_bot"; // ЮЗЕРНЕЙМ ВШИТ СЮДА
            let equationToShare = lastEquation !== "" ? lastEquation : display.innerText;
            let textToShare = uiText[lang].shareTextTitle + '`' + equationToShare + '`';
            let shareUrl = 'https://t.me/share/url?url=https://t.me/' + botUsername + '&text=' + encodeURIComponent(textToShare);
            tg.openTelegramLink(shareUrl);
        }
    </script>
</body>
</html>
# calculator-number1
