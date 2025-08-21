<script>
  import Timer from './lib/Timer.svelte';
  import Stopwatch from './lib/Stopwatch.svelte';
  import PresetTimers from './lib/PresetTimers.svelte';
  import { onMount } from 'svelte';

  let activeTab = 'timer';
  let savedTimers = [];

  onMount(() => {
    // 로컬 스토리지에서 저장된 타이머 불러오기
    const saved = localStorage.getItem('savedTimers');
    if (saved) {
      savedTimers = JSON.parse(saved);
    }
  });

  function saveTimer(event) {
    const timer = event.detail;
    savedTimers = [...savedTimers, timer];
    localStorage.setItem('savedTimers', JSON.stringify(savedTimers));
  }

  function deleteTimer(event) {
    const index = event.detail;
    savedTimers = savedTimers.filter((_, i) => i !== index);
    localStorage.setItem('savedTimers', JSON.stringify(savedTimers));
  }
</script>

<main>
  <div class="app-container">
    <header>
      <h1>⏰ Timer-Watch</h1>
      <p>집중하고 성장하는 시간을 만들어보세요</p>
    </header>

    <nav class="tab-navigation">
      <button 
        class="tab-button {activeTab === 'timer' ? 'active' : ''}" 
        on:click={() => activeTab = 'timer'}
      >
        🕐 타이머
      </button>
      <button 
        class="tab-button {activeTab === 'stopwatch' ? 'active' : ''}" 
        on:click={() => activeTab = 'stopwatch'}
      >
        ⏱️ 스톱워치
      </button>
      <button 
        class="tab-button {activeTab === 'presets' ? 'active' : ''}" 
        on:click={() => activeTab = 'presets'}
      >
        ⭐ 프리셋
      </button>
    </nav>

    <div class="content">
      {#if activeTab === 'timer'}
        <Timer on:saveTimer={saveTimer} />
      {:else if activeTab === 'stopwatch'}
        <Stopwatch />
      {:else if activeTab === 'presets'}
        <PresetTimers {savedTimers} on:deleteTimer={deleteTimer} />
      {/if}
    </div>
  </div>
</main>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    color: #333;
  }

  .app-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem;
    min-height: 100vh;
  }

  header {
    text-align: center;
    margin-bottom: 2rem;
    color: white;
  }

  header h1 {
    font-size: 3rem;
    font-weight: 700;
    margin-bottom: 0.5rem;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
  }

  header p {
    font-size: 1.2rem;
    opacity: 0.9;
  }

  .tab-navigation {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .tab-button {
    background: rgba(255, 255, 255, 0.1);
    border: 2px solid rgba(255, 255, 255, 0.2);
    color: white;
    padding: 1rem 2rem;
    border-radius: 50px;
    cursor: pointer;
    font-size: 1.1rem;
    font-weight: 600;
    transition: all 0.3s ease;
    backdrop-filter: blur(10px);
  }

  .tab-button:hover {
    background: rgba(255, 255, 255, 0.2);
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.2);
  }

  .tab-button.active {
    background: rgba(255, 255, 255, 0.3);
    border-color: rgba(255, 255, 255, 0.5);
    box-shadow: 0 8px 25px rgba(0,0,0,0.2);
  }

  .content {
    background: rgba(255, 255, 255, 0.95);
    border-radius: 20px;
    padding: 2rem;
    box-shadow: 0 20px 40px rgba(0,0,0,0.1);
    backdrop-filter: blur(10px);
  }

  @media (max-width: 768px) {
    .app-container {
      padding: 1rem;
    }
    
    header h1 {
      font-size: 2rem;
    }
    
    .tab-navigation {
      flex-direction: column;
      align-items: center;
    }
    
    .tab-button {
      width: 100%;
      max-width: 300px;
    }
  }
</style>
