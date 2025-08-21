<script>
  import { createEventDispatcher } from 'svelte';
  import { onMount, onDestroy } from 'svelte';

  const dispatch = createEventDispatcher();

  let hours = 0;
  let minutes = 25;
  let seconds = 0;
  let isRunning = false;
  let isPaused = false;
  let timeLeft = 0;
  let interval;
  let audio;

  // 프리셋 타이머 설정
  const presets = [
    { name: '뽀모도로', hours: 0, minutes: 25, seconds: 0, color: '#ff6b6b' },
    { name: '짧은 휴식', hours: 0, minutes: 5, seconds: 0, color: '#4ecdc4' },
    { name: '긴 휴식', hours: 0, minutes: 15, seconds: 0, color: '#45b7d1' },
    { name: '운동 세트', hours: 0, minutes: 3, seconds: 0, color: '#96ceb4' },
    { name: '요리 타이머', hours: 0, minutes: 30, seconds: 0, color: '#feca57' },
    { name: '명상', hours: 0, minutes: 10, seconds: 0, color: '#a8e6cf' }
  ];

  onMount(() => {
    // 알림음 설정
    audio = new Audio('data:audio/wav;base64,UklGRnoGAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YQoGAACBhYqFbF1fdJivrJBhNjVgodDbq2EcBj+a2/LDciUFLIHO8tiJNwgZaLvt559NEAxQp+PwtmMcBjiR1/LMeSwFJHfH8N2QQAoUXrTp66hVFApGn+DyvmwhBSuBzvLZiTYIG2m98OScTgwOUarm7blmGgU7k9n1unEiBC13yO/eizEIHWq+8+OWT');
    audio.volume = 0.5;
    
    updateTimeLeft();
  });

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });

  function updateTimeLeft() {
    timeLeft = hours * 3600 + minutes * 60 + seconds;
  }

  function formatTime(totalSeconds) {
    const h = Math.floor(totalSeconds / 3600);
    const m = Math.floor((totalSeconds % 3600) / 60);
    const s = totalSeconds % 60;
    return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
  }

  function startTimer() {
    if (timeLeft <= 0) return;
    
    isRunning = true;
    isPaused = false;
    
    interval = setInterval(() => {
      timeLeft--;
      
      if (timeLeft <= 0) {
        stopTimer();
        playNotification();
        showNotification();
      }
    }, 1000);
  }

  function pauseTimer() {
    isPaused = true;
    clearInterval(interval);
  }

  function resumeTimer() {
    if (timeLeft <= 0) return;
    
    isPaused = false;
    startTimer();
  }

  function stopTimer() {
    isRunning = false;
    isPaused = false;
    clearInterval(interval);
    timeLeft = hours * 3600 + minutes * 60 + seconds;
  }

  function resetTimer() {
    stopTimer();
    timeLeft = hours * 3600 + minutes * 60 + seconds;
  }

  function setPreset(preset) {
    hours = preset.hours;
    minutes = preset.minutes;
    seconds = preset.seconds;
    resetTimer();
  }

  function playNotification() {
    if (audio) {
      audio.play().catch(e => console.log('알림음 재생 실패:', e));
    }
  }

  function showNotification() {
    if ('Notification' in window && Notification.permission === 'granted') {
      new Notification('타이머 완료!', {
        body: '설정한 시간이 완료되었습니다.',
        icon: '/favicon.ico'
      });
    }
  }

  function requestNotificationPermission() {
    if ('Notification' in window && Notification.permission === 'default') {
      Notification.requestPermission();
    }
  }

  function saveCurrentTimer() {
    const timerName = prompt('타이머 이름을 입력하세요:');
    if (timerName) {
      dispatch('saveTimer', {
        name: timerName,
        hours,
        minutes,
        seconds,
        color: `hsl(${Math.random() * 360}, 70%, 60%)`
      });
    }
  }

  $: progress = timeLeft > 0 ? ((hours * 3600 + minutes * 60 + seconds - timeLeft) / (hours * 3600 + minutes * 60 + seconds)) * 100 : 0;
  $: timeDisplay = formatTime(timeLeft);
</script>

