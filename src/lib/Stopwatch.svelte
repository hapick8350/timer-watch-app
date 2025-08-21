<script>
  import { onMount, onDestroy } from 'svelte';

  let elapsedTime = 0;
  let isRunning = false;
  let interval;
  let lapTimes = [];
  let lapCounter = 1;

  onMount(() => {
    // 컴포넌트 마운트 시 초기화
  });

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });

  function formatTime(totalMilliseconds) {
    const totalSeconds = Math.floor(totalMilliseconds / 1000);
    const minutes = Math.floor(totalSeconds / 60);
    const seconds = totalSeconds % 60;
    const milliseconds = Math.floor((totalMilliseconds % 1000) / 10);
    
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}.${milliseconds.toString().padStart(2, '0')}`;
  }

  function startStopwatch() {
    if (!isRunning) {
      isRunning = true;
      const startTime = Date.now() - elapsedTime;
      
      interval = setInterval(() => {
        elapsedTime = Date.now() - startTime;
      }, 10);
    }
  }

  function stopStopwatch() {
    isRunning = false;
    clearInterval(interval);
  }

  function resetStopwatch() {
    stopStopwatch();
    elapsedTime = 0;
    lapTimes = [];
    lapCounter = 1;
  }

  function recordLap() {
    if (isRunning) {
      const lapTime = elapsedTime;
      const previousLapTime = lapTimes.length > 0 ? lapTimes[lapTimes.length - 1].totalTime : 0;
      const lapDuration = lapTime - previousLapTime;
      
      lapTimes = [...lapTimes, {
        id: lapCounter,
        totalTime: lapTime,
        lapDuration: lapDuration,
        formattedTotal: formatTime(lapTime),
        formattedLap: formatTime(lapDuration)
      }];
      
      lapCounter++;
    }
  }

  function clearLaps() {
    lapTimes = [];
    lapCounter = 1;
  }

  function getFastestLap() {
    if (lapTimes.length === 0) return null;
    return lapTimes.reduce((fastest, current) => 
      current.lapDuration < fastest.lapDuration ? current : fastest
    );
  }

  function getSlowestLap() {
    if (lapTimes.length === 0) return null;
    return lapTimes.reduce((slowest, current) => 
      current.lapDuration > slowest.lapDuration ? current : slowest
    );
  }

  $: displayTime = formatTime(elapsedTime);
  $: fastestLap = getFastestLap();
  $: slowestLap = getSlowestLap();
</script>

<div class="stopwatch-container">
  <div class="stopwatch-display">
    <div class="time-display">
      <div class="main-time">{displayTime}</div>
      <div class="status-text">
        {#if isRunning}
          실행 중...
        {:else if elapsedTime > 0}
          정지됨
        {:else}
          준비됨
        {/if}
      </div>
    </div>
  </div>

  <div class="stopwatch-controls">
    <div class="control-buttons">
      {#if !isRunning}
        <button class="btn btn-primary" on:click={startStopwatch}>
          ▶️ 시작
        </button>
      {:else}
        <button class="btn btn-secondary" on:click={stopStopwatch}>
          ⏸️ 정지
        </button>
      {/if}
      
      <button class="btn btn-success" on:click={recordLap} disabled={!isRunning}>
        ⏱️ 랩
      </button>
      
      <button class="btn btn-warning" on:click={resetStopwatch} disabled={elapsedTime === 0 && lapTimes.length === 0}>
        🔄 리셋
      </button>
    </div>
  </div>

  {#if lapTimes.length > 0}
    <div class="lap-times">
      <div class="lap-header">
        <h3>랩 타임 기록</h3>
        <button class="btn btn-danger small" on:click={clearLaps}>
          🗑️ 기록 삭제
        </button>
      </div>
      
      <div class="lap-stats">
        {#if fastestLap}
          <div class="stat-item fastest">
            <span class="stat-label">최고 기록</span>
            <span class="stat-value">{fastestLap.formattedLap}</span>
          </div>
        {/if}
        {#if slowestLap}
          <div class="stat-item slowest">
            <span class="stat-label">최저 기록</span>
            <span class="stat-value">{slowestLap.formattedLap}</span>
          </div>
        {/if}
      </div>

      <div class="lap-list">
        <div class="lap-item header">
          <span class="lap-number">랩</span>
          <span class="lap-time">랩 타임</span>
          <span class="total-time">총 시간</span>
        </div>
        
        {#each lapTimes.slice().reverse() as lap}
          <div class="lap-item {lap === fastestLap ? 'fastest' : ''} {lap === slowestLap ? 'slowest' : ''}">
            <span class="lap-number">#{lap.id}</span>
            <span class="lap-time">{lap.formattedLap}</span>
            <span class="total-time">{lap.formattedTotal}</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}

  <div class="instructions">
    <h4>💡 사용법</h4>
    <ul>
      <li><strong>시작</strong>: 스톱워치를 시작합니다</li>
      <li><strong>정지</strong>: 스톱워치를 일시정지합니다</li>
      <li><strong>랩</strong>: 현재 시간을 랩 타임으로 기록합니다</li>
      <li><strong>리셋</strong>: 스톱워치와 모든 기록을 초기화합니다</li>
    </ul>
  </div>
</div>

<style>
  .stopwatch-container {
    text-align: center;
  }

  .stopwatch-display {
    margin-bottom: 2rem;
  }

  .time-display {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 20px;
    padding: 3rem 2rem;
    color: white;
    box-shadow: 0 15px 35px rgba(102, 126, 234, 0.3);
  }

  .main-time {
    font-size: 4rem;
    font-weight: 700;
    font-family: 'Courier New', monospace;
    margin-bottom: 0.5rem;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
  }

  .status-text {
    font-size: 1.2rem;
    opacity: 0.9;
    font-weight: 500;
  }

  .stopwatch-controls {
    margin-bottom: 2rem;
  }

  .control-buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .btn {
    padding: 1rem 2rem;
    border: none;
    border-radius: 50px;
    font-size: 1.1rem;
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

  .btn.small {
    padding: 0.5rem 1rem;
    font-size: 0.9rem;
    min-width: auto;
  }

  .btn-primary {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
  }

  .btn-secondary {
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
    color: white;
  }

  .btn-success {
    background: linear-gradient(135deg, #48cae4 0%, #00b4d8 100%);
    color: white;
  }

  .btn-warning {
    background: linear-gradient(135deg, #feca57 0%, #ff9ff3 100%);
    color: white;
  }

  .btn-danger {
    background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
    color: white;
  }

  .lap-times {
    margin-bottom: 2rem;
  }

  .lap-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
  }

  .lap-header h3 {
    color: #333;
    font-size: 1.3rem;
    margin: 0;
  }

  .lap-stats {
    display: flex;
    justify-content: center;
    gap: 2rem;
    margin-bottom: 1.5rem;
  }

  .stat-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 1rem;
    border-radius: 15px;
    min-width: 120px;
  }

  .stat-item.fastest {
    background: linear-gradient(135deg, #a8e6cf 0%, #88d8c0 100%);
    color: #333;
  }

  .stat-item.slowest {
    background: linear-gradient(135deg, #ffb3ba 0%, #ff8b94 100%);
    color: #333;
  }

  .stat-label {
    font-size: 0.9rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
  }

  .stat-value {
    font-size: 1.2rem;
    font-weight: 700;
    font-family: 'Courier New', monospace;
  }

  .lap-list {
    background: #f8f9fa;
    border-radius: 15px;
    overflow: hidden;
    max-height: 300px;
    overflow-y: auto;
  }

  .lap-item {
    display: grid;
    grid-template-columns: 1fr 2fr 2fr;
    padding: 1rem;
    border-bottom: 1px solid #e9ecef;
    transition: background-color 0.3s ease;
  }

  .lap-item:last-child {
    border-bottom: none;
  }

  .lap-item.header {
    background: #e9ecef;
    font-weight: 700;
    color: #495057;
    position: sticky;
    top: 0;
  }

  .lap-item.fastest {
    background: linear-gradient(135deg, #a8e6cf 0%, #88d8c0 100%);
  }

  .lap-item.slowest {
    background: linear-gradient(135deg, #ffb3ba 0%, #ff8b94 100%);
  }

  .lap-number {
    font-weight: 600;
    color: #495057;
  }

  .lap-time, .total-time {
    font-family: 'Courier New', monospace;
    font-weight: 600;
  }

  .lap-time {
    color: #28a745;
  }

  .total-time {
    color: #6c757d;
  }

  .instructions {
    background: #f8f9fa;
    border-radius: 15px;
    padding: 1.5rem;
    text-align: left;
  }

  .instructions h4 {
    color: #333;
    margin-bottom: 1rem;
    font-size: 1.1rem;
  }

  .instructions ul {
    list-style: none;
    padding: 0;
  }

  .instructions li {
    padding: 0.5rem 0;
    border-bottom: 1px solid #e9ecef;
  }

  .instructions li:last-child {
    border-bottom: none;
  }

  .instructions strong {
    color: #667eea;
  }

  @media (max-width: 768px) {
    .main-time {
      font-size: 2.5rem;
    }
    
    .control-buttons {
      flex-direction: column;
      align-items: center;
    }
    
    .btn {
      width: 100%;
      max-width: 200px;
    }
    
    .lap-stats {
      flex-direction: column;
      align-items: center;
      gap: 1rem;
    }
    
    .lap-item {
      grid-template-columns: 1fr 1fr 1fr;
      font-size: 0.9rem;
    }
    
    .lap-header {
      flex-direction: column;
      gap: 1rem;
    }
  }
</style>
