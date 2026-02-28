<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900">
    
    <!-- View 1: Setup -->
    <div v-if="currentView === 'setup'" class="min-h-screen flex items-center justify-center p-6">
      <div class="w-full max-w-xl">
        <div class="text-center mb-12">
          <div class="text-6xl mb-6">🎙️</div>
          <h1 class="text-6xl font-bold mb-4 text-transparent bg-clip-text bg-gradient-to-r from-blue-400 via-indigo-400 to-purple-500">DeepCast</h1>
          <p class="text-xl text-gray-400">进行深度研究并转化为引人入胜的播客</p>
        </div>
          
        <div class="card glass-panel rounded-2xl">
          <form @submit.prevent="startProduction" class="card-body p-8">
            <div class="form-control mb-6">
              <textarea 
                v-model="form.topic" 
                class="w-full textarea textarea-bordered h-32 text-lg leading-relaxed resize-none macos-input rounded-xl" 
                placeholder="💡 请输入播客主题（例如：AI Agent 的发展趋势）"
                required
                @keydown.enter.prevent="startProduction"></textarea>
            </div>
              
            <div class="alert bg-blue-500/10 border border-blue-500/20 mb-8 rounded-xl">
              <span class="text-sm text-blue-300 font-medium">🔍 使用混合搜索引擎 (Tavily + SerpApi)</span>
            </div>

            <button 
              class="btn btn-lg w-full font-semibold rounded-xl macos-btn-primary border-0" 
              :disabled="!form.topic.trim()">
              ✨ 开始制作播客
            </button>
          </form>
        </div>
      </div>
    </div>

    <!-- View 2: Production -->
    <div v-else-if="currentView === 'producing'" class="min-h-screen p-6">
      <div class="max-w-7xl mx-auto">
      <!-- Navbar / Header -->
      <div class="nav-glass rounded-xl shadow-lg mb-6 px-6 py-4">
        <div class="flex items-center justify-between gap-4">
          <div class="flex items-center gap-3">
            <span class="text-3xl filter drop-shadow-md">🎙️</span>
            <span class="text-2xl font-bold text-white tracking-tight">DeepCast</span>
          </div>
          <div class="flex items-center gap-3">
            <button v-if="reportReady" class="btn btn-ghost btn-sm text-blue-300 hover:bg-white/5" @click="downloadReport">
              📄 下载研究报告
            </button>
            <button v-if="!podcastReady" class="btn btn-ghost btn-sm text-red-400 hover:bg-white/5" @click="cancelProduction">
              取消制作
            </button>
          </div>
        </div>
      </div>

      <!-- Main Content -->
      <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">
        
        <!-- Left Column: Progress Steps -->
        <div class="lg:col-span-1">
          <div class="card glass-panel h-[500px] rounded-xl">
            <div class="card-body p-6 relative overflow-hidden">
               <!-- Decorative element -->
               <div class="absolute top-0 right-0 -mr-8 -mt-8 w-40 h-40 bg-blue-500/20 rounded-full blur-3xl"></div>
               <div class="absolute bottom-0 left-0 -ml-8 -mb-8 w-40 h-40 bg-purple-500/20 rounded-full blur-3xl"></div>

              <h2 class="text-xl font-bold text-white mb-8 flex items-center justify-center gap-3 z-10">
                <div class="p-2 bg-white/10 rounded-lg backdrop-blur-md shadow-inner border border-white/5">
                    <span v-if="productionStage === 'done'" class="text-2xl">✅</span>
                    <span v-else class="text-3xl animate-spin-slow inline-block">🔄</span>
                </div>
                <span class="tracking-wide">制作流程</span>
              </h2>
              
              <div class="flex-1 w-full flex justify-center pl-4">
                  <ul class="steps steps-vertical font-medium w-full h-full justify-evenly">
                    <li class="step gap-3" :class="getStepClass('research')">
                      <div class="flex flex-col text-left py-2 min-w-[120px]">
                        <div class="flex items-center gap-2">
                            <span class="text-xl filter drop-shadow" :class="{ 'animate-bounce': productionStage === 'research' }">🔍</span>
                            <span class="font-bold text-gray-200">深度研究</span>
                        </div>
                        <span class="text-xs text-gray-400 font-normal ml-8 mt-1">网络搜索 & 信息聚合</span>
                      </div>
                    </li>
                    <li class="step gap-3" :class="getStepClass('script')">
                        <div class="flex flex-col text-left py-2 min-w-[120px]">
                            <div class="flex items-center gap-2">
                                <span class="text-xl filter drop-shadow" :class="{ 'animate-bounce': productionStage === 'script' }">✍️</span>
                                <span class="font-bold text-gray-200">剧本创作</span>
                            </div>
                            <span class="text-xs text-gray-400 font-normal ml-8 mt-1">生成对话 & 角色分配</span>
                        </div>
                    </li>
                    <li class="step gap-3" :class="getStepClass('audio')">
                        <div class="flex flex-col text-left py-2 min-w-[120px]">
                            <div class="flex items-center gap-2">
                                <span class="text-xl filter drop-shadow" :class="{ 'animate-bounce': productionStage === 'audio' }">🎵</span>
                                <span class="font-bold text-gray-200">音频合成</span>
                            </div>
                            <span class="text-xs text-gray-400 font-normal ml-8 mt-1">TTS 语音生成 & 拼接</span>
                        </div>
                    </li>
                    <li class="step gap-3" :class="{ 'step-primary': podcastReady || productionStage === 'done' }">
                        <div class="flex flex-col text-left py-2 min-w-[120px]">
                            <div class="flex items-center gap-2">
                                <span class="text-xl filter drop-shadow" :class="{ 'animate-pulse': podcastReady }">🎉</span>
                                <span class="font-bold text-gray-200">完成</span>
                            </div>
                            <span class="text-xs text-gray-400 font-normal ml-8 mt-1">播放 & 下载播客</span>
                        </div>
                    </li>
                  </ul>
              </div>
            </div>
          </div>
        </div>

        <!-- Right Column: Logs & Output -->
        <div class="lg:col-span-3 flex flex-col gap-4">
          
          <!-- macOS Style Terminal -->
          <div class="macos-terminal rounded-xl shadow-2xl overflow-hidden" style="height: 500px;">
             <!-- macOS Title Bar -->
             <div class="macos-titlebar bg-gradient-to-b from-[#3d3d3d] to-[#2d2d2d] px-4 py-3 flex items-center shrink-0 border-b border-[#1a1a1a]">
                <!-- Traffic Lights -->
                <div class="flex items-center gap-2 mr-4">
                   <div class="w-3 h-3 rounded-full bg-[#ff5f57] shadow-inner hover:brightness-110 cursor-pointer transition-all" title="关闭"></div>
                   <div class="w-3 h-3 rounded-full bg-[#febc2e] shadow-inner hover:brightness-110 cursor-pointer transition-all" title="最小化"></div>
                   <div class="w-3 h-3 rounded-full bg-[#28c840] shadow-inner hover:brightness-110 cursor-pointer transition-all" title="最大化"></div>
                </div>
                <!-- Title -->
                <div class="flex-1 text-center">
                   <span class="text-[#9a9a9a] text-sm font-medium tracking-wide">deepcast — zsh — {{ logs.length }} lines</span>
                </div>
                <!-- Placeholder for symmetry -->
                <div class="w-16"></div>
             </div>
             <!-- Terminal Content -->
             <div class="bg-[#1e1e1e] overflow-y-auto p-4 flex-1 font-mono text-sm custom-scrollbar terminal-content" ref="logContainer" style="height: calc(100% - 44px);">
                <!-- Welcome Message -->
                <div v-if="logs.length === 0 && !isWaiting" class="text-[#6a9955] mb-2">
                  <span class="text-[#569cd6]">deepcast</span><span class="text-[#d4d4d4]">@</span><span class="text-[#4ec9b0]">studio</span> <span class="text-[#d4d4d4]">~</span> <span class="text-[#dcdcaa]">ready</span>
                </div>
                <!-- Log Entries -->
                <div v-for="(log, i) in logs" :key="i" class="mb-1 leading-relaxed" :class="getLogClass(log.message)">
                  <span class="text-[#6a6a6a] mr-2 text-xs select-none">[{{ log.time }}]</span>
                  <span class="terminal-text">{{ log.message }}</span>
                </div>
                <!-- Waiting States -->
                <div v-if="isWaiting && logs.length === 0" class="text-[#dcdcaa] text-center mt-8">
                  <span class="inline-block animate-pulse">⏳ 正在初始化...</span>
                </div>
                <div v-else-if="isWaiting" class="text-[#dcdcaa] mt-2 flex items-center gap-2">
                  <span class="inline-block w-2 h-4 bg-[#569cd6] animate-blink"></span>
                  <span>处理中{{ waitingDots }}</span>
                </div>
             </div>
          </div>

          <!-- Result Actions -->
          <div v-if="podcastReady" class="flex gap-4">
               <a :href="audioUrl" download class="btn macos-btn-primary flex-1 btn-lg text-lg rounded-xl border-0">
                  ⬇️ 下载 MP3
               </a>
               <button class="btn glass text-white flex-1 btn-lg text-lg rounded-xl" @click="currentView = 'player'">
                  🎧 进入播放器
               </button>
          </div>
          
          <!-- Inline Player -->
           <div v-if="podcastReady" class="card glass-panel rounded-xl mt-2">
             <div class="card-body p-4">
               <div class="flex items-center gap-3 mb-2">
                 <span class="text-xl">🎧</span>
                 <h3 class="text-sm font-bold text-gray-200">快速试听</h3>
               </div>
               <audio class="w-full opacity-90 hover:opacity-100 transition-opacity" :src="audioUrl" controls></audio>
             </div>
           </div>

        </div>
      </div>
    </div>
    </div>

    <!-- View 3: Player -->
    <div v-else-if="currentView === 'player'" class="hero min-h-screen">
      <div class="hero-content flex-col lg:flex-row-reverse gap-8 w-full max-w-6xl items-start">
         <!-- Right: Report -->
         <div class="card glass-panel shadow-2xl flex-1 h-[70vh] w-full lg:w-3/5 overflow-hidden rounded-2xl border border-white/10">
            <div class="card-body p-0 flex flex-col h-full bg-black/20">
              <div class="p-6 border-b border-white/10 sticky top-0 z-10 bg-black/40 backdrop-blur-md">
                <div class="flex items-center justify-between">
                  <h2 class="card-title text-white">📄 研究报告</h2>
                  <button class="btn btn-xs btn-ghost text-white/50" @click="downloadReport">下载</button>
                </div>
              </div>
              <div class="overflow-y-auto p-8 custom-scrollbar flex-1 text-gray-200">
                <article class="prose prose-sm prose-invert max-w-none" v-html="md.render(reportMarkdown)"></article>
              </div>
            </div>
         </div>

         <!-- Left: Player -->
         <div class="card glass-panel shadow-2xl flex-shrink-0 w-full lg:w-2/5 text-center h-auto rounded-2xl border border-white/10">
            <figure class="px-10 pt-12 pb-4">
              <div class="avatar placeholder">
                <div class="bg-black/40 text-white rounded-full w-48 h-48 ring-4 ring-white/10 shadow-[0_0_50px_rgba(0,0,0,0.5)] flex items-center justify-center relative overflow-hidden backdrop-blur-md">
                   <!-- Vinyl Animation -->
                   <div class="absolute inset-0 border-[2px] border-white/5 rounded-full" style="margin: 2px"></div>
                   <div class="absolute inset-0 border-[2px] border-white/5 rounded-full" style="margin: 10px"></div>
                   <div class="absolute inset-0 border-[2px] border-white/5 rounded-full" style="margin: 20px"></div>
                   
                   <div class="absolute inset-0 border-[10px] border-black/60 rounded-full opacity-40" :class="{ 'animate-spin': isPlaying }" style="animation-duration: 4s;"></div>
                   
                   <!-- Center Label -->
                   <div class="z-10 w-16 h-16 rounded-full bg-gradient-to-tr from-blue-500 to-purple-500 shadow-inner flex items-center justify-center">
                     <span class="text-xl font-bold text-white">DC</span>
                   </div>
                </div>
              </div>
            </figure>
            <div class="card-body items-center text-center pt-2">
              <h2 class="card-title text-2xl text-white font-bold drop-shadow-md">{{ form.topic }}</h2>
              <p class="text-blue-200/60 text-sm font-medium tracking-widest uppercase mb-6">DeepCast Original</p>
              
              <div class="w-full bg-black/30 rounded-xl p-4 border border-white/5 shadow-inner">
                 <audio 
                    ref="audioPlayer" 
                    :src="audioUrl" 
                    controls 
                    class="w-full"
                    @play="isPlaying = true"
                    @pause="isPlaying = false"
                 ></audio>
              </div>
              
              <div class="card-actions mt-8 w-full gap-3 flex-col">
                <a :href="audioUrl" download class="btn macos-btn-primary w-full border-0 rounded-xl text-lg h-12">
                  ⬇️ 下载 MP3
                </a>
                <button class="btn btn-ghost text-white/50 hover:text-white w-full" @click="resetApp">
                  🪄 制作新播客
                </button>
              </div>
            </div>
         </div>
      </div>
    </div>

  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, nextTick } from "vue";
