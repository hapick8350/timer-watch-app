<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  export let savedTimers = [];

  function formatTime(hours, minutes, seconds) {
    const totalSeconds = hours * 3600 + minutes * 60 + seconds;
    const h = Math.floor(totalSeconds / 3600);
    const m = Math.floor((totalSeconds % 3600) / 60);
    const s = totalSeconds % 60;
    return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
  }

  function deleteTimer(index) {
    dispatch('deleteTimer', index);
  }

  function copyToClipboard(timer) {
    const timerText = `${timer.name}: ${formatTime(timer.hours, timer.minutes, timer.seconds)}`;
    navigator.clipboard.writeText(timerText).then(() => {
      // 복사 성공 알림 (간단한 토스트 메시지)
      showToast('타이머 정보가 클립보드에 복사되었습니다!');
    }).catch(err => {
      console.error('클립보드 복사 실패:', err);
    });
  }

  function showToast(message) {
    // 간단한 토스트 메시지 표시
    const toast = document.createElement('div');
    toast.className = 'toast';
    toast.textContent = message;
    document.body.appendChild(toast);
    
    setTimeout(() => {
      toast.classList.add('show');
    }, 100);
    
    setTimeout(() => {
      toast.classList.remove('show');
      setTimeout(() => {
        document.body.removeChild(toast);
      }, 300);
    }, 2000);
  }

  function getTimerCategory(timer) {
    const totalMinutes = timer.hours * 60 + timer.minutes;
    
    if (totalMinutes <= 5) return '짧은 타이머';
    if (totalMinutes <= 15) return '중간 타이머';
    if (totalMinutes <= 30) return '긴 타이머';
    return '매우 긴 타이머';
  }

  function getCategoryIcon(category) {
    switch (category) {
      case '짧은 타이머': return '⚡';
      case '중간 타이머': return '⏱️';
      case '긴 타이머': return '🕐';
      case '매우 긴 타이머': return '⏰';
      default: return '⏱️';
    }
  }
</script>

