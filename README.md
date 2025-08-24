<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Precision Stopwatch</title>
    <style>
        :root {
            --primary: #3a86ff;
            --secondary: #8338ec;
            --accent: #ff006e;
            --dark: #212529;
            --light: #f8f9fa;
            --success: #38b000;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background: linear-gradient(135deg, var(--dark), #343a40);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            color: var(--light);
        }

        .container {
            width: 90%;
            max-width: 500px;
            background: rgba(0, 0, 0, 0.3);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 2rem;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .display {
            font-size: 4rem;
            font-weight: 300;
            text-align: center;
            margin: 1.5rem 0;
            font-feature-settings: "tnum";
            font-variant-numeric: tabular-nums;
        }

        .controls {
            display: flex;
            justify-content: center;
            gap: 1rem;
            margin-bottom: 2rem;
        }

        button {
            border: none;
            border-radius: 50px;
            padding: 0.8rem 1.5rem;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s ease;
            text-transform: uppercase;
            letter-spacing: 1px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 0.5rem;
        }

        .start {
            background-color: var(--primary);
            color: white;
        }

        .pause {
            background-color: var(--accent);
            color: white;
        }

        .reset {
            background-color: var(--dark);
            color: var(--light);
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        .lap {
            background-color: var(--secondary);
            color: white;
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }

        button:active {
            transform: translateY(0);
        }

        .laps {
            margin-top: 2rem;
            max-height: 300px;
            overflow-y: auto;
            border-top: 1px solid rgba(255, 255, 255, 0.1);
            padding-top: 1rem;
        }

        .laps-header {
            display: flex;
            justify-content: space-between;
            padding: 0.5rem 1rem;
            font-weight: 600;
            color: var(--primary);
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            margin-bottom: 0.5rem;
        }

        .lap-item {
            display: flex;
            justify-content: space-between;
            padding: 0.8rem 1rem;
            background: rgba(0, 0, 0, 0.2);
            border-radius: 8px;
            margin-bottom: 0.5rem;
            font-size: 0.9rem;
        }

        .lap-item:first-child {
            background: rgba(58, 134, 255, 0.2);
            border-left: 3px solid var(--primary);
        }

        .fastest {
            color: var(--success);
        }

        .slowest {
            color: var(--accent);
        }

        @media (max-width: 480px) {
            .display {
                font-size: 3rem;
            }
            
            .controls {
                flex-wrap: wrap;
            }
            
            button {
                flex: 1 1 40%;
                padding: 0.8rem 0;
            }
        }

        /* Scrollbar styling */
        ::-webkit-scrollbar {
            width: 8px;
        }

        ::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.1);
            border-radius: 10px;
        }

        ::-webkit-scrollbar-thumb {
            background: rgba(255, 255, 255, 0.2);
            border-radius: 10px;
        }

        ::-webkit-scrollbar-thumb:hover {
            background: rgba(255, 255, 255, 0.3);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 style="text-align: center; margin-bottom: 1rem;">Precision Stopwatch</h1>
        <div class="display" id="display">00:00:00.000</div>
        
        <div class="controls">
            <button class="start" id="startBtn">
                Start
            </button>
            <button class="pause" id="pauseBtn" disabled>
                Pause
            </button>
            <button class="lap" id="lapBtn" disabled>
                Lap
            </button>
            <button class="reset" id="resetBtn" disabled>
                Reset
            </button>
        </div>
        
        <div class="laps" id="lapsContainer">
            <div class="laps-header">
                <span>Lap</span>
                <span>Time</span>
            </div>
            <div id="laps"></div>
        </div>
    </div>

    <script>
        let startTime;
        let elapsedTime = 0;
        let timerInterval;
        let laps = [];
        
        const display = document.getElementById('display');
        const startBtn = document.getElementById('startBtn');
        const pauseBtn = document.getElementById('pauseBtn');
        const lapBtn = document.getElementById('lapBtn');
        const resetBtn = document.getElementById('resetBtn');
        const lapsContainer = document.getElementById('laps');
        
        function formatTime(time) {
            const date = new Date(time);
            const hours = date.getUTCHours().toString().padStart(2, '0');
            const minutes = date.getUTCMinutes().toString().padStart(2, '0');
            const seconds = date.getUTCSeconds().toString().padStart(2, '0');
            const milliseconds = Math.floor(date.getUTCMilliseconds() / 10).toString().padStart(2, '0');
            
            return `${hours}:${minutes}:${seconds}.${milliseconds}`;
        }
        
        function startTimer() {
            startTime = Date.now() - elapsedTime;
            timerInterval = setInterval(() => {
                elapsedTime = Date.now() - startTime;
                display.textContent = formatTime(elapsedTime);
            }, 10);
            
            startBtn.disabled = true;
            pauseBtn.disabled = false;
            lapBtn.disabled = false;
            resetBtn.disabled = false;
        }
        
        function pauseTimer() {
            clearInterval(timerInterval);
            startBtn.disabled = false;
            pauseBtn.disabled = true;
        }
        
        function resetTimer() {
            clearInterval(timerInterval);
            elapsedTime = 0;
            display.textContent = '00:00:00.000';
            laps = [];
            lapsContainer.innerHTML = '';
            
            startBtn.disabled = false;
            pauseBtn.disabled = true;
            lapBtn.disabled = true;
            resetBtn.disabled = true;
        }
        
        function recordLap() {
            const lapTime = elapsedTime;
            laps.push(lapTime);
            
            // Clear previous fastest/slowest classes
            const lapItems = document.querySelectorAll('.lap-item');
            lapItems.forEach(item => {
                item.classList.remove('fastest', 'slowest');
            });
            
            // Create new laps list
            lapsContainer.innerHTML = '';
            
            // Find fastest and slowest laps
            const lapTimes = laps.map((lap, index) => {
                return {
                    index,
                    time: index === 0 ? lap : lap - laps[index - 1]
                };
            });
            
            const allLapTimes = lapTimes.map(l => l.time).filter(t => t > 0);
            const fastestTime = Math.min(...allLapTimes);
            const slowestTime = Math.max(...allLapTimes);
            
            // Rebuild laps list with formatting
            lapTimes.forEach((lap, index) => {
                const lapElement = document.createElement('div');
                lapElement.className = 'lap-item';
                
                // For the first lap, display total time
                const lapDuration = index === 0 ? lap.time : lap.time;
                const formattedTime = formatTime(lapDuration);
                
                // Add classes for fastest/slowest laps
                if (lap.time === fastestTime) {
                    lapElement.classList.add('fastest');
                } else if (lap.time === slowestTime) {
                    lapElement.classList.add('slowest');
                }
                
                lapElement.innerHTML = `
                    <span>Lap ${index + 1}</span>
                    <span>${formattedTime}</span>
                `;
                
                lapsContainer.appendChild(lapElement);
            });
            
            // Scroll to latest lap
            const lapsDiv = document.getElementById('lapsContainer');
            lapsDiv.scrollTop = lapsDiv.scrollHeight;
        }
        
        startBtn.addEventListener('click', startTimer);
        pauseBtn.addEventListener('click', pauseTimer);
        resetBtn.addEventListener('click', resetTimer);
        lapBtn.addEventListener('click', recordLap);
    </script>
</body>
</html>
