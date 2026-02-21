<template>
  <div class="game-container">
    <!-- 黑板背景 -->
    <div class="blackboard"></div>
    
    <!-- 游戏标题 -->
    <div class="game-title">🎓 双人英语单词大比拼</div>
    
    <!-- 游戏主界面 -->
    <div v-if="gameStatus === 'playing'" class="game-main">
      <!-- 公共倒计时 -->
      <div class="timer-area">
        <div class="timer-label">公共倒计时</div>
        <div class="timer-value" :class="{ danger: timeLeft <= 30 }">{{ formatTime(timeLeft) }}</div>
      </div>
      
      <!-- 玩家1区域 -->
      <div class="player player1">
        <div class="player-card">
          <div class="player-name">玩家1</div>
          <div class="player-score">{{ score1 }}</div>
          <div>积分</div>
        </div>
      </div>
      
      <!-- 答题区域（左右分栏） -->
      <div class="answer-area">
        <!-- 玩家1答题面板 -->
        <div class="answer-panel">
          <div class="panel-title">玩家1答题区</div>
          <div class="word-meaning">{{ player1Question.question }}</div>
          <div class="options">
            <div 
              class="option" 
              v-for="(option, index) in player1Question.options" 
              :key="index"
              :class="{ selected: selectedOption1 === index }"
              @click="selectedOption1 = index"
            >
              <span class="option-letter">{{ ['A', 'B', 'C', 'D'][index] }}</span>
              <span class="option-text">{{ option }}</span>
            </div>
          </div>
          <button 
            class="btn btn-primary submit-btn" 
            @click="submitAnswer(1)" 
            :disabled="selectedOption1 === -1"
          >
            提交答案
          </button>
          <div v-if="player1Feedback" class="player-feedback" :class="player1Correct ? 'correct' : 'wrong'">
            {{ player1Feedback }}
          </div>
        </div>
        
        <!-- 玩家2答题面板 -->
        <div class="answer-panel">
          <div class="panel-title">玩家2答题区</div>
          <div class="word-meaning">{{ player2Question.question }}</div>
          <div class="options">
            <div 
              class="option" 
              v-for="(option, index) in player2Question.options" 
              :key="index"
              :class="{ selected: selectedOption2 === index }"
              @click="selectedOption2 = index"
            >
              <span class="option-letter">{{ ['A', 'B', 'C', 'D'][index] }}</span>
              <span class="option-text">{{ option }}</span>
            </div>
          </div>
          <button 
            class="btn btn-primary submit-btn" 
            @click="submitAnswer(2)" 
            :disabled="selectedOption2 === -1"
          >
            提交答案
          </button>
          <div v-if="player2Feedback" class="player-feedback" :class="player2Correct ? 'correct' : 'wrong'">
            {{ player2Feedback }}
          </div>
        </div>
      </div>
      
      <!-- 玩家2区域 -->
      <div class="player player2">
        <div class="player-card">
          <div class="player-name">玩家2</div>
          <div class="player-score">{{ score2 }}</div>
          <div>积分</div>
        </div>
      </div>
    </div>
    
    <!-- 开始游戏界面 -->
    <div v-else-if="gameStatus === 'start'" class="start-screen">
      <div class="start-card">
        <h1>双人英语单词大比拼</h1>
        <p class="rule">游戏规则：</p>
        <ul class="rule-list">
          <li>公共倒计时2分钟，时间结束停止答题</li>
          <li>题目为Unit2单词/短语，看英文选中文释义</li>
          <li>双方题目独立随机抽取，答题互不干扰</li>
          <li>答对一题+10分，自动刷新下一道题</li>
          <li>答错不扣分，题目保持不变，可重新作答</li>
          <li>时间结束后积分高的一方获胜</li>
        </ul>
        <div class="btn-group">
          <button class="btn btn-primary start-btn" @click="startGame">开始游戏</button>
          <button class="btn btn-secondary manage-btn" @click="showManagePanel = true">管理题目</button>
        </div>
      </div>
    </div>
    
    <!-- 结束游戏界面 -->
    <div v-else-if="gameStatus === 'end'" class="end-screen">
      <div class="end-card">
        <h1>游戏结束！</h1>
        <div class="final-score">
          <div class="score-item">
            <div class="player-name">玩家1</div>
            <div class="score-value">{{ score1 }}</div>
          </div>
          <div class="vs">VS</div>
          <div class="score-item">
            <div class="player-name">玩家2</div>
            <div class="score-value">{{ score2 }}</div>
          </div>
        </div>
        <div class="result" :class="winner === 0 ? 'draw' : ''">
          {{ winner === 0 ? '平局！' : (winner === 1 ? '🎉 玩家1获胜！' : '🎉 玩家2获胜！') }}
        </div>
        <div class="btn-group">
          <button class="btn btn-primary start-btn" @click="startGame">再来一局</button>
          <button class="btn btn-secondary manage-btn" @click="showManagePanel = true">管理题目</button>
        </div>
      </div>
    </div>
    
    <!-- 题目管理弹窗 -->
    <div v-if="showManagePanel" class="modal-overlay" @click.self="showManagePanel = false">
      <div class="modal-content">
        <div class="modal-header">
          <h2>题目管理</h2>
          <button class="close-btn" @click="showManagePanel = false">×</button>
        </div>
        <div class="modal-body">
          <!-- 添加题目表单 -->
          <div class="add-form">
            <h3>添加新题目</h3>
            <div class="form-item">
              <label>题目：</label>
              <input type="text" v-model="newQuestion.question" placeholder="可输入英文单词/短语">
            </div>
            <div class="form-item">
              <label>选项A：</label>
              <input type="text" v-model="newQuestion.options[0]" placeholder="请输入选项A">
            </div>
            <div class="form-item">
              <label>选项B：</label>
              <input type="text" v-model="newQuestion.options[1]" placeholder="请输入选项B">
            </div>
            <div class="form-item">
              <label>选项C：</label>
              <input type="text" v-model="newQuestion.options[2]" placeholder="请输入选项C">
            </div>
            <div class="form-item">
              <label>选项D：</label>
              <input type="text" v-model="newQuestion.options[3]" placeholder="请输入选项D">
            </div>
            <div class="form-item">
              <label>正确答案：</label>
              <select v-model="newQuestion.correct">
                <option :value="0">A</option>
                <option :value="1">B</option>
                <option :value="2">C</option>
                <option :value="3">D</option>
              </select>
            </div>
            <button class="btn btn-primary add-btn" @click="addQuestion">添加题目</button>
          </div>
          
          <!-- 题目列表 -->
          <div class="question-list">
            <h3>现有题目（共{{ wordBank.length }}题）</h3>
            <div class="list-scroll">
              <div v-for="(question, index) in wordBank" :key="index" class="question-item">
                <div class="question-info">
                  <span class="q-index">{{ index + 1 }}.</span>
                  <span class="q-meaning">{{ question.question }}</span>
                  <span class="q-answer">（正确答案：{{ ['A', 'B', 'C', 'D'][question.correct] }}）</span>
                </div>
                <button class="delete-btn" @click="deleteQuestion(index)">删除</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { Howl } from 'howler';

