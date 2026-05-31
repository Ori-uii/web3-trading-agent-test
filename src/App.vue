<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue";

// 这些数组是页面数据。以后接真实接口时，可以先从这里替换成接口返回的数据。
const navLinks = ["Products", "Resources", "Ecosystem", "About"];

const partners = ["YouTube", "bilibili", "Douyin", "Zhihu", "Xiaohongshu"];

const latestUpdates = [
  {
    category: "Product Update",
    date: "Jun 01, 2026",
    title: "NeuroTrade Adds AI-Native Market Execution",
    description:
      "The homepage now highlights an agent workflow for building, monitoring, and automating Web3 financial trading decisions.",
    cta: "Read More",
    visual: "agent",
  },
  {
    category: "Design System",
    date: "May 31, 2026",
    title: "Dark Tech Landing Page Refined for Scroll Stories",
    description:
      "Navigation, hero messaging, partner marquee, and the second-page open economy section now work together as a polished Web3 homepage.",
    cta: "View Update",
    visual: "design",
  },
  {
    category: "Infrastructure",
    date: "May 31, 2026",
    title: "Vue 3 and Vite Foundation Ready for Future Trading Tools",
    description:
      "The project has been rebuilt as a Vue 3 application with reusable data-driven sections, ready for videos, APIs, and new dashboard modules.",
    cta: "Explore",
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

// 第三页始终按「上一条、当前条、下一条」排列，所以收起的卡片会出现在大卡片两侧。
const orderedLatestUpdates = computed(() =>
  [-1, 0, 1].map((offset) => {
    const index =
      (activeUpdateIndex.value + offset + latestUpdates.length) %
      latestUpdates.length;

    return {
      ...latestUpdates[index],
      index,
      position: offset,
      isActive: index === activeUpdateIndex.value,
    };
  }),
);

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

const showRailCursor = (event, position) => {
  moveRailCursor(event);
  railCursorDirection.value = position < 0 ? "left" : "right";
  railCursorVisible.value = true;
};

const hideRailCursor = () => {
  railCursorVisible.value = false;
};

onMounted(() => {
  updateViewport();
  startLatestCarousel();
  window.addEventListener("scroll", updateViewport, { passive: true });
  window.addEventListener("resize", updateViewport);
});

onUnmounted(() => {
  window.clearInterval(latestIntervalId);
  window.removeEventListener("scroll", updateViewport);
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
        <a
          v-for="link in navLinks"
          :key="link"
          :href="`#${link.toLowerCase()}`"
        >
          {{ link }}
        </a>
      </nav>

      <a class="launch-button" href="#launch">Launch App</a>
    </header>

    <section class="welcome-section" id="learn">
      <div class="welcome-layout">
        <h2>Welcome to the</h2>
        <div class="video-frame-shadow" aria-hidden="true"></div>
        <h2>Open Economy</h2>
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
          v-for="update in orderedLatestUpdates"
          :key="update.title"
          class="latest-card"
          :class="{ 'latest-card--active': update.isActive }"
          :role="update.isActive ? undefined : 'button'"
          :tabindex="update.isActive ? undefined : 0"
          :aria-label="
            update.isActive ? update.title : `展开更新：${update.title}`
          "
          @click="!update.isActive && showLatestUpdate(update.index)"
          @keydown.enter.prevent="
            !update.isActive && showLatestUpdate(update.index)
          "
          @keydown.space.prevent="
            !update.isActive && showLatestUpdate(update.index)
          "
          @pointerenter="
            !update.isActive && showRailCursor($event, update.position)
          "
          @pointermove="!update.isActive && moveRailCursor($event)"
          @pointerleave="hideRailCursor"
        >
          <div class="latest-rail-label">
            <span>{{ update.category }}</span>
          </div>

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
            v-if="update.isActive"
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
