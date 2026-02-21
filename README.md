
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DonaTracker Pro - Meta $20k</title>
    <style>
        :root { --pink: #ff85a2; --green: #2ecc71; --red: #e74c3c; --blue: #3498db; --dark: #2c3e50; }
        body { font-family: 'Segoe UI', sans-serif; background: #f4f7f6; margin: 0; padding: 15px; color: var(--dark); }
        .card { background: white; border-radius: 15px; padding: 20px; margin-bottom: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
        h2 { margin-top: 0; font-size: 1.1rem; border-bottom: 2px solid var(--pink); display: inline-block; padding-bottom: 5px; }
        .input-group { margin: 10px 0; flex: 1; }
        label { display: block; font-size: 0.75rem; color: #666; margin-bottom: 5px; font-weight: bold; }
        input { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; font-size: 1rem; }
        button { width: 100%; padding: 12px; border: none; border-radius: 8px; color: white; font-weight: bold; cursor: pointer; margin-top: 10px; transition: 0.3s; }
        .btn-main { background: var(--pink); }
        .btn-sec { background: var(--blue); }
        .btn-ghost { background: #95a5a6; font-size: 0.8rem; }
        .stats { display: flex; justify-content: space-between; font-weight: bold; margin-top: 10px; }
        .progress-container { background: #eee; height: 15px; border-radius: 10px; margin: 10px 0; }
        #progress-bar { background: var(--green); height: 100%; width: 0%; border-radius: 10px; transition: 0.8s; }
        #history-list { margin-top: 15px; max-height: 250px; overflow-y: auto; font-size: 0.85rem; border-top: 1px solid #eee; }
        .history-item { padding: 8px 0; border-bottom: 1px solid #f9f9f9; display: flex; justify-content: space-between; }
        .type-sale { color: var(--green); font-weight: bold; }
        .type-expense { color: var(--red); font-weight: bold; }
        .row { display: flex; gap: 8px; flex-wrap: wrap; }
    </style>
</head>
<body>

    <div class="card">
        <h2>Mi Meta: $20,000</h2>
        <div class="progress-container"><div id="progress-bar"></div></div>
        <div class="stats">
            <span>Ahorrado: $<span id="total-saved">0</span></span>
            <span style="color: var(--pink)">Faltan: $<span id="missing">20000</span></span>
        </div>
    </div>

    <div class="card">
        <h2>🍩 Venta del Día (1 Caja)</h2>
        <div class="row">
            <div class="input-group">
                <label>Uds ($3)</label>
                <input type="number" id="daily-units" placeholder="0">
            </div>
            <div class="input-group">
                <label>2x$5</label>
                <input type="number" id="daily-combos2" placeholder="0">
            </div>
            <div class="input-group">
                <label>4x$10</label>
                <input type="number" id="daily-combos4" placeholder="0">
            </div>
        </div>
        <button class="btn-main" onclick="saveTransaction('venta')">Registrar Venta de Caja</button>
    </div>

    <div class="card">
        <h2>💸 Gastos / Deudas</h2>
        <input type="text" id="exp-name" placeholder="¿En qué gastaste? (Ej: Pasajes)" style="margin-bottom: 10px;">
        <input type="number" id="exp-amount" placeholder="Monto $">
        <button class="btn-sec" onclick="saveTransaction('gasto')">Registrar Gasto</button>
    </div>

    <div class="card">
        <div style="display: flex; justify-content: space-between; align-items: center;">
            <h2>📜 Historial Mensual</h2>
            <button onclick="clearHistory()" class="btn-ghost" style="width: auto; padding: 5px 10px;">Borrar Todo</button>
        </div>
        <div id="history-list"></div>
    </div>

    <script>
        let balance = parseFloat(localStorage.getItem('balance')) || 0;
        let history = JSON.parse(localStorage.getItem('history')) || [];
        const meta = 20000;

        function updateUI() {
            document.getElementById('total-saved').innerText = balance.toLocaleString();
            document.getElementById('missing').innerText = (meta - balance).toLocaleString();
            let percent = (balance / meta) * 100;
            document.getElementById('progress-bar').style.width = Math.min(percent, 100) + "%";
            
            const list = document.getElementById('history-list');
            list.innerHTML = history.length === 0 ? '<p style="color:#999">No hay registros aún.</p>' : '';
            
            history.slice().reverse().forEach(item => {
                const div = document.createElement('div');
                div.className = 'history-item';
                const colorClass = item.type === 'venta' ? 'type-sale' : 'type-expense';
                div.innerHTML = `
                    <span><b>${item.date}</b> - ${item.detail}</span>
                    <span class="${colorClass}">${item.type === 'venta' ? '+' : '-'}$${item.amount}</span>
                `;
                list.appendChild(div);
            });

            localStorage.setItem('balance', balance);
            localStorage.setItem('history', JSON.stringify(history));
        }

        function saveTransaction(type) {
            let amount = 0;
            let detail = "";
            const date = new Date().toLocaleDateString('es-ES', {day:'2-digit', month:'short'});

            if (type === 'venta') {
                const units = (document.getElementById('daily-units').value || 0) * 3;
                const c2 = (document.getElementById('daily-combos2').value || 0) * 5;
                const c4 = (document.getElementById('daily-combos4').value || 0) * 10;
                
                // Calculamos ganancia restando los $20 que te costó la caja
                amount = (units + c2 + c4) - 20;
                detail = "Venta Caja (Ganancia)";
                
                if((units + c2 + c4) == 0) return alert("Ingresa alguna venta");
            } else {
                amount = parseFloat(document.getElementById('exp-amount').value);
                detail = document.getElementById('exp-name').value || "Gasto";
                if(!amount) return alert("Ingresa un monto");
            }

            if (type === 'venta') balance += amount; else balance -= amount;
            
            history.push({ date, type, amount, detail });
            updateUI();
            
            // Limpiar campos
            document.querySelectorAll('input').forEach(i => i.value = '');
        }

        function clearHistory() {
            if(confirm("¿Deseas borrar todo el historial y reiniciar el balance?")) {
                balance = 0;
                history = [];
                updateUI();
            }
        }

        updateUI();
    </script>
</body>
</html>
