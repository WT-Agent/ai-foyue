<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">佛曰解惑开示案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">感悟红尘理趣与赛博解压，点击“一键同款求问”即可即刻体验</p>
      </div>
      <div class="showcase-badge">红尘解惑 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索困惑内容、开示流派或禅理关键词..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.style }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">俗世困惑：</span>“{{ sample.trouble }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">禅修谶语：</span>{{ sample.whisper }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.trouble)">
            一键同款求问
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的解惑案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string): void;
}>();

const categories = ['全部', '职场内卷', '财富焦虑', '情感纠葛', '虚度光阴', '深夜内耗'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface ZenSample {
  id: number;
  category: string;
  trouble: string;
  style: string;
  whisper: string;
}

// 精选 30 条佛曰解惑开示案例
const raw30Samples: ZenSample[] = [
  {
    id: 1,
    category: '职场内卷',
    trouble: '每天加班到深夜，感觉身体和灵魂都被掏空，不知道为了什么。',
    style: '传统禅宗开示',
    whisper: '佛曰：行到水穷处，坐看云起时。汝背负之繁重，皆因欲念过多。放下今日事，安眠便是修行。'
  },
  {
    id: 2,
    category: '深夜内耗',
    trouble: '总是担心未来会失败，焦虑得整晚失眠，脑子里胡思乱想。',
    style: '当头棒喝警醒',
    whisper: '大棒敲击：明日之忧本无物，汝却在脑中自设监牢。今夜不睡，明日必败，自我感动的焦虑最是无用！'
  },
  {
    id: 3,
    category: '财富焦虑',
    trouble: '物价在涨，房贷压得喘不过气，存不下钱感觉随时会崩溃。',
    style: '九巨擘禅意论',
    whisper: '马斯克开示：物质财富的积累本质上是在抵抗熵增，但汝之内心能量已因焦虑而耗尽，当遵循初心，简化资产。'
  },
  {
    id: 4,
    category: '情感纠葛',
    trouble: '和喜欢的人分手了，每天都在看对方的社交动态，走不出来。',
    style: '温情心灵抚慰',
    whisper: '执念如春蚕吐丝。世间一切相遇皆有因果，放手非失去，而是给彼此的善缘腾出空间。'
  },
  {
    id: 5,
    category: '虚度光阴',
    trouble: '明知道该去看书考证，却总是抱着手机玩到深夜，内心负罪感极重。',
    style: '赛博木鱼解压',
    whisper: '系统警报：检测到拖延症妄念占用 99% CPU。立即运行磁盘清理，格式化短视频缓存，点击木鱼，功德加一。'
  },
  {
    id: 6,
    category: '职场内卷',
    trouble: '同事之间互相攀比，工作环境充满勾心斗角，感觉特别累。',
    style: '传统禅宗开示',
    whisper: '佛曰：万法唯心造。他人攀比是风吹幡动，汝若心如止水，则无风无浪，波澜不惊。'
  },
  {
    id: 7,
    category: '深夜内耗',
    trouble: '觉得自己快三十岁了还一事无成，处处不如人，非常自卑。',
    style: '当头棒喝警醒',
    whisper: '大棒敲击：时钟是凡俗造物，汝以此度量一生，无异于刻舟求剑。成败皆是执念，先去把碗洗了。'
  },
  {
    id: 8,
    category: '财富焦虑',
    trouble: '创业失败了，欠了一笔债，感觉人生彻底无望了。',
    style: '温情心灵抚慰',
    whisper: '千金散尽还复来。欠债是财物之亏空，而非生命之终点。青山尚在，柴火总能重燃，愿此暖意予汝力量。'
  },
  {
    id: 9,
    category: '情感纠葛',
    trouble: '处在一段消耗的感情里，明知道对方不合适却狠不下心断开。',
    style: '九巨擘禅意论',
    whisper: '乔布斯开示：人生的系统应当像产品设计一样，做极致的减法。不合适的模块不及时切除，必将导致整体崩溃。'
  },
  {
    id: 10,
    category: '虚度光阴',
    trouble: '每天浑浑噩噩，找不到人生的目标和意义，觉得活得很虚无。',
    style: '传统禅宗开示',
    whisper: '佛曰：无一物中无尽藏。意义非追寻而得，而在当下一吐一吸、一餐一茶之中，活在当下即是意义。'
  },
  {
    id: 11,
    category: '职场内卷',
    trouble: '刚刚被公司裁员了，中年危机突然降临，未来不知何去何从。',
    style: '温情心灵抚慰',
    whisper: '危机亦是转机之始。被社会系统退回，正是重新审视自己生命轨迹的良机。别急，安顿好内心，路在脚下。'
  },
  {
    id: 12,
    category: '深夜内耗',
    trouble: '特别在意别人的看法和评价，别人一句无心的话我能琢磨好几天。',
    style: '当头棒喝警醒',
    whisper: '大棒敲击：汝之尊严，难道路边一言便可践踏？把内耗的精力拿去修行，他人之口，与汝何干！'
  },
  {
    id: 13,
    category: '财富焦虑',
    trouble: '身边朋友都买车买房结婚了，只有我还是个租房的单身游民，很焦虑。',
    style: '九巨擘禅意论',
    whisper: '秦始皇开示：六国强弱各异，统一步调需按客观律法。汝之人生当如筑长城，稳扎稳打，何必与他人并驾齐驱。'
  },
  {
    id: 14,
    category: '情感纠葛',
    trouble: '父母总是逼我相亲，觉得我让他们在亲戚面前丢脸，家庭关系很紧张。',
    style: '传统禅宗开示',
    whisper: '佛曰：因缘会遇时，果报还自受。父母之急源于关切与世俗习气。以柔克刚，微笑面对，执念自消。'
  },
  {
    id: 15,
    category: '虚度光阴',
    trouble: '周末只想着躺平睡觉，家务也不想做，觉得整个人在慢慢腐烂。',
    style: '赛博木鱼解压',
    whisper: '系统提示：低功耗待机模式已满载。检测到本地系统积灰。敲击木鱼刷新内核，站起来喝杯水。'
  },
  {
    id: 16,
    category: '深夜内耗',
    trouble: '白天工作社交表现得很开朗，晚上一个人时却感到无比孤独和空虚。',
    style: '温情心灵抚慰',
    whisper: '面具戴久了，灵魂也需要呼吸。孤独是生命原本的底色，在寂静中与真实的自己对话，便不再感到空虚。'
  },
  {
    id: 17,
    category: '职场内卷',
    trouble: '工作十年了，遇到了职业瓶颈期，升职无望，跳槽又不敢，很纠结。',
    style: '九巨擘禅意论',
    whisper: '比尔盖茨开示：当系统无法直接迭代时，需进行底层重构。寻找支线任务，以渐进的方式突破瓶颈。'
  },
  {
    id: 18,
    category: '财富焦虑',
    trouble: '总觉得赚的钱不够多，对未来的养老和生病充满了恐惧，不敢消费。',
    style: '当头棒喝警醒',
    whisper: '大棒敲击：存财以备灾，灾未至而汝先忧死。恐惧是虚幻的影子，汝却将其当成当下的枷锁。可笑至极！'
  },
  {
    id: 19,
    category: '情感纠葛',
    trouble: '在友情里感觉总是自己在主动付出，对方很冷淡，不知道要不要继续。',
    style: '传统禅宗开示',
    whisper: '佛曰：来如风雨，去似微尘。善缘随喜，恶缘随避。付出求回执，便是执念，随缘而安即可。'
  },
  {
    id: 20,
    category: '虚度光阴',
    trouble: '买了好多书和网课，却从来没有看完过，总是半途而废，很自责。',
    style: '赛博木鱼解压',
    whisper: '系统警告：检测到信息囤积症。购买行为并非技能下载。请停止下载任务，点击木鱼，静心研读一页书。'
  },
  {
    id: 21,
    category: '深夜内耗',
    trouble: '想起多年前犯过的一个愚蠢错误，现在依然感到无比尴尬和后悔。',
    style: '温情心灵抚慰',
    whisper: '过去的已经随时间流逝，今日之汝已非昨日之身。宽恕那个年轻愚笨的自己，才是对当下最好的修行。'
  },
  {
    id: 22,
    category: '职场内卷',
    trouble: '老板是个画饼大师，每天被各种不切实际的 KPI 折磨，快要抑郁了。',
    style: '当头棒喝警醒',
    whisper: '大棒敲击：饼是面粉做的，老板画的饼是虚空做的。拿一份工资做一份事，何必将情绪抵押给虚空？'
  },
  {
    id: 23,
    category: '财富焦虑',
    trouble: '买股票被套牢亏了好多钱，每天看着绿色的大盘心情极度抑郁。',
    style: '赛博木鱼解压',
    whisper: '系统提示：业力指数因波动剧烈而超标。股市涨跌是系统随机扰动。点击木鱼，清空情绪缓存，平安是福。'
  },
  {
    id: 24,
    category: '情感纠葛',
    trouble: '已婚生活只剩下鸡毛蒜皮的争吵，没有了爱情，感觉像在坐牢。',
    style: '传统禅宗开示',
    whisper: '佛曰：前世五百次的回眸，换得今生擦肩而过。婚姻是同船渡，争吵是风浪。多看善处，少起怨心。'
  },
  {
    id: 25,
    category: '虚度光阴',
    trouble: '每天都在做重复的琐事，感觉自己的才华正在被慢慢消磨殆尽。',
    style: '九巨擘禅意论',
    whisper: '柏拉图开示：卓越并非一时之举，而是习惯的产物。琐事是理念世界的投影，在平凡中坚持探索真理。'
  },
  {
    id: 26,
    category: '深夜内耗',
    trouble: '身体总是处于亚健康状态，经常偏头痛失眠，对什么都提不起兴趣。',
    style: '温情心灵抚慰',
    whisper: '身心一体，痛是身体向汝发出的求救信号。放慢脚步，规律作息，用慈悲心对待这具疲惫的皮囊。'
  },
  {
    id: 27,
    category: '职场内卷',
    trouble: '想转行去当数字游民，但又害怕失去稳定收入带来的安全感，非常纠结。',
    style: '九巨擘禅意论',
    whisper: '贝索斯开示：运用逆向后悔最小化框架。十年后汝会后悔没有尝试，还是会后悔放弃了这几千元月薪？'
  },
  {
    id: 28,
    category: '财富焦虑',
    trouble: '生病住院花光了积蓄，感觉抗风险能力太弱，前途一片黑暗。',
    style: '温情心灵抚慰',
    whisper: '钱财乃身外之物，留得青山在，不怕没柴烧。身体康复便是最大的功德，一切都会慢慢好起来的。'
  },
  {
    id: 29,
    category: '情感纠葛',
    trouble: '身边的亲人渐渐老去生病，对死亡感到无比的恐惧，不知道该怎么面对。',
    style: '传统禅宗开示',
    whisper: '佛曰：生死疲劳，从贪欲起。生老病死是自然法度。珍惜当下相聚的每一刻，送别时心怀感恩，便无遗憾。'
  },
  {
    id: 30,
    category: '虚度光阴',
    trouble: '退休后无所事事，感觉被社会抛弃了，每天都觉得很无聊。',
    style: '传统禅宗开示',
    whisper: '佛曰：心远地自偏。无工作之羁绊，正是行云流水、赏花品茶、修身养性的好时节。清净无为，大自在。'
  }
];

const samples = ref<ZenSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.trouble.includes(searchQuery.value) || 
      s.style.includes(searchQuery.value) || 
      s.whisper.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