// 1. 游戏状态
const gameStatus = ref('start'); // start/playing/end
const timeLeft = ref(120); // 2分钟倒计时
const timer = ref(null);
const backgroundMusic = ref(null); // 【修复】声明背景音乐变量

// 2. 玩家数据
const score1 = ref(0);
const score2 = ref(0);
// 玩家选择的选项
const selectedOption1 = ref(-1);
const selectedOption2 = ref(-1);
// 玩家答题反馈
const player1Feedback = ref('');
const player2Feedback = ref('');
const player1Correct = ref(false);
const player2Correct = ref(false);
// 玩家各自的题目
const player1Question = ref({});
const player2Question = ref({});
const winner = ref(0); // 0平局 1玩家1 2玩家2

// 3. 完整替换为你提供的单词/短语题库
const showManagePanel = ref(false);
const wordBank = ref([
  // --- 核心单词部分：英译中 + 中译英（24×2=48题，正确答案均匀覆盖A/B/C/D）---
  
  // 1. illegal 正确答案分布：B、D
  { question: 'illegal', options: ['合法的', '不合法的；非法的', '合理的', '强制的'], correct: 1 },
  { question: '不合法的；非法的', options: ['legal', 'reasonable', 'compulsory', 'illegal'], correct: 3 },
  
  // 2. hunt 正确答案分布：C、A
  { question: 'hunt', options: ['种植', '烹饪', '打猎；搜寻；追捕', '阅读'], correct: 2 },
  { question: '打猎；搜寻；追捕', options: ['hunt', 'plant', 'cook', 'read'], correct: 0 },
  
  // 3. immediately 正确答案分布：A、D
  { question: 'immediately', options: ['立刻；立即', '缓慢地', '逐渐地', '偶尔地'], correct: 0 },
  { question: '立刻；立即', options: ['slowly', 'gradually', 'occasionally', 'immediately'], correct: 3 },
  
  // 4. alarming 正确答案分布：B、C
  { question: 'alarming', options: ['令人安心的', '惊人的；使人惊恐的', '普通的', '无聊的'], correct: 1 },
  { question: '惊人的；使人惊恐的', options: ['reassuring', 'common', 'alarming', 'boring'], correct: 2 },
  
  // 5. rate 正确答案分布：A、D
  { question: 'rate', options: ['划分等级；比率', '数量', '质量', '速度'], correct: 0 },
  { question: '划分等级；比率', options: ['quantity', 'quality', 'speed', 'rate'], correct: 3 },
  
  // 6. extinct 正确答案分布：B、C
  { question: 'extinct', options: ['活跃的', '已灭绝的', '濒危的', '常见的'], correct: 1 },
  { question: '已灭绝的', options: ['active', 'endangered', 'extinct', 'common'], correct: 2 },
  
  // 7. aware 正确答案分布：A、D
  { question: 'aware', options: ['知道的；意识到的', '未知的', '模糊的', '陌生的'], correct: 0 },
  { question: '知道的；意识到的', options: ['unknown', 'vague', 'strange', 'aware'], correct: 3 },
  
  // 8. endanger 正确答案分布：B、C
  { question: 'endanger', options: ['保护', '使遭受危险；危害', '拯救', '帮助'], correct: 1 },
  { question: '使遭受危险；危害', options: ['protect', 'save', 'endanger', 'help'], correct: 2 },
  
  // 9. concern 正确答案分布：A、D
  { question: 'concern', options: ['涉及；让……担忧', '忽略', '放心', '忘记'], correct: 0 },
  { question: '涉及；让……担忧', options: ['ignore', 'relax', 'forget', 'concern'], correct: 3 },
  
  // 10. adapt 正确答案分布：A、B
  { question: 'adapt', options: ['适应；使适合', '拒绝', '放弃', '逃避'], correct: 0 },
  { question: '适应；使适合', options: ['refuse', 'adapt', 'abandon', 'escape'], correct: 1 },
  
  // 11. reserve 正确答案分布：B、C
  { question: 'reserve', options: ['取消', '保护区；预订；保留', '删除', '释放'], correct: 1 },
  { question: '保护区；预订；保留', options: ['cancel', 'delete', 'reserve', 'release'], correct: 2 },
  
  // 12. observe 正确答案分布：A、D
  { question: 'observe', options: ['观察；注视；遵守', '无视', '违反', '猜测'], correct: 0 },
  { question: '观察；注视；遵守', options: ['ignore', 'violate', 'guess', 'observe'], correct: 3 },
  
  // 13. beauty 正确答案分布：A、B
  { question: 'beauty', options: ['美；美人；美好的东西', '丑陋', '缺陷', '缺点'], correct: 0 },
  { question: '美；美人；美好的东西', options: ['ugliness', 'beauty', 'defect', 'shortcoming'], correct: 1 },
  
  // 14. effective 正确答案分布：B、C
  { question: 'effective', options: ['无效的', '有效的；生效的', '无用的', '虚假的'], correct: 1 },
  { question: '有效的；生效的', options: ['ineffective', 'useless', 'effective', 'false'], correct: 2 },
  
  // 15. recover 正确答案分布：B、D
  { question: 'recover', options: ['失去', '恢复；康复；找回', '恶化', '放弃'], correct: 1 },
  { question: '恢复；康复；找回', options: ['lose', 'worsen', 'abandon', 'recover'], correct: 3 },
  
  // 16. remove 正确答案分布：C、A
  { question: 'remove', options: ['添加', '保留', '去除；移开；脱去', '安装'], correct: 2 },
  { question: '去除；移开；脱去', options: ['remove', 'add', 'keep', 'install'], correct: 0 },
  
  // 17. intend 正确答案分布：A、D
  { question: 'intend', options: ['打算；计划；想要', '拒绝', '放弃', '忘记'], correct: 0 },
  { question: '打算；计划；想要', options: ['refuse', 'abandon', 'forget', 'intend'], correct: 3 },
  
  // 18. threat 正确答案分布：B、C
  { question: 'threat', options: ['承诺', '威胁；危及', '保护', '安慰'], correct: 1 },
  { question: '威胁；危及', options: ['promise', 'protect', 'threat', 'comfort'], correct: 2 },
  
  // 19. exist 正确答案分布：A、B
  { question: 'exist', options: ['存在；生存', '消失', '灭亡', '虚构'], correct: 0 },
  { question: '存在；生存', options: ['disappear', 'exist', 'perish', 'fiction'], correct: 1 },
  
  // 20. harmony 正确答案分布：B、D
  { question: 'harmony', options: ['冲突', '和谐；融洽', '混乱', '矛盾'], correct: 1 },
  { question: '和谐；融洽', options: ['conflict', 'chaos', 'contradiction', 'harmony'], correct: 3 },
  
  // 21. reduce 正确答案分布：B、C
  { question: 'reduce', options: ['增加', '减少；降低', '提升', '扩大'], correct: 1 },
  { question: '减少；降低', options: ['increase', 'improve', 'reduce', 'expand'], correct: 2 },
  
  // 22. neighbourhood 正确答案分布：C、A
  { question: 'neighbourhood', options: ['城市', '乡村', '街区；临近的地方', '山区'], correct: 2 },
  { question: '街区；临近的地方', options: ['neighbourhood', 'city', 'countryside', 'mountain'], correct: 0 },
  
  // 23. emotion 正确答案分布：A、D
  { question: 'emotion', options: ['情感；情绪', '理智', '身体', '智力'], correct: 0 },
  { question: '情感；情绪', options: ['reason', 'body', 'intelligence', 'emotion'], correct: 3 },
  
  // 24. unusual 正确答案分布：C、B
  { question: 'unusual', options: ['寻常的', '普通的', '特别的；不寻常的', '平凡的'], correct: 2 },
  { question: '特别的；不寻常的', options: ['usual', 'unusual', 'ordinary', 'common'], correct: 1 },

  // --- 高频短语部分：英译中 + 中译英（14×2=28题，正确答案均匀覆盖A/B/C/D）---
  
  // 1. on earth 正确答案分布：B、D
  { question: 'on earth', options: ['在地球上', '究竟；到底', '在地面上', '在世界上'], correct: 1 },
  { question: '究竟；到底', options: ['on the earth', 'on the ground', 'in the world', 'on earth'], correct: 3 },
  
  // 2. die out 正确答案分布：C、A
  { question: 'die out', options: ['出生', '出现', '灭亡；逐渐消失', '存活'], correct: 2 },
  { question: '灭亡；逐渐消失', options: ['die out', 'be born', 'appear', 'survive'], correct: 0 },
  
  // 3. be aware of 正确答案分布：A、D
  { question: 'be aware of', options: ['意识到；知道', '无视', '忘记', '忽略'], correct: 0 },
  { question: '意识到；知道', options: ['ignore', 'forget', 'neglect', 'be aware of'], correct: 3 },
  
  // 4. on average 正确答案分布：C、B
  { question: 'on average', options: ['总计', '最多', '平均', '最少'], correct: 2 },
  { question: '平均', options: ['in total', 'on average', 'at most', 'at least'], correct: 1 },
  
  // 5. be concerned about 正确答案分布：B、C
  { question: 'be concerned about', options: ['对……冷漠', '为……担忧；关心', '对……无视', '对……放心'], correct: 1 },
  { question: '为……担忧；关心', options: ['be indifferent to', 'be blind to', 'be concerned about', 'be at ease about'], correct: 2 },
  
  // 6. adapt to 正确答案分布：A、D
  { question: 'adapt to', options: ['适应', '拒绝', '放弃', '逃避'], correct: 0 },
  { question: '适应', options: ['refuse', 'abandon', 'escape', 'adapt to'], correct: 3 },
  
  // 7. under pressure 正确答案分布：C、A
  { question: 'under pressure', options: ['在放松中', '无压力', '在压力下；承受压力', '空闲中'], correct: 2 },
  { question: '在压力下；承受压力', options: ['under pressure', 'in relaxation', 'stress-free', 'at leisure'], correct: 0 },
  
  // 8. make out 正确答案分布：B、D
  { question: 'make out', options: ['混淆', '看清；听清；分清', '无视', '忘记'], correct: 1 },
  { question: '看清；听清；分清', options: ['confuse', 'ignore', 'forget', 'make out'], correct: 3 },
  
  // 9. remind sb.of sb./sth. 正确答案分布：A、C
  { question: 'remind sb.of sb./sth.', options: ['使某人想起……', '使某人忘记', '使某人忽略', '使某人厌恶'], correct: 0 },
  { question: '使某人想起……', options: ['make sb.forget', 'make sb.ignore', 'remind sb.of sb./sth.', 'make sb.dislike'], correct: 2 },
  
  // 10. watch over 正确答案分布：A、B
  { question: 'watch over', options: ['保护；照管；监督', '无视', '伤害', '放弃'], correct: 0 },
  { question: '保护；照管；监督', options: ['ignore', 'watch over', 'harm', 'abandon'], correct: 1 },
  
  // 11. day and night 正确答案分布：C、D
  { question: 'day and night', options: ['偶尔', '断断续续', '日日夜夜；夜以继日', '白天'], correct: 2 },
  { question: '日日夜夜；夜以继日', options: ['occasionally', 'on and off', 'in the daytime', 'day and night'], correct: 3 },
  
  // 12. due to 正确答案分布：B、A
  { question: 'due to', options: ['所以', '由于；因为', '但是', '然而'], correct: 1 },
  { question: '由于；因为', options: ['due to', 'so', 'but', 'however'], correct: 0 },
  
  // 13. search for 正确答案分布：C、D
  { question: 'search for', options: ['隐藏', '放弃', '搜索；查找', '忽略'], correct: 2 },
  { question: '搜索；查找', options: ['hide', 'abandon', 'ignore', 'search for'], correct: 3 },
  
  // 14. stir up 正确答案分布：C、B
  { question: 'stir up', options: ['平息', '安抚', '激起；挑起', '熄灭'], correct: 2 },
  { question: '激起；挑起', options: ['calm down', 'stir up', 'comfort', 'put out'], correct: 1 }
]);
const newQuestion = ref({
  question: '',
  options: ['', '', '', ''],
  correct: 0
});

