<script setup>
import { onMounted, ref } from 'vue'
const colors = ref([])
const s = ref(100)
const l = ref(50)

const seenColors = ref([])
const controllers = ref([])

const BATCH_SIZE = 30
async function run() {
  // Kill any existing requests
  controllers.value.forEach((controller) => controller.abort())

  colors.value = []
  seenColors.value = []
  for (let b = 0; b < 360 / BATCH_SIZE; b++) {
    const requests = []
    for (let i = b * BATCH_SIZE; i < (b + 1) * BATCH_SIZE; i++) {
      const controller = new AbortController()
      const req = fetch(`https://www.thecolorapi.com/id?hsl=${i},${s.value}%,${l.value}%`, {
        signal: controller.signal,
      })
        .then((rs) => rs.json())
        .then((data) => {
          if (seenColors.value.includes(data.name.value)) return
          seenColors.value.push(data.name.value)

          colors.value = [
            ...colors.value,
            {
              name: `${data.name.value} <br> (i:${i}) <br> (dist:${data.name.distance})`,
              bg: data.hsl.value,
              data,
              i,
            },
          ]
        })
      controllers.value = [...controllers.value, controller]
      requests.push(req)
    }
    await Promise.all(requests)
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
  <main class="grid grid-cols-4 w-screen relative">
    <!-- {{ colors?.[0].data }} -->
    <div
      v-for="color in colors"
      :key="color.name"
      class="text-black flex-1 min-w-[200px] h-[200px]"
      :style="{ 'background-color': color.bg }"
      v-html="color.name"
    ></div>
    <div class="fixed bottom-0 flex justify-center w-full">
      <div
        class="mx-auto w-fit px-2 py-1 drop-shadow-black drop-shadow-2xl rounded-t bg-white text-black"
      >
        <div class="flex gap-2">
          <label class="w-[5rem]" for="s">Saturation: </label>
          <input type="range" name="s" id="s" v-model="s" v-on:change="run" />
          <input class="w-12" type="number" name="sn" id="sn" v-model="s" v-on:change="run" />
        </div>
        <div class="flex gap-2">
          <label class="w-[5rem]" for="l">Lightness:</label>
          <input type="range" name="l" id="l" v-model="l" v-on:change="run" />
          <input class="w-12" type="number" name="ln" id="ln" v-model="l" v-on:change="run" />
        </div>
      </div>
    </div>
  </main>
</template>
