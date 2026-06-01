<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue";

// 这些数组是页面数据。以后接真实接口时，可以先从这里替换成接口返回的数据。
const navLinks = ["Products", "Resources", "Ecosystem", "About"];

const resourcePreviewItems = [
  {
    title: "Insights",
    description: "Market research and key learnings.",
  },
  {
    title: "Blog",
    description: "Product updates and highlights.",
  },
];

const partners = ["YouTube", "bilibili", "Douyin", "Zhihu", "Xiaohongshu"];

const latestUpdates = [
  {
    category: "产品更新",
    date: "2026 年 6 月 1 日",
    title: "NeuroTrade 加入 AI 原生市场执行能力",
    description:
      "新的交易 Agent 工作流可以帮助用户构建、监控并自动化 Web3 金融交易策略，让市场观察、风险提示和执行动作形成更清晰的闭环。",
    cta: "查看详情",
    visual: "agent",
  },
  {
    category: "界面升级",
    date: "2026 年 5 月 31 日",
    title: "深色科技主页完成滚动叙事升级",
    description:
      "首页导航、大标题、合作伙伴滚动栏和开放经济介绍区已经重新整理，整体视觉更接近现代 Web3 金融基础设施项目主页。",
    cta: "查看更新",
    visual: "design",
  },
  {
    category: "工程进展",
    date: "2026 年 5 月 31 日",
    title: "Vue 3 与 Vite 基础架构已准备就绪",
    description:
      "项目已经重构为 Vue 3 + Vite 应用，后续可以继续接入本地视频、实时行情接口、策略面板和交易仪表盘模块。",
    cta: "了解更多",
    visual: "infra",
  },
];

// 为了让合作伙伴滚动时无缝衔接，这里把同一组名字复制一遍。
const scrollingPartners = [...partners, ...partners];

const scrollY = ref(0);
const viewportWidth = ref(window.innerWidth);
const viewportHeight = ref(window.innerHeight);
const activeUpdateIndex = ref(0);
const railCursorVisible = ref(false);
const railCursorX = ref(0);
const railCursorY = ref(0);
const railCursorDirection = ref("right");

let latestIntervalId;
let wheelSnapTimeoutId;
let isWheelSnapping = false;

const updateViewport = () => {
  scrollY.value = window.scrollY;
  viewportWidth.value = window.innerWidth;
  viewportHeight.value = window.innerHeight;
};

const scrollProgress = computed(() => {
  const distance = viewportHeight.value * 0.92;

  return Math.min(scrollY.value / distance, 1);
});

const shadeStyle = computed(() => ({
  opacity: String(1 - scrollProgress.value * 0.9),
}));

const heroContentStyle = computed(() => ({
  opacity: String(1 - scrollProgress.value * 1.25),
  transform: `translateY(${-48 * scrollProgress.value}px)`,
}));

const nextLatestUpdate = () => {
  activeUpdateIndex.value = (activeUpdateIndex.value + 1) % latestUpdates.length;
};

const startLatestCarousel = () => {
  window.clearInterval(latestIntervalId);
  latestIntervalId = window.setInterval(nextLatestUpdate, 5000);
};

const showLatestUpdate = (index) => {
  activeUpdateIndex.value = index;
  startLatestCarousel();
};

const moveRailCursor = (event) => {
  railCursorX.value = event.clientX;
  railCursorY.value = event.clientY;
};

const showRailCursor = (event, index) => {
  moveRailCursor(event);
  railCursorDirection.value = index < activeUpdateIndex.value ? "left" : "right";
  railCursorVisible.value = true;
};

const hideRailCursor = () => {
  railCursorVisible.value = false;
};

const releaseWheelSnap = () => {
  window.clearTimeout(wheelSnapTimeoutId);
  wheelSnapTimeoutId = window.setTimeout(() => {
    isWheelSnapping = false;
  }, 720);
};

const snapToWelcomeOnWheel = (event) => {
  if (event.deltaY <= 0 || isWheelSnapping) {
    return;
  }

  const welcomeSection = document.getElementById("learn");

  if (!welcomeSection) {
    return;
  }

  const welcomeTop = welcomeSection.offsetTop;

  if (window.scrollY >= welcomeTop - 2) {
    return;
  }

  event.preventDefault();
  isWheelSnapping = true;
  window.scrollTo({
    top: welcomeTop,
    behavior: "smooth",
  });
  releaseWheelSnap();
};

onMounted(() => {
  updateViewport();
  startLatestCarousel();
  window.addEventListener("scroll", updateViewport, { passive: true });
  window.addEventListener("wheel", snapToWelcomeOnWheel, { passive: false });
  window.addEventListener("resize", updateViewport);
});

onUnmounted(() => {
  window.clearInterval(latestIntervalId);
  window.clearTimeout(wheelSnapTimeoutId);
  window.removeEventListener("scroll", updateViewport);
  window.removeEventListener("wheel", snapToWelcomeOnWheel);
  window.removeEventListener("resize", updateViewport);
});
</script>