// 4. 核心方法
// 格式化时间
const formatTime = (seconds) => {
  const m = Math.floor(seconds / 60);
  const s = seconds % 60;
  return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
};

// 单独给玩家1刷新随机题目
const getPlayer1Question = () => {
  if (wordBank.value.length < 1) {
    alert('题库不能为空！');
    return false;
  }
  const randomIndex = Math.floor(Math.random() * wordBank.value.length);
  player1Question.value = wordBank.value[randomIndex];
  return true;
};

// 单独给玩家2刷新随机题目
const getPlayer2Question = () => {
  if (wordBank.value.length < 1) {
    alert('题库不能为空！');
    return false;
  }
  const randomIndex = Math.floor(Math.random() * wordBank.value.length);
  player2Question.value = wordBank.value[randomIndex];
  return true;
};

// 开始游戏
const startGame = () => {
  // 【修复】点击开始游戏时播放音乐（触发用户交互）
  if (backgroundMusic.value) {
    backgroundMusic.value.play();
  }
  
  // 初始化双方题目
  if (!getPlayer1Question() || !getPlayer2Question()) return;
  
  // 重置所有状态
  score1.value = 0;
  score2.value = 0;
  selectedOption1.value = -1;
  selectedOption2.value = -1;
  player1Feedback.value = '';
  player2Feedback.value = '';
  player1Correct.value = false;
  player2Correct.value = false;
  timeLeft.value = 120;
  gameStatus.value = 'playing';
  
  // 启动倒计时
  if (timer.value) clearInterval(timer.value);
  timer.value = setInterval(() => {
    timeLeft.value--;
    if (timeLeft.value <= 0) {
      endGame();
    }
  }, 1000);
};

