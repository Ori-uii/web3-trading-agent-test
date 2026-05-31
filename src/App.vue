<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue";

// 这些数组是页面数据。以后接真实接口时，可以先从这里替换成接口返回的数据。
const navLinks = ["Products", "Resources", "Ecosystem", "About"];

const partners = ["YouTube", "bilibili", "Douyin", "Zhihu", "Xiaohongshu"];

// 为了让合作伙伴滚动时无缝衔接，这里把同一组名字复制一遍。
const scrollingPartners = [...partners, ...partners];

const scrollY = ref(0);
const viewportWidth = ref(window.innerWidth);
const viewportHeight = ref(window.innerHeight);

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

onMounted(() => {
  updateViewport();
  window.addEventListener("scroll", updateViewport, { passive: true });
  window.addEventListener("resize", updateViewport);
});

onUnmounted(() => {
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

    <span class="media-credit">Background video: pending</span>
  </div>
</template>