import { runResearchStream, cancelResearch, type ResearchStreamEvent } from "./services/api";
import MarkdownIt from "markdown-it";

// Markdown renderer
const md = new MarkdownIt();

// --- Types ---
type ViewState = "setup" | "producing" | "player";
type ProductionStage = "research" | "script" | "audio" | "done";

interface LogEntry {
  time: string;
  message: string;
}

// --- State ---
const currentView = ref<ViewState>("setup");
const productionStage = ref<ProductionStage>("research");
const form = reactive({
  topic: ""
});

const logs = ref<LogEntry[]>([]);
const isPlaying = ref(false);
const reportReady = ref(false);
const podcastReady = ref(false);

const audioProgress = reactive({
  current: 0,
  total: 0,
  role: ""
});

const currentStatusMessage = ref("");
const isWaiting = ref(false);
const waitingDots = ref(".");
let waitingInterval: ReturnType<typeof setInterval> | null = null;

const reportMarkdown = ref("");
const audioUrl = ref("");

const audioPlayer = ref<HTMLAudioElement | null>(null);
const logContainer = ref<HTMLElement | null>(null);
let abortController: AbortController | null = null;

// --- Helpers ---

function startWaitingAnimation() {
  stopWaitingAnimation();
  isWaiting.value = true;
  waitingDots.value = ".";
  waitingInterval = setInterval(() => {
    waitingDots.value = waitingDots.value.length >= 3 ? "." : waitingDots.value + ".";
  }, 500);
}