// 提交答案（核心逻辑：答对才换题，答错不换题可重答）
const submitAnswer = (player) => {
  // 游戏未进行中不处理
  if (gameStatus.value !== 'playing') return;

  // 获取当前提交玩家的信息
  let selectedIndex, question, isCorrect;
  if (player === 1) {
    selectedIndex = selectedOption1.value;
    question = player1Question.value;
  } else {
    selectedIndex = selectedOption2.value;
    question = player2Question.value;
  }

  // 校验是否选择了选项
  if (selectedIndex === -1) return;
  
  // 判断答案是否正确
  isCorrect = selectedIndex === question.correct;

  // 分对错处理逻辑
  if (isCorrect) {
    // 答对：结算加分，给出正确反馈
    if (player === 1) {
      player1Correct.value = true;
      player1Feedback.value = `✅ 答对了！+10分，即将进入下一题`;
      score1.value += 10;
    } else {
      player2Correct.value = true;
      player2Feedback.value = `✅ 答对了！+10分，即将进入下一题`;
      score2.value += 10;
    }

    // 延迟后，仅刷新当前玩家的题目，重置状态
    setTimeout(() => {
      if (gameStatus.value === 'playing') {
        if (player === 1) {
          getPlayer1Question();
          selectedOption1.value = -1;
          player1Feedback.value = '';
          player1Correct.value = false;
        } else {
          getPlayer2Question();
          selectedOption2.value = -1;
          player2Feedback.value = '';
          player2Correct.value = false;
        }
      }
    }, 800);
  } else {
    // 答错：不加分，不换题，给出错误反馈，清空选择让玩家重答
    if (player === 1) {
      player1Correct.value = false;
      player1Feedback.value = `❌ 答错了，请重新选择`;
    } else {
      player2Correct.value = false;
      player2Feedback.value = `❌ 答错了，请重新选择`;
    }

    // 延迟后，仅清空当前玩家的选择，题目完全不变
    setTimeout(() => {
      if (gameStatus.value === 'playing') {
        if (player === 1) {
          selectedOption1.value = -1;
        } else {
          selectedOption2.value = -1;
        }
      }
    }, 800);
  }
};

