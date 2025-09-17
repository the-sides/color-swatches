<script setup>
import { onMounted, ref, watch } from 'vue'
const delay = async () => new Promise((r) => setTimeout(() => r(), 500))
const colors = ref([])
const s = ref(100)
const l = ref(50)
const seenColors = ref([])
async function run() {
  colors.value = []
  seenColors.value = []
  for (let i = 0; i < 360; i++) {
    fetch(`https://www.thecolorapi.com/id?hsl=${i},${s.value}%,${l.value}%`)
      .then((rs) => rs.json())
      .then((data) => {
        if (seenColors.value.includes(data.name.value)) return
        seenColors.value.push(data.name.value)

        colors.value = [
          ...colors.value,
          {
            name: `${data.name.value} (${i})`,
            bg: data.hsl.value,
            data,
            i,
          },
        ]
      })
    // await delay()
  }
}

// Runs too often
// watch(s, run)
// watch(l, run)

onMounted(() => {
  run()
})
</script>
<template>
  <main class="flex flex-wrap w-screen">
    <!-- {{ colors?.[0].data }} -->
    <div
      v-for="color in colors.sort((a, b) => a.i - b.i)"
      :key="color.name"
      class="text-black"
      :style="{ 'background-color': color.bg }"
    >
      {{ color.name }}
    </div>
    <div class="fixed bottom-0 inset-x-0 mx-auto">
      <input type="range" name="sr" id="sr" v-model="s" v-on:change="run" />
      <input type="number" name="sn" id="sn" v-model="s" v-on:change="run" />
      <input type="range" name="l" id="l" v-model="l" v-on:change="run" />
      <input type="number" name="ln" id="sn" v-model="l" v-on:change="run" />
    </div>
  </main>
</template>

<style scoped>
body {
  background: #efe;
  display: flex;
  flex-wrap: wrap;
}

div {
  width: 200px;
  height: 200px;
  /*   background: blue */
}

pre {
  word-wrap: wrap;
}
</style>