<div class="timer-container">
  <div class="timer-display">
    <div class="time-circle">
      <svg class="progress-ring" width="300" height="300">
        <circle
          class="progress-ring-circle-bg"
          stroke="#e0e0e0"
          stroke-width="8"
          fill="transparent"
          r="140"
          cx="150"
          cy="150"
        />
        <circle
          class="progress-ring-circle"
          stroke="#667eea"
          stroke-width="8"
          fill="transparent"
          r="140"
          cx="150"
          cy="150"
          stroke-dasharray="{2 * Math.PI * 140}"
          stroke-dashoffset="{2 * Math.PI * 140 * (1 - progress / 100)}"
          transform="rotate(-90 150 150)"
        />
      </svg>
      <div class="time-text">
        <div class="time-display">{timeDisplay}</div>
        <div class="timer-status">
          {#if isRunning && !isPaused}
            실행 중...
          {:else if isPaused}
            일시정지
          {:else}
            준비됨
          {/if}
        </div>
      </div>
    </div>
  </div>

  <div class="timer-controls">
    <div class="time-inputs">
      <div class="input-group">
        <label>시간</label>
        <input 
          type="number" 
          bind:value={hours} 
          min="0" 
          max="23" 
          disabled={isRunning}
          on:change={updateTimeLeft}
        />
      </div>
      <div class="input-group">
        <label>분</label>
        <input 
          type="number" 
          bind:value={minutes} 
          min="0" 
          max="59" 
          disabled={isRunning}
          on:change={updateTimeLeft}
        />
      </div>
      <div class="input-group">
        <label>초</label>
        <input 
          type="number" 
          bind:value={seconds} 
          min="0" 
          max="59" 
          disabled={isRunning}
          on:change={updateTimeLeft}
        />
      </div>
    </div>

    <div class="control-buttons">
      {#if !isRunning}
        <button class="btn btn-primary" on:click={startTimer} disabled={timeLeft <= 0}>
          ▶️ 시작
        </button>
      {:else if isPaused}
        <button class="btn btn-primary" on:click={resumeTimer}>
          ▶️ 재개
        </button>
      {:else}
        <button class="btn btn-secondary" on:click={pauseTimer}>
          ⏸️ 일시정지
        </button>
      {/if}
      
      <button class="btn btn-danger" on:click={stopTimer} disabled={!isRunning && !isPaused}>
        ⏹️ 정지
      </button>
      
      <button class="btn btn-warning" on:click={resetTimer}>
        🔄 리셋
      </button>
      
      <button class="btn btn-success" on:click={saveCurrentTimer}>
        💾 저장
      </button>
    </div>
  </div>

  <div class="preset-timers">
    <h3>빠른 설정</h3>
    <div class="preset-grid">
      {#each presets as preset}
        <button 
          class="preset-btn" 
          style="background-color: {preset.color}"
          on:click={() => setPreset(preset)}
          disabled={isRunning}
        >
          <div class="preset-name">{preset.name}</div>
          <div class="preset-time">{formatTime(preset.hours * 3600 + preset.minutes * 60 + preset.seconds)}</div>
        </button>
      {/each}
    </div>
  </div>

  <div class="notification-settings">
    <button class="btn btn-info" on:click={requestNotificationPermission}>
      🔔 알림 권한 요청
    </button>
  </div>
</div>

<style>
  .timer-container {
    text-align: center;
  }

  .timer-display {
    margin-bottom: 2rem;
  }

  .time-circle {
    position: relative;
    display: inline-block;
  }

  .progress-ring {
    transform: rotate(-90deg);
  }

  .progress-ring-circle {
    transition: stroke-dashoffset 0.35s;
    transform-origin: 50% 50%;
  }

  .time-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
  }

  .time-display {
    font-size: 3rem;
    font-weight: 700;
    color: #333;
    font-family: 'Courier New', monospace;
    margin-bottom: 0.5rem;
  }

  .timer-status {
    font-size: 1rem;
    color: #666;
    font-weight: 500;
  }

  .timer-controls {
    margin-bottom: 2rem;
  }

  .time-inputs {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 1.5rem;
  }

  .input-group {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .input-group label {
    font-size: 0.9rem;
    color: #666;
    margin-bottom: 0.5rem;
    font-weight: 600;
  }

  .input-group input {
    width: 80px;
    padding: 0.8rem;
    border: 2px solid #e0e0e0;
    border-radius: 10px;
    font-size: 1.2rem;
    text-align: center;
    font-weight: 600;
    transition: all 0.3s ease;
  }

  .input-group input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }

  .input-group input:disabled {
    background-color: #f5f5f5;
    color: #999;
  }

  .control-buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .btn {
    padding: 0.8rem 1.5rem;
    border: none;
    border-radius: 50px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    min-width: 120px;
  }

  .btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.2);
  }

  .btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none;
  }

  .btn-primary {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
  }

  .btn-secondary {
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
    color: white;
  }

  .btn-danger {
    background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
    color: white;
  }

  .btn-warning {
    background: linear-gradient(135deg, #feca57 0%, #ff9ff3 100%);
    color: white;
  }

  .btn-success {
    background: linear-gradient(135deg, #48cae4 0%, #00b4d8 100%);
    color: white;
  }

  .btn-info {
    background: linear-gradient(135deg, #a8e6cf 0%, #88d8c0 100%);
    color: #333;
  }

  .preset-timers {
    margin-bottom: 2rem;
  }

  .preset-timers h3 {
    margin-bottom: 1rem;
    color: #333;
    font-size: 1.3rem;
  }

  .preset-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 1rem;
    max-width: 600px;
    margin: 0 auto;
  }

  .preset-btn {
    background: linear-gradient(135deg, var(--color) 0%, var(--color-dark) 100%);
    border: none;
    border-radius: 15px;
    padding: 1.5rem 1rem;
    cursor: pointer;
    transition: all 0.3s ease;
    color: white;
    font-weight: 600;
  }

  .preset-btn:hover:not(:disabled) {
    transform: translateY(-3px);
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  }

  .preset-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .preset-name {
    font-size: 1.1rem;
    margin-bottom: 0.5rem;
  }

  .preset-time {
    font-size: 0.9rem;
    opacity: 0.9;
  }

  .notification-settings {
    margin-top: 2rem;
  }

  @media (max-width: 768px) {
    .time-display {
      font-size: 2rem;
    }
    
    .time-inputs {
      flex-direction: column;
      align-items: center;
    }
    
    .control-buttons {
      flex-direction: column;
      align-items: center;
    }
    
    .btn {
      width: 100%;
      max-width: 200px;
    }
    
    .preset-grid {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>