// 结束游戏
const endGame = () => {
  if (timer.value) clearInterval(timer.value);
  gameStatus.value = 'end';
  
  // 判定胜负
  if (score1.value > score2.value) {
    winner.value = 1;
  } else if (score2.value > score1.value) {
    winner.value = 2;
  } else {
    winner.value = 0;
  }
};

// 5. 题目管理方法
// 添加题目
const addQuestion = () => {
  if (!newQuestion.value.question.trim()) {
    alert('请输入题目！');
    return;
  }
  for (let i = 0; i < 4; i++) {
    if (!newQuestion.value.options[i].trim()) {
      alert(`请输入选项${['A', 'B', 'C', 'D'][i]}！`);
      return;
    }
  }
  
  wordBank.value.push({
    question: newQuestion.value.question.trim(),
    options: newQuestion.value.options.map(o => o.trim()),
    correct: newQuestion.value.correct
  });
  
  // 重置表单
  newQuestion.value = { question: '', options: ['', '', '', ''], correct: 0 };
  alert('题目添加成功！');
};

// 删除题目
const deleteQuestion = (index) => {
  if (confirm('确定要删除这道题吗？')) {
    wordBank.value.splice(index, 1);
  }
};

// 6. 生命周期
onMounted(() => {
  // 【修复】仅初始化音乐，不自动播放（等待用户点击开始游戏）
  backgroundMusic.value = new Howl({
    src: ['https://incompetech.com/music/royalty-free/mp3-royaltyfree/Carefree.mp3'],
    loop: true,
    autoplay: false // 关闭自动播放
  });
});

