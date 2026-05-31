<script setup>
// defineProps 用来声明这个组件需要从父组件接收哪些数据。
defineProps({
  symbol: {
    type: String,
    required: true,
  },
  name: {
    type: String,
    required: true,
  },
  price: {
    type: String,
    required: true,
  },
  change: {
    type: String,
    required: true,
  },
});
</script>

<template>
  <article class="price-card">
    <div>
      <span class="price-symbol">{{ symbol }}</span>
      <p>{{ name }}</p>
    </div>

    <div class="price-meta">
      <strong>{{ price }}</strong>
      <span>{{ change }}</span>
    </div>
  </article>
</template>
