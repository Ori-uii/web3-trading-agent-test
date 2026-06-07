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

const productSections = [
  {
    title: "Assets",
    items: [
      {
        name: "USDY",
        tag: "Not Available in US",
        description: "Freely transferable yieldcoin designed for DeFi.",
        icon: "yield",
      },
      {
        name: "OUSG",
        tag: "For Institutions & HNWIs",
        description: "Short-Term US Treasuries & Agency Bonds.",
        icon: "treasury",
      },
    ],
  },
  {
    title: "Platforms & Protocols",
    items: [
      {
        name: "Ondo Global Markets",
        tag: "Not Available in US",
        description: "Access tokenized stocks and ETFs.",
        icon: "market",
      },
      {
        name: "Flux",
        description: "Borrow and lend with tokenized assets.",
        icon: "flux",
      },
      {
        name: "Nexus",
        tag: "For Asset Issuers",
        description: "Offer instant 24/7 liquidity to your investors.",
        icon: "nexus",
      },
    ],
  },
];

const infrastructureProducts = [
  {
    name: "Neuro Chain",
    description: "The L1 blockchain built for institutional-grade RWAs.",
    icon: "chain",
  },
  {
    name: "Bridge",
    description: "Transfer Web3 financial assets across supported chains.",
    icon: "bridge",
  },
  {
    name: "Converter",
    description: "Convert between accumulating and rebasing tokens.",
    icon: "converter",
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
const activeProductKey = ref("USDY");
const productMenuOpen = ref(false);
const resourceMenuOpen = ref(false);
const railCursorVisible = ref(false);
const railCursorX = ref(0);
const railCursorY = ref(0);
const railCursorDirection = ref("right");

let latestIntervalId;
let productCloseTimeoutId;
let resourceCloseTimeoutId;
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

const mix = (start, end, progress) => start + (end - start) * progress;

const heroVideoStyle = computed(() => {
  const progress = scrollProgress.value;
  const isMobile = viewportWidth.value <= 800;
  const hasReachedWelcome = progress >= 1;
  const targetWidth = isMobile
    ? Math.min(viewportWidth.value * 0.84, 560)
    : Math.max(Math.min(viewportWidth.value * 0.38, 620), 280);
  const targetHeight = targetWidth / 1.62;

  return {
    position: hasReachedWelcome ? "absolute" : "fixed",
    top: hasReachedWelcome ? `${viewportHeight.value * 1.5}px` : "50%",
    left: "50%",
    width: `${mix(viewportWidth.value, targetWidth, progress)}px`,
    height: `${mix(viewportHeight.value, targetHeight, progress)}px`,
    borderRadius: `${mix(0, 28, progress)}px`,
    opacity: "1",
    transform: "translate(-50%, -50%)",
  };
});

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

const setActiveProduct = (name) => {
  activeProductKey.value = name;
};

const openProductMenu = () => {
  window.clearTimeout(productCloseTimeoutId);
  window.clearTimeout(resourceCloseTimeoutId);
  resourceMenuOpen.value = false;
  productMenuOpen.value = true;
};

const closeProductMenuLater = () => {
  window.clearTimeout(productCloseTimeoutId);
  productCloseTimeoutId = window.setTimeout(() => {
    productMenuOpen.value = false;
  }, 60);
};

const openResourceMenu = () => {
  window.clearTimeout(productCloseTimeoutId);
  window.clearTimeout(resourceCloseTimeoutId);
  productMenuOpen.value = false;
  resourceMenuOpen.value = true;
};

const closeResourceMenuLater = () => {
  window.clearTimeout(resourceCloseTimeoutId);
  resourceCloseTimeoutId = window.setTimeout(() => {
    resourceMenuOpen.value = false;
  }, 60);
};

const handleNavEnter = (link) => {
  if (link === "Products") {
    openProductMenu();
  }

  if (link === "Resources") {
    openResourceMenu();
  }
};

const handleNavLeave = (link) => {
  if (link === "Products") {
    closeProductMenuLater();
  }

  if (link === "Resources") {
    closeResourceMenuLater();
  }
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
  window.clearTimeout(productCloseTimeoutId);
  window.clearTimeout(resourceCloseTimeoutId);
  window.clearTimeout(wheelSnapTimeoutId);
  window.removeEventListener("scroll", updateViewport);
  window.removeEventListener("wheel", snapToWelcomeOnWheel);
  window.removeEventListener("resize", updateViewport);
});
</script>

<template>
  <div class="app-shell" :class="{ 'is-scrolled': scrollProgress > 0.55 }">
    <video class="hero-video" autoplay muted loop playsinline :style="heroVideoStyle">
      <source src="/hero-video.mp4" type="video/mp4" />
    </video>

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
          :class="{
            'nav-item--products': link === 'Products',
            'is-product-open': link === 'Products' && productMenuOpen,
            'nav-item--resources': link === 'Resources',
            'is-resource-open': link === 'Resources' && resourceMenuOpen,
          }"
          @pointerenter="handleNavEnter(link)"
          @pointerleave="handleNavLeave(link)"
        >
          <a
            :href="
              link === 'Products'
                ? '#products-menu'
                : link === 'Resources'
                  ? '#resources-menu'
                  : `#${link.toLowerCase()}`
            "
            @click="
              (link === 'Products' || link === 'Resources') &&
                ($event.preventDefault(), $event.currentTarget.blur())
            "
          >
            {{ link }}
          </a>

          <div
            v-if="link === 'Products'"
            class="product-popover"
            :class="{ 'product-popover--open': productMenuOpen }"
            aria-label="Products preview"
            @pointerenter="openProductMenu"
            @pointerleave="closeProductMenuLater"
          >
            <div class="product-columns">
              <div class="product-left">
                <section
                  v-for="section in productSections"
                  :key="section.title"
                  class="product-section"
                >
                  <h3>{{ section.title }}</h3>

                  <a
                    v-for="item in section.items"
                    :key="item.name"
                    class="product-option"
                    :class="{ 'product-option--active': item.name === activeProductKey }"
                    href="#products-menu"
                    @pointerenter="setActiveProduct(item.name)"
                    @click.prevent
                  >
                    <span
                      class="product-icon"
                      :class="`product-icon--${item.icon}`"
                      aria-hidden="true"
                    ></span>

                    <span class="product-copy">
                      <strong>
                        {{ item.name }}
                        <span v-if="item.tag">{{ item.tag }}</span>
                      </strong>
                      <span>{{ item.description }}</span>
                    </span>
                  </a>
                </section>
              </div>

              <div class="product-right">
                <div class="product-hero-visual" aria-hidden="true">
                  <span></span>
                  <span></span>
                  <span></span>
                </div>

                <section class="product-section product-section--infrastructure">
                  <h3>Infrastructure</h3>

                  <a
                    v-for="item in infrastructureProducts"
                    :key="item.name"
                    class="product-option"
                    :class="{ 'product-option--active': item.name === activeProductKey }"
                    href="#products-menu"
                    @pointerenter="setActiveProduct(item.name)"
                    @click.prevent
                  >
                    <span
                      class="product-icon"
                      :class="`product-icon--${item.icon}`"
                      aria-hidden="true"
                    ></span>

                    <span class="product-copy">
                      <strong>{{ item.name }}</strong>
                      <span>{{ item.description }}</span>
                    </span>
                  </a>
                </section>
              </div>
            </div>
          </div>

          <div
            v-if="link === 'Resources'"
            class="resource-popover"
            :class="{ 'resource-popover--open': resourceMenuOpen }"
            aria-label="Resources preview"
            @pointerenter="openResourceMenu"
            @pointerleave="closeResourceMenuLater"
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

    <span class="media-credit">Background video: local footage</span>
  </div>
</template>