onUnmounted(() => {
  if (timer.value) clearInterval(timer.value);
  // 【修复】组件卸载时停止音乐
  if (backgroundMusic.value) {
    backgroundMusic.value.stop();
  }
});
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Microsoft YaHei", sans-serif;
}

body {
  background-color: #f5f5f5;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-image: linear-gradient(to bottom, #e8f4f8, #f0f8fb);
}

.game-container {
  width: 95%;
  max-width: 1400px;
  height: 95vh;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0,0,0,0.1);
  display: flex;
  position: relative;
  overflow: hidden;
  margin: 0 auto;
}

/* 黑板背景 */
.blackboard {
  position: absolute;
  top: 50px;
  left: 50%;
  transform: translateX(-50%);
  width: 90%;
  height: 80%;
  background-color: #2c474d;
  border-radius: 8px;
  box-shadow: inset 0 0 20px rgba(0,0,0,0.3);
  z-index: 1;
}

/* 游戏标题 */
.game-title {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 24px;
  font-weight: bold;
  color: #333;
  z-index: 2;
}

/* 游戏主界面 */
.game-main {
  width: 100%;
  height: 100%;
  display: flex;
  position: relative;
  z-index: 2;
}

/* 公共倒计时 */
.timer-area {
  position: absolute;
  top: 60px;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  z-index: 3;
}