function stopWaitingAnimation() {
  isWaiting.value = false;
  if (waitingInterval) {
    clearInterval(waitingInterval);
    waitingInterval = null;
  }
}

function getLogClass(message: string): string {
  // macOS Terminal style colors
  if (message.includes("[STAGE]")) return "terminal-stage";
  if (message.includes("[TASK")) return "terminal-info";
  if (message.includes("[TOOL]")) return "terminal-tool";
  if (message.includes("[SOURCES]")) return "terminal-warning";
  if (message.includes("✅") || message.includes("status=completed")) return "terminal-success";
  if (message.includes("❌") || message.includes("ERROR") || message.includes("failed")) return "terminal-error";
  if (message.includes("⚠️") || message.includes("WARNING")) return "terminal-warning";
  if (message.includes("INFO:")) return "terminal-muted";
  if (message.includes("━")) return "terminal-divider";
  return "terminal-default";
}

function getStepClass(step: ProductionStage) {
  const stepsOrder = ["research", "script", "audio", "done"];
  const currentIdx = stepsOrder.indexOf(productionStage.value);
  const stepIdx = stepsOrder.indexOf(step);
  
  if (currentIdx > stepIdx) return "step-primary"; // Completed
  if (currentIdx === stepIdx) return "step-primary font-bold"; // Active
  return "";
}

