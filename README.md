<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Upper Decky</title>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; --primary: #38bdf8; --text: #f8fafc; --danger: #ef4444; }
        body { font-family: -apple-system, system-ui; background: var(--bg); color: var(--text); margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; box-sizing: border-box; }
        .container { width: 100%; max-width: 400px; text-align: center; }
        .hidden { display: none !important; }
        .seats-display { font-size: 4rem; font-weight: 800; color: var(--primary); margin: 10px 0; }
        .sub-text { opacity: 0.6; font-size: 0.9rem; text-transform: uppercase; letter-spacing: 1px; }
        .btn { display: block; width: 100%; padding: 20px; margin: 12px 0; border-radius: 16px; border: none; font-size: 1.2rem; font-weight: 700; cursor: pointer; -webkit-tap-highlight-color: transparent; }
        .btn-main { background: var(--primary); color: var(--bg); }
        .btn-land { background: var(--danger); color: white; }
        .btn-sub { background: var(--card); color: var(--text); }
        .btn-new-flight { background: transparent; color: #94a3b8; border: 1px solid #334155; margin-top: 30px; font-size: 0.9rem; }
        .history-list { text-align: left; margin-top: 20px; background: var(--card); border-radius: 12px; padding: 10px; max-height: 300px; overflow-y: auto; }
        .history-item { padding: 12px; border-bottom: 1px solid #334155; font-size: 0.85rem; display: flex; justify-content: space-between; }
        input { width: 100%; padding: 15px; border-radius: 12px; border: 2px solid var(--primary); background: var(--bg); color: white; margin: 15px 0; font-size: 1.5rem; text-align: center; box-sizing: border-box; }
    </style>
</head>
<body>

    <div class="container">
        <div id="view-setup" class="hidden">
            <h1>Set Up Trip</h1>
            <p>How many seats available?</p>
            <input type="number" id="input-seats" placeholder="20" inputmode="numeric">
            <button class="btn btn-main" onclick="startNewSet()">Initialize Set</button>
        </div>

        <div id="view-main" class="hidden">
            <div class="sub-text">Seats Remaining</div>
            <div id="display-seats" class="seats-display">0</div>
            
            <button class="btn btn-main" onclick="takeoff()">UPPER DECKY</button>
            <button class="btn btn-sub" onclick="showHistory()">MANAGE TRIP</button>
            <button class="btn btn-new-flight" onclick="resetAll()">NEW FLIGHT (RESET SET)</button>
        </div>

        <div id="view-flying" class="hidden">
            <div class="sub-text">In Flight</div>
            <div id="timer" class="seats-display">00:00</div>
            <button class="btn btn-land" onclick="showDisposal()">LAND</button>
        </div>

        <div id="view-disposal" class="hidden">
            <h2>Disposal Location</h2>
            <button class="btn btn-sub" onclick="land('Back in Tub')">Back in the Tub</button>
            <button class="btn btn-sub" onclick="land('Bin')">Bin</button>
            <button class="btn btn-sub" onclick="land('Pocket')">Pocket</button>
            <button class="btn btn-sub" onclick="land('Other')">Other</button>
        </div>

        <div id="view-history" class="hidden">
            <h2>Set History</h2>
            <div id="history-content" class="history-list"></div>
            <button class="btn btn-sub" onclick="showMain()">Back to Cockpit</button>
        </div>
    </div>

    <script>
        let state = JSON.parse(localStorage.getItem('deckyStore')) || {
            seats: 0, active: false, flying: false, startTime: null, history: []
        };

        let timerTick;

        function save() {
            localStorage.setItem('deckyStore', JSON.stringify(state));
            render();
        }

        function render() {
            document.querySelectorAll('.container > div').forEach(div => div.classList.add('hidden'));
            
            if (!state.active) {
                document.getElementById('view-setup').classList.remove('hidden');
            } else if (state.flying) {
                document.getElementById('view-flying').classList.remove('hidden');
                startTimer();
            } else {
                document.getElementById('view-main').classList.remove('hidden');
                document.getElementById('display-seats').innerText = state.seats;
                clearInterval(timerTick);
            }
        }

        function startNewSet() {
            const val = document.getElementById('input-seats').value;
            if (val > 0) {
                state = { seats: parseInt(val), active: true, flying: false, startTime: null, history: [] };
                save();
            }
        }

        function takeoff() {
            state.flying = true;
            state.startTime = Date.now();
            save();
        }

        function showDisposal() {
            document.getElementById('view-flying').classList.add('hidden');
            document.getElementById('view-disposal').classList.remove('hidden');
        }

        function land(method) {
            const durationMs = Date.now() - state.startTime;
            const m = Math.floor(durationMs / 60000);
            const s = Math.floor((durationMs % 60000) / 1000);

            state.history.unshift({
                time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
                duration: `${m}m ${s}s`,
                disposal: method
            });

            state.seats = Math.max(0, state.seats - 1);
            state.flying = false;
            state.startTime = null;
            if (state.seats === 0) state.active = false;
            save();
        }

        function startTimer() {
            clearInterval(timerTick);
            timerTick = setInterval(() => {
                const diff = Date.now() - state.startTime;
                const m = Math.floor(diff / 60000).toString().padStart(2, '0');
                const s = Math.floor((diff % 60000) / 1000).toString().padStart(2, '0');
                document.getElementById('timer').innerText = `${m}:${s}`;
            }, 1000);
        }

        function showHistory() {
            document.getElementById('view-main').classList.add('hidden');
            document.getElementById('view-history').classList.remove('hidden');
            const list = document.getElementById('history-content');
            list.innerHTML = state.history.map(h => `
                <div class="history-item">
                    <span><strong>${h.duration}</strong> (${h.disposal})</span>
                    <span style="opacity:0.5">${h.time}</span>
                </div>
            `).join('') || '<p style="text-align:center">No records found.</p>';
        }

        function showMain() { render(); }

        function resetAll() {
            if(confirm("Start a new set? This clears all current progress.")) {
                state.active = false;
                save();
            }
        }

        render();
    </script>
</body>
</html>