.timer-label {
  font-size: 16px;
  color: #fff;
  margin-bottom: 5px;
}

.timer-value {
  font-size: 48px;
  font-weight: bold;
  color: #fff;
  text-shadow: 0 0 10px rgba(0,0,0,0.3);
}

.timer-value.danger {
  color: #e74c3c;
}

/* 玩家区域 */
.player {
  width: 15%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.player1 {
  align-items: flex-end;
}

.player2 {
  align-items: flex-start;
}

.player-card {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
  text-align: center;
  width: 150px;
}

.player-name {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 10px;
  color: #333;
}

.player-score {
  font-size: 36px;
  font-weight: bold;
  color: #42b983;
  margin: 10px 0;
}

/* 答题区域 */
.answer-area {
  width: 70%;
  height: 100%;
  display: flex;
  justify-content: space-between;
  gap: 30px;
  padding: 100px 20px 20px;
}

.answer-panel {
  flex: 1;
  background-color: rgba(255,255,255,0.95);
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
}

.panel-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
  text-align: center;
  padding-bottom: 10px;
  border-bottom: 1px solid #eee;
}

.word-meaning {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin: 10px 0 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 5px;
  text-align: center;
}

/* 选项样式 */
.options {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
  flex: 1;
}

.option {
  padding: 12px 15px;
  border: 2px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 16px;
  display: flex;
  align-items: center;
}

.option-letter {
  display: inline-block;
  width: 30px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  background-color: #42b983;
  color: #fff;
  border-radius: 50%;
  margin-right: 15px;
  font-weight: bold;
  font-size: 16px;
}

.option-text {
  flex: 1;
  font-size: 16px;
}

