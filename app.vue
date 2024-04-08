<template>
  <div class="color-selector">
    <select v-model="$colorMode.preference">
      <option value="system">System</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
      <option value="sepia">Sepia</option>
    </select>
  </div>
  <NuxtPage class="page" />
</template>

<style>
.page {
  max-width: 1000px;
  margin: 0 auto;
}

body {
  background-color: #fff;
  color: rgba(0, 0, 0, 0.8);
}

.dark-mode body {
  background-color: #091a28;
  color: #ebf4f1;
}

.sepia-mode body {
  background-color: #f1e7d0;
  color: #433422;
}

.color-selector {
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 1000;
}

.color-selector select {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  color: #333;
  background-color: #fff;
  transition: all 0.3s ease;
}

.color-selector select:hover {
  border-color: #888;
}

.color-selector select:focus {
  outline: none;
  border-color: #007BFF;
}
</style>