function addLog(message: string) {
  const time = new Date().toLocaleTimeString([], { hour12: false, hour: "2-digit", minute: "2-digit", second: "2-digit" });
  logs.value.push({ time, message });
  nextTick(() => {
    if (logContainer.value) {
      logContainer.value.scrollTop = logContainer.value.scrollHeight;
    }
  });
}

// --- Actions ---

async function startProduction() {
  if (!form.topic.trim()) return;

  // Reset State
  currentView.value = "producing";
  productionStage.value = "research";
  logs.value = [];
  reportMarkdown.value = "";
  audioUrl.value = "";
  audioProgress.current = 0;
  audioProgress.total = 0;
  currentStatusMessage.value = "正在初始化...";
  reportReady.value = false;
  podcastReady.value = false;

  abortController = new AbortController();
  startWaitingAnimation();

  addLog("🚀 启动 DeepCast 制作流程...");
  addLog(`📌 主题: ${form.topic}`);

  try {
    await runResearchStream(
      { topic: form.topic },
      handleStreamEvent,
      { signal: abortController.signal }
    );
  } catch (err: any) {
    if (err.name === "AbortError" || err.message?.includes("aborted")) {
      addLog("🛑 制作已取消。");
    } else {
      addLog(`❌ 错误: ${err.message || err}`);
      console.error(err);
    }
  } finally {
    stopWaitingAnimation();
  }
}