.option:hover {
  border-color: #42b983;
  background-color: #f8f9fa;
}

.option.selected {
  background-color: #42b983;
  color: #fff;
  border-color: #42b983;
}

.option.selected .option-letter {
  background-color: #fff;
  color: #42b983;
}

/* 按钮样式 */
.btn {
  padding: 10px 25px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin: 0 10px;
  transition: all 0.3s;
}

.btn:disabled {
  background-color: #999;
  cursor: not-allowed;
}

.btn-primary {
  background-color: #42b983;
  color: #fff;
}

.btn-primary:hover:not(:disabled) {
  background-color: #359469;
}

.btn-secondary {
  background-color: #666;
  color: #fff;
}

.btn-secondary:hover {
  background-color: #444;
}

.submit-btn {
  width: 100%;
  padding: 12px;
  font-size: 16px;
}

.player-feedback {
  margin-top: 10px;
  text-align: center;
  font-weight: bold;
  font-size: 16px;
  height: 24px;
}

.player-feedback.correct {
  color: #42b983;
}

.player-feedback.wrong {
  color: #e74c3c;
}

/* 开始/结束界面 */
.start-screen, .end-screen {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  z-index: 2;
}

.start-card, .end-card {
  background-color: rgba(255,255,255,0.95);
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 5px 20px rgba(0,0,0,0.2);
  text-align: center;
  max-width: 600px;
}

.start-card h1, .end-card h1 {
  font-size: 32px;
  color: #333;
  margin-bottom: 20px;
}

.rule {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 10px;
}

.rule-list {
  text-align: left;
  margin: 0 auto 30px;
  max-width: 450px;
}

.rule-list li {
  margin: 10px 0;
  color: #666;
  line-height: 1.6;
}

.btn-group {
  display: flex;
  justify-content: center;
  gap: 20px;
}

.start-btn, .manage-btn {
  padding: 12px 30px;
  font-size: 18px;
}

/* 结束界面 */
.final-score {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 40px;
  margin: 30px 0;
}

.score-item {
  text-align: center;
}

.score-item .player-name {
  font-size: 20px;
  color: #333;
  margin-bottom: 10px;
}

.score-value {
  font-size: 48px;
  font-weight: bold;
  color: #42b983;
}

.vs {
  font-size: 24px;
  font-weight: bold;
  color: #666;
}

.result {
  font-size: 28px;
  font-weight: bold;
  color: #42b983;
  margin-bottom: 30px;
}

.result.draw {
  color: #666;
}

/* 题目管理弹窗 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}

.modal-content {
  background-color: #fff;
  border-radius: 10px;
  width: 90%;
  max-width: 800px;
  max-height: 80vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-header {
  padding: 20px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header h2 {
  font-size: 24px;
  color: #333;
}

.close-btn {
  background: none;
  border: none;
  font-size: 32px;
  color: #999;
  cursor: pointer;
  line-height: 1;
}

.close-btn:hover {
  color: #333;
}

.modal-body {
  padding: 20px;
  overflow-y: auto;
  display: flex;
  gap: 30px;
}

.add-form, .question-list {
  flex: 1;
}

.add-form h3, .question-list h3 {
  font-size: 18px;
  color: #333;
  margin-bottom: 15px;
}

.form-item {
  margin-bottom: 15px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.form-item label {
  width: 80px;
  text-align: right;
  font-size: 14px;
  color: #333;
}

.form-item input, .form-item select {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.add-btn {
  width: 100%;
  margin-top: 10px;
}

.list-scroll {
  max-height: 400px;
  overflow-y: auto;
}

.question-item {
  padding: 12px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.question-info {
  flex: 1;
}

.q-index {
  font-weight: bold;
  color: #333;
  margin-right: 5px;
}

.q-meaning {
  color: #333;
  margin-right: 10px;
}

.q-answer {
  color: #42b983;
  font-size: 14px;
}

.delete-btn {
  background-color: #e74c3c;
  color: #fff;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.delete-btn:hover {
  background-color: #c0392b;
}
</style>