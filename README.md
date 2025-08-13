<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>项目破产概率计算</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #f0f2f5;
            color: #333;
            line-height: 1.6;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: 40px auto;
            padding: 30px;
            background-color: #fff;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            border-radius: 12px;
        }
        h1, h2 {
            text-align: center;
            color: #2c3e50;
        }
        .input-group {
            margin-bottom: 25px;
        }
        .input-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
            color: #555;
        }
        .input-group input {
            width: 100%;
            padding: 12px;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 16px;
        }
        .input-group input:focus {
            border-color: #3498db;
            outline: none;
            box-shadow: 0 0 5px rgba(52, 152, 219, 0.5);
        }
        .result-box {
            background-color: #ecf0f1;
            padding: 20px;
            border-radius: 8px;
            margin-top: 25px;
        }
        .result-box p {
            margin: 0;
            padding: 5px 0;
            font-size: 18px;
            color: #2c3e50;
        }
        .result-box strong {
            color: #e74c3c;
        }
        .result-box .f-value {
            font-size: 22px;
            font-weight: bold;
        }
        .error-message {
            color: #e74c3c;
            font-weight: bold;
            text-align: center;
            margin-top: 15px;
        }
        .info-section {
            margin-top: 40px;
            padding-top: 20px;
            border-top: 1px dashed #bdc3c7;
        }
        .info-section h3 {
            color: #3498db;
        }
        .info-section p, .info-section li {
            font-size: 14px;
            color: #666;
        }
        .unit {
            font-size: 14px;
            color: #7f8c8d;
            margin-left: 5px;
        }
        .action-buttons {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 20px;
        }
        .action-buttons button {
            padding: 12px 25px;
            font-size: 16px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        #calculateBtn {
            background-color: #3498db;
            color: #fff;
        }
        #calculateBtn:hover {
            background-color: #2980b9;
        }
        #clearRecordsBtn {
            background-color: #e74c3c;
            color: #fff;
        }
        #clearRecordsBtn:hover {
            background-color: #c0392b;
        }
        #records-list {
            margin-top: 30px;
            padding: 0;
            list-style: none;
            border-top: 1px dashed #bdc3c7;
        }
        #records-list li {
            background-color: #f9f9f9;
            border: 1px solid #eee;
            margin-top: 10px;
            padding: 15px;
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        #records-list li h4 {
            margin: 0;
            color: #2c3e50;
        }
        #records-list li p {
            margin: 0;
            font-size: 14px;
            color: #666;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>项目破产概率计算</h1>
    <p style="text-align: center;">根据收益和成本，计算破产概率和建议资金值</p>

    <div class="input-group">
        <label for="projectNameInput">项目名称 (可选)</label>
        <input type="text" id="projectNameInput" placeholder="例如：碎贤者，转贤者">
    </div>

    <div class="input-group">
        <label for="pInput">每次收入</label>
        <input type="text" id="pInput" placeholder="例如：1000, 10k, 1m">
        <span class="unit"></span>
    </div>

    <div class="input-group">
        <label for="c0Input">每次成本</label>
        <input type="text" id="c0Input" placeholder="例如：800, 0.8k, 0.8m">
        <span class="unit"></span>
    </div>

    <div class="input-group">
        <label for="wInput">资金</label>
        <input type="text" id="wInput" placeholder="例如：5000, 5k, 5m">
        <span class="unit"></span>
    </div>

    <div class="result-box">
        <div id="result">
            <p>破产概率 : <strong id="fValue" class="f-value"></strong></p>
            <p>推荐资金 : <strong id="FValue"></strong> <span class="unit"></span></p>
        </div>
        <div id="error" class="error-message" style="display:none;"></div>
    </div>

    <div class="action-buttons">
        <button id="calculateBtn">确认计算</button>
        <button id="clearRecordsBtn">清除所有记录</button>
    </div>

    <div class="info-section">
        <h2>项目记录</h2>
        <ul id="records-list">
            </ul>
    </div>
</div>

<script>
    document.addEventListener('DOMContentLoaded', () => {
        const projectNameInput = document.getElementById('projectNameInput');
        const pInput = document.getElementById('pInput');
        const c0Input = document.getElementById('c0Input');
        const wInput = document.getElementById('wInput');
        const calculateBtn = document.getElementById('calculateBtn');
        const clearRecordsBtn = document.getElementById('clearRecordsBtn');

        const fValueElement = document.getElementById('fValue');
        const FValueElement = document.getElementById('FValue');
        const errorElement = document.getElementById('error');
        const resultElement = document.getElementById('result');
        const recordsList = document.getElementById('records-list');

        const LOCAL_STORAGE_KEY_INPUTS = 'projectRiskInputs';
        const LOCAL_STORAGE_KEY_RECORDS = 'projectRiskRecords';

        // Helper function to parse input strings with k, m, b
        const parseInput = (value) => {
            const lowerCaseValue = value.toLowerCase().trim();
            if (!lowerCaseValue) return NaN;

            const num = parseFloat(lowerCaseValue);
            if (isNaN(num)) {
                if (lowerCaseValue.endsWith('b')) {
                    return parseFloat(lowerCaseValue.slice(0, -1)) * 1e9;
                }
                if (lowerCaseValue.endsWith('m')) {
                    return parseFloat(lowerCaseValue.slice(0, -1)) * 1e6;
                }
                if (lowerCaseValue.endsWith('k')) {
                    return parseFloat(lowerCaseValue.slice(0, -1)) * 1e3;
                }
                return NaN;
            } else {
                if (lowerCaseValue.endsWith('b')) {
                    return num * 1e9;
                }
                if (lowerCaseValue.endsWith('m')) {
                    return num * 1e6;
                }
                if (lowerCaseValue.endsWith('k')) {
                    return num * 1e3;
                }
                return num;
            }
        };

        // Format numbers with k, m, b suffixes
        const formatNumber = (num) => {
            if (Math.abs(num) >= 1e9) {
                return (num / 1e9).toFixed(2) + ' B';
            }
            if (Math.abs(num) >= 1e6) {
                return (num / 1e6).toFixed(2) + ' M';
            }
            if (Math.abs(num) >= 1e3) {
                return (num / 1e3).toFixed(2) + ' K';
            }
            return num.toLocaleString('zh-CN', { maximumFractionDigits: 2 });
        };

        const formatProbability = (prob) => {
            if (isNaN(prob)) return "N/A";
            return (prob * 100).toFixed(4) + '%';
        };

        // Main calculation logic
        const calculateAndSave = () => {
            const p = parseInput(pInput.value);
            const c0 = parseInput(c0Input.value);
            const w = parseInput(wInput.value);
            const projectName = projectNameInput.value || '未命名项目';

            // Save the current inputs to local storage
            const inputs = {
                projectName: projectNameInput.value,
                p: pInput.value,
                c0: c0Input.value,
                w: wInput.value
            };
            localStorage.setItem(LOCAL_STORAGE_KEY_INPUTS, JSON.stringify(inputs));

            if (isNaN(p) || isNaN(c0) || isNaN(w) || p <= 0 || c0 <= 0) {
                resultElement.style.display = 'none';
                errorElement.textContent = '请输入有效的正数。';
                errorElement.style.display = 'block';
                return;
            }

            if (p <= c0) {
                resultElement.style.display = 'none';
                errorElement.textContent = '模型要求收入大于成本，否则破产概率为100%。';
                errorElement.style.display = 'block';
                return;
            }

            errorElement.style.display = 'none';
            resultElement.style.display = 'block';

            let r_approx = 1 / c0;
            if (p / c0 > 1) {
                r_approx = 1 / p;
            }

            for (let i = 0; i < 100; i++) {
                const f = r_approx - (1 / c0) * Math.exp((-p / c0) + r_approx * p);
                const f_prime = 1 - (p / c0) * Math.exp((-p / c0) + r_approx * p);
                if (Math.abs(f_prime) < 1e-9) {
                    break;
                }
                const next_r_approx = r_approx - f / f_prime;
                if (Math.abs(next_r_approx - r_approx) < 1e-9) {
                    r_approx = next_r_approx;
                    break;
                }
                r_approx = next_r_approx;
            }
            const r = r_approx;

            const F = c0 / (1 - r * c0);
            const f_w = Math.exp(-w / F);

            FValueElement.textContent = formatNumber(F);
            fValueElement.textContent = formatProbability(f_w);

            // Save the record
            const newRecord = {
                id: Date.now(),
                projectName: projectName,
                p: formatNumber(p),
                c0: formatNumber(c0),
                w: formatNumber(w),
                bankruptcyProbability: formatProbability(f_w),
                recommendedCapital: formatNumber(F),
                timestamp: new Date().toLocaleString('zh-CN')
            };

            const records = JSON.parse(localStorage.getItem(LOCAL_STORAGE_KEY_RECORDS) || '[]');
            records.unshift(newRecord); // Add to the beginning of the list
            localStorage.setItem(LOCAL_STORAGE_KEY_RECORDS, JSON.stringify(records));

            displayRecords();
        };

        // Function to display saved records
        const displayRecords = () => {
            recordsList.innerHTML = '';
            const records = JSON.parse(localStorage.getItem(LOCAL_STORAGE_KEY_RECORDS) || '[]');

            if (records.length === 0) {
                recordsList.innerHTML = '<p style="text-align: center; color: #999;">暂无记录</p>';
                return;
            }

            records.forEach(record => {
                const li = document.createElement('li');
                li.innerHTML = `
                    <h4>${record.projectName}</h4>
                    <p><b>收入:</b> ${record.p} | <b>成本:</b> ${record.c0} | <b>资金:</b> ${record.w}</p>
                    <p><b>破产概率:</b> <strong style="color: #e74c3c;">${record.bankruptcyProbability}</strong> | <b>推荐资金:</b> ${record.recommendedCapital}</p>
                    <p style="text-align: right; font-size: 12px; color: #aaa;">${record.timestamp}</p>
                `;
                recordsList.appendChild(li);
            });
        };
        
        // Function to clear all records
        const clearAllRecords = () => {
            if (confirm('确定要清除所有项目记录吗？此操作无法撤销。')) {
                localStorage.removeItem(LOCAL_STORAGE_KEY_RECORDS);
                displayRecords();
            }
        };

        // Load saved inputs from local storage on page load
        const loadInputs = () => {
            const savedInputs = JSON.parse(localStorage.getItem(LOCAL_STORAGE_KEY_INPUTS));
            if (savedInputs) {
                projectNameInput.value = savedInputs.projectName || '';
                pInput.value = savedInputs.p || '';
                c0Input.value = savedInputs.c0 || '';
                wInput.value = savedInputs.w || '';
            }
        };

        // Event Listeners
        calculateBtn.addEventListener('click', calculateAndSave);
        clearRecordsBtn.addEventListener('click', clearAllRecords);

        projectNameInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                e.preventDefault();
                pInput.focus();
            }
        });

        pInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                e.preventDefault();
                c0Input.focus();
            }
        });

        c0Input.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                e.preventDefault();
                wInput.focus();
            }
        });

        wInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                e.preventDefault();
                calculateBtn.click();
            }
        });

        // Initial setup
        loadInputs();
        displayRecords();
        // Since we're now using a button for calculation, we don't need a live update on every input change,
        // but we can still calculate once at the start with loaded values.
        calculateAndSave();
    });
</script>

</body>
</html>