function handleStreamEvent(event: ResearchStreamEvent) {
  console.log("Event:", event.type, event);

  // 1. Log Event
  if (event.type === "log") {
    const msg = String((event as any).message || "");
    // 去掉可能的颜色代码如果后端没去掉
    const cleanMsg = msg.replace(/\u001b\[\d+m/g, "");
    addLog(`INFO: ${cleanMsg}`);
    
    // 从日志中解析 TTS 进度 (作为备份机制)
    // 格式: [TTS 6/13] ✓ Host 语音生成成功
    const ttsMatch = cleanMsg.match(/\[TTS (\d+)\/(\d+)\]/);
    if (ttsMatch) {
      audioProgress.current = parseInt(ttsMatch[1], 10);
      audioProgress.total = parseInt(ttsMatch[2], 10);
      currentStatusMessage.value = `音频生成: ${audioProgress.current}/${audioProgress.total}`;
    }
    return;
  }

  // 2. Stage Change
  if (event.type === "stage_change") {
    const payload = event as any;
    const stage = payload.stage;
    const message = payload.message || "";
    currentStatusMessage.value = message;
    
    addLog(`━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━`);
    addLog(`📌 [STAGE] ${stage.toUpperCase()} - ${message}`);
    addLog(`━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━`);
    
    if (stage === "report") productionStage.value = "research";
    else if (stage === "script") productionStage.value = "script";
    else if (stage === "audio") productionStage.value = "audio";
    // Backend distinguishes a separate "synthesis" stage for final audio stitching,
    // but the UI groups it under the overall "audio" production stage for simplicity.
    else if (stage === "synthesis") productionStage.value = "audio";
  }

  // 3. Task / Tool Updates (Simplified logging)
  if (event.type === "tool_call") {
    const p = event as any;
    addLog(`🔧 [TOOL] ${p.tool} - ${p.agent || 'Agent'}`);
  }

  if (event.type === "task_status") {
    const p = event as any;
    if (p.status === "completed") {
      addLog(`✅ [TASK ${p.task_id}] ${p.title}`);
    } else if (p.status === "in_progress") {
      addLog(`🚀 [TASK ${p.task_id}] ${p.title} (In Progress)`);
    } else if (p.status === "failed") {
      addLog(`❌ [TASK ${p.task_id}] Failed: ${p.title}`);
    }
  }

  // 4. Report Ready
  if (event.type === "final_report") {
    reportMarkdown.value = String((event as any).report);
    reportReady.value = true;
    addLog(`📄 [REPORT] 报告已生成`);
  }

  // 5. Script Ready
  if (event.type === "podcast_script") {
    productionStage.value = "audio";
    addLog(`🎙️ [SCRIPT] 剧本已生成`);
  }

  // 6. Audio Progress
  if (event.type === "audio_start") {
    const p = event as any;
    audioProgress.total = p.total || 0;
    addLog(`🎵 [AUDIO] 开始生成音频, 共 ${audioProgress.total} 段`);
  }
  
  if (event.type === "audio_progress") {
    const p = event as any;
    audioProgress.current = p.current;
    audioProgress.total = p.total;
    currentStatusMessage.value = `生成音频: ${p.role} (${p.current}/${p.total})`;
  }

  // 7. Podcast Ready
  if (event.type === "podcast_ready") {
    const p = event as any;
    const filename = String(p.file).split(/[\\/]/).pop();
    if (filename) {
      const baseUrl = import.meta.env.VITE_API_BASE_URL || "http://localhost:8000";
      audioUrl.value = `${baseUrl}/output/audio/${filename}`;
      podcastReady.value = true;
      productionStage.value = "done";
      currentStatusMessage.value = "🎉 播客制作完成！";
      stopWaitingAnimation();
      addLog(`🎉 [PODCAST] 制作完成: ${filename}`);
    }
  }

  // 8. Done (Catch-all)
  if (event.type === "done") {
    addLog(`✅ [DONE] 所有任务结束`);
    stopWaitingAnimation();
    productionStage.value = "done";
    
    // 如果没有收到 podcast_ready 事件，尝试获取最新的音频文件
    if (!podcastReady.value && audioProgress.total > 0) {
      const baseUrl = import.meta.env.VITE_API_BASE_URL || "http://localhost:8000";
      // 尝试从后端获取最新的音频文件
      fetch(`${baseUrl}/api/audio/latest`)
        .then(res => res.json())
        .then(data => {
          if (data.file) {
            audioUrl.value = `${baseUrl}${data.url}`;
            podcastReady.value = true;
            currentStatusMessage.value = "🎉 播客制作完成！";
            addLog(`🎉 [PODCAST] 找到音频文件: ${data.file}`);
          } else {
            currentStatusMessage.value = "任务完成（音频未生成）";
            addLog(`⚠️ 未找到音频文件: ${data.error || '未知错误'}`);
          }
        })
        .catch(err => {
          currentStatusMessage.value = "任务完成（无法获取音频）";
          addLog(`⚠️ 获取音频文件失败: ${err.message}`);
        });
    } else if (podcastReady.value) {
      currentStatusMessage.value = "🎉 播客制作完成！";
    } else {
      currentStatusMessage.value = "任务完成（音频可能未生成）";
    }
  }
}

function cancelProduction() {
  if (confirm("确定要取消制作吗？")) {
    addLog("🛑 用户请求取消制作...");
    // 先通知后端停止，再断开 SSE 连接
    cancelResearch().then(() => {
      addLog("✅ 后端已接收取消请求");
    });
    if (abortController) {
      abortController.abort();
      abortController = null;
    }
    stopWaitingAnimation();
    productionStage.value = "done";
    
    // 给一点时间让状态重置
    setTimeout(() => {
      currentView.value = "setup";
      currentStatusMessage.value = "";
    }, 1000);
  }
}

function resetApp() {
  currentView.value = "setup";
  form.topic = "";
  isPlaying.value = false;
  currentStatusMessage.value = "";
  reportReady.value = false;
  podcastReady.value = false;
  audioUrl.value = "";
  stopWaitingAnimation();
}

function downloadReport() {
  if (!reportMarkdown.value) return;
  const blob = new Blob([reportMarkdown.value], { type: 'text/markdown;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `DeepCast深度研究报告.md`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}
</script>

<style scoped>
/* macOS Terminal Styles */
.macos-terminal {
  background: #1e1e1e;
  border: 1px solid #3d3d3d;
  box-shadow: 
    0 22px 70px 4px rgba(0, 0, 0, 0.56),
    0 0 0 1px rgba(0, 0, 0, 0.3);
}

.macos-titlebar {
  -webkit-app-region: drag;
  user-select: none;
}

.terminal-content {
  font-family: 'SF Mono', 'Monaco', 'Menlo', 'Consolas', monospace;
  font-size: 13px;
  line-height: 1.6;
}

/* Terminal Color Classes - VS Code Dark+ inspired */
.terminal-stage {
  color: #569cd6;
  font-weight: 600;
  padding-bottom: 2px;
  margin-bottom: 2px;
}

.terminal-info {
  color: #4fc1ff;
}

.terminal-tool {
  color: #c586c0;
}

.terminal-success {
  color: #4ec9b0;
}

.terminal-error {
  color: #f14c4c;
}

.terminal-warning {
  color: #dcdcaa;
}

.terminal-muted {
  color: #6a9955;
}

.terminal-divider {
  color: #3d3d3d;
  opacity: 0.8;
}

.terminal-default {
  color: #d4d4d4;
}

/* Blinking cursor animation */
@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

.animate-blink {
  animation: blink 1s step-end infinite;
}

/* Custom Scrollbar for log and report - macOS style */
.custom-scrollbar::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.15);
  border-radius: 4px;
}
.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.25);
}