<div class="preset-timers-container">
  {#if savedTimers.length === 0}
    <div class="empty-state">
      <div class="empty-icon">💾</div>
      <h3>저장된 타이머가 없습니다</h3>
      <p>타이머 탭에서 원하는 타이머를 설정하고 저장해보세요!</p>
      <div class="empty-features">
        <div class="feature-item">
          <span class="feature-icon">🎯</span>
          <span>자주 사용하는 타이머 저장</span>
        </div>
        <div class="feature-item">
          <span class="feature-icon">⚡</span>
          <span>빠른 접근</span>
        </div>
        <div class="feature-item">
          <span class="feature-icon">🎨</span>
          <span>개인화된 색상</span>
        </div>
      </div>
    </div>
  {:else}
    <div class="preset-header">
      <h3>저장된 타이머 ({savedTimers.length})</h3>
      <p>자주 사용하는 타이머들을 빠르게 불러올 수 있습니다</p>
    </div>

    <div class="timers-grid">
      {#each savedTimers as timer, index}
        <div class="timer-card" style="--timer-color: {timer.color}">
          <div class="timer-header">
            <div class="timer-info">
              <h4 class="timer-name">{timer.name}</h4>
              <div class="timer-category">
                {getCategoryIcon(getTimerCategory(timer))} {getTimerCategory(timer)}
              </div>
            </div>
            <div class="timer-actions">
              <button 
                class="action-btn copy-btn" 
                on:click={() => copyToClipboard(timer)}
                title="클립보드에 복사"
              >
                📋
              </button>
              <button 
                class="action-btn delete-btn" 
                on:click={() => deleteTimer(index)}
                title="삭제"
              >
                🗑️
              </button>
            </div>
          </div>
          
          <div class="timer-time">
            {formatTime(timer.hours, timer.minutes, timer.seconds)}
          </div>
          
          <div class="timer-details">
            <div class="detail-item">
              <span class="detail-label">시간:</span>
              <span class="detail-value">{timer.hours}시간</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">분:</span>
              <span class="detail-value">{timer.minutes}분</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">초:</span>
              <span class="detail-value">{timer.seconds}초</span>
            </div>
          </div>
          
          <div class="timer-footer">
            <span class="created-info">저장됨</span>
          </div>
        </div>
      {/each}
    </div>

    <div class="preset-actions">
      <button class="btn btn-primary" on:click={() => window.location.reload()}>
        🔄 새로고침
      </button>
    </div>
  {/if}
</div>

<style>
  .preset-timers-container {
    text-align: center;
  }

  .empty-state {
    padding: 3rem 2rem;
    color: #666;
  }

  .empty-icon {
    font-size: 4rem;
    margin-bottom: 1rem;
  }

  .empty-state h3 {
    color: #333;
    margin-bottom: 1rem;
    font-size: 1.5rem;
  }

  .empty-state p {
    margin-bottom: 2rem;
    font-size: 1.1rem;
  }

  .empty-features {
    display: flex;
    justify-content: center;
    gap: 2rem;
    flex-wrap: wrap;
  }

  .feature-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 10px;
    font-weight: 500;
  }

  .feature-icon {
    font-size: 1.2rem;
  }

  .preset-header {
    margin-bottom: 2rem;
  }

  .preset-header h3 {
    color: #333;
    font-size: 1.5rem;
    margin-bottom: 0.5rem;
  }

  .preset-header p {
    color: #666;
    font-size: 1rem;
  }

  .timers-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
  }

  .timer-card {
    background: white;
    border-radius: 20px;
    padding: 1.5rem;
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
    border: 3px solid transparent;
    background-clip: padding-box;
    position: relative;
    transition: all 0.3s ease;
    overflow: hidden;
  }

  .timer-card::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 4px;
    background: var(--timer-color);
  }

  .timer-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 20px 40px rgba(0,0,0,0.15);
  }

  .timer-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 1rem;
  }

  .timer-info {
    flex: 1;
  }

  .timer-name {
    color: #333;
    font-size: 1.3rem;
    font-weight: 700;
    margin: 0 0 0.5rem 0;
  }

  .timer-category {
    color: #666;
    font-size: 0.9rem;
    font-weight: 500;
  }

  .timer-actions {
    display: flex;
    gap: 0.5rem;
  }

  .action-btn {
    background: none;
    border: none;
    font-size: 1.2rem;
    cursor: pointer;
    padding: 0.5rem;
    border-radius: 8px;
    transition: all 0.3s ease;
  }

  .action-btn:hover {
    background: #f8f9fa;
    transform: scale(1.1);
  }

  .copy-btn:hover {
    background: #e3f2fd;
  }

  .delete-btn:hover {
    background: #ffebee;
  }

  .timer-time {
    font-size: 2.5rem;
    font-weight: 700;
    font-family: 'Courier New', monospace;
    color: #333;
    margin: 1rem 0;
    text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
  }

  .timer-details {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    margin-bottom: 1rem;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 10px;
  }

  .detail-item {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .detail-label {
    font-size: 0.8rem;
    color: #666;
    font-weight: 600;
    margin-bottom: 0.3rem;
  }

  .detail-value {
    font-size: 1rem;
    color: #333;
    font-weight: 700;
  }

  .timer-footer {
    text-align: center;
  }

  .created-info {
    font-size: 0.8rem;
    color: #999;
    font-style: italic;
  }

  .preset-actions {
    margin-top: 2rem;
  }

  .btn {
    padding: 1rem 2rem;
    border: none;
    border-radius: 50px;
    font-size: 1.1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
  }

  .btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.2);
  }

  .toast {
    position: fixed;
    top: 20px;
    right: 20px;
    background: #333;
    color: white;
    padding: 1rem 1.5rem;
    border-radius: 10px;
    font-weight: 500;
    z-index: 1000;
    opacity: 0;
    transform: translateX(100%);
    transition: all 0.3s ease;
  }

  .toast.show {
    opacity: 1;
    transform: translateX(0);
  }

  @media (max-width: 768px) {
    .timers-grid {
      grid-template-columns: 1fr;
    }
    
    .empty-features {
      flex-direction: column;
      align-items: center;
    }
    
    .timer-details {
      grid-template-columns: 1fr;
      gap: 0.5rem;
    }
    
    .timer-time {
      font-size: 2rem;
    }
    
    .timer-header {
      flex-direction: column;
      align-items: flex-start;
      gap: 1rem;
    }
    
    .timer-actions {
      align-self: flex-end;
    }
  }
</style>