<template>
  <div class="app-shell" :class="{ 'is-scrolled': scrollProgress > 0.55 }">
    <div class="hero-shade" :style="shadeStyle"></div>

    <section class="hero">
      <div class="announcement">
        <span>Autonomous Web3 Trading Infrastructure</span>
        <a href="#learn">Learn More</a>
      </div>

      <main class="hero-content" :style="heroContentStyle">
        <p class="eyebrow">AI-native market execution</p>
        <h1>Web3 Financial Trading Agent</h1>
      </main>

      <section class="partner-strip" aria-label="合作伙伴">
        <div class="partner-track">
          <span
            v-for="(partner, index) in scrollingPartners"
            :key="`${partner}-${index}`"
          >
            {{ partner }}
          </span>
        </div>
      </section>
    </section>

    <header class="site-header" aria-label="Primary navigation">
      <a class="brand" href="#" aria-label="Web3 Financial Trading Agent home">
        <span class="brand-mark" aria-hidden="true">
          <span></span>
        </span>
        <span>NeuroTrade</span>
      </a>

      <nav class="nav-links" aria-label="Main menu">
        <div
          v-for="link in navLinks"
          :key="link"
          class="nav-item"
          :class="{ 'nav-item--resources': link === 'Resources' }"
        >
          <a
            :href="link === 'Resources' ? '#resources-menu' : `#${link.toLowerCase()}`"
            @click="
              link === 'Resources' &&
                ($event.preventDefault(), $event.currentTarget.blur())
            "
          >
            {{ link }}
          </a>

          <div
            v-if="link === 'Resources'"
            class="resource-popover"
            aria-label="Resources preview"
          >
            <div class="resource-list">
              <a
                v-for="item in resourcePreviewItems"
                :key="item.title"
                class="resource-option"
                href="#resources-menu"
                @click.prevent
              >
                <span class="resource-icon" aria-hidden="true">
                  <span></span>
                  <span></span>
                  <span></span>
                  <span></span>
                </span>

                <span class="resource-text">
                  <strong>{{ item.title }}</strong>
                  <span>{{ item.description }}</span>
                </span>
              </a>
            </div>

            <div class="resource-preview" aria-hidden="true">
              <div class="resource-preview-stack">
                <span></span>
                <span></span>
                <span></span>
              </div>
            </div>
          </div>
        </div>
      </nav>

      <a class="launch-button" href="#launch">Launch App</a>
    </header>

    <section class="welcome-section" id="learn">
      <div class="welcome-layout">
        <h2>欢迎来到</h2>
        <div class="video-frame-shadow" aria-hidden="true"></div>
        <h2>Web3 世界</h2>
      </div>

      <p>
        Build, monitor, and automate Web3 financial trading workflows with an
        AI-native agent designed for modern digital markets.
      </p>
    </section>

    <section class="latest-section" id="resources">
      <h2>See the Latest <span>from NeuroTrade</span></h2>

      <div class="latest-carousel" aria-label="最近更新轮播">
        <article
          v-for="(update, index) in latestUpdates"
          :key="update.title"
          class="latest-card"
          :class="{ 'latest-card--active': index === activeUpdateIndex }"
          :role="index === activeUpdateIndex ? undefined : 'button'"
          :tabindex="index === activeUpdateIndex ? undefined : 0"
          :aria-label="
            index === activeUpdateIndex ? update.title : `展开更新：${update.title}`
          "
          @click="index !== activeUpdateIndex && showLatestUpdate(index)"
          @keydown.enter.prevent="
            index !== activeUpdateIndex && showLatestUpdate(index)
          "
          @keydown.space.prevent="
            index !== activeUpdateIndex && showLatestUpdate(index)
          "
          @pointerenter="
            index !== activeUpdateIndex && showRailCursor($event, index)
          "
          @pointermove="index !== activeUpdateIndex && moveRailCursor($event)"
          @pointerleave="hideRailCursor"
        >
          <div class="latest-card-content">
            <div class="latest-visual" :class="`latest-visual--${update.visual}`">
            <div class="visual-grid" aria-hidden="true">
              <span></span>
              <span></span>
              <span></span>
              <span></span>
              <span></span>
              <span></span>
            </div>

            <div class="visual-badge">
              <span class="brand-mark" aria-hidden="true">
                <span></span>
              </span>
              <strong>NeuroTrade</strong>
            </div>
          </div>

          <div class="latest-copy">
            <div class="latest-meta">
                {{ update.category }} · {{ update.date }}
            </div>

              <h3>{{ update.title }}</h3>
              <p>{{ update.description }}</p>
              <a href="#resources">{{ update.cta }}</a>
            </div>
          </div>

          <div
            v-if="index === activeUpdateIndex"
            :key="activeUpdateIndex"
            class="latest-timer"
            aria-label="5 秒自动切换计时"
          ></div>
        </article>
      </div>
    </section>

    <div
      v-show="railCursorVisible"
      class="rail-cursor"
      :class="`rail-cursor--${railCursorDirection}`"
      :style="{ left: `${railCursorX}px`, top: `${railCursorY}px` }"
      aria-hidden="true"
    >
      <span class="rail-cursor-horizontal" aria-hidden="true">
        {{ railCursorDirection === "left" ? "‹" : "›" }}
      </span>
      <span class="rail-cursor-vertical" aria-hidden="true">
        {{ railCursorDirection === "left" ? "⌃" : "⌄" }}
      </span>
    </div>

    <span class="media-credit">Background video: pending</span>
  </div>
</template>