/* Hide scrollbar when not hovering (macOS behavior) */
.terminal-content:not(:hover)::-webkit-scrollbar-thumb {
  background: transparent;
}

/* Animation for spinning loader */
@keyframes spin-slow {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.animate-spin-slow {
  animation: spin-slow 3s linear infinite;
}

/* macOS / Glassmorphism Design System */
.glass-panel {
  background: rgba(30, 30, 30, 0.7);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}

.macos-input {
  background: rgba(0, 0, 0, 0.2) !important;
  border: 1px solid rgba(255, 255, 255, 0.1) !important;
  color: #fff !important;
  transition: all 0.3s ease;
}

.macos-input:focus {
  background: rgba(0, 0, 0, 0.4) !important;
  border-color: #0A84FF !important; /* macOS Blue */
  box-shadow: 0 0 0 3px rgba(10, 132, 255, 0.2);
  outline: none;
}

.macos-btn-primary {
  background: linear-gradient(180deg, #0A84FF 0%, #007AFF 100%);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 1px 2px rgba(0,0,0,0.2), inset 0 1px 1px rgba(255,255,255,0.2);
  transition: all 0.2s;
}
.macos-btn-primary:hover {
  filter: brightness(1.05);
  transform: translateY(-0.5px);
  box-shadow: 0 4px 12px rgba(0, 122, 255, 0.3), inset 0 1px 1px rgba(255,255,255,0.2);
}
.macos-btn-primary:active {
  transform: translateY(0.5px);
  filter: brightness(0.95);
}
.macos-btn-primary:disabled {
  opacity: 0.5;
  filter: grayscale(0.5);
  transform: none;
  cursor: not-allowed;
}

.nav-glass {
  background: rgba(40, 40, 40, 0.85);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}
</style>
