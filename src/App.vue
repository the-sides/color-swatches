<script setup>
import { onMounted, ref, watch } from 'vue'
const colors = ref([])
const s = ref(100)
const l = ref(50)

// 360 / 15 = 24 reqs in each batch
// 360 / 5 = 72 reqs in each batch
const GAP_BETWEEN_HUES = 5
const seenColors = ref([])
const controllers = ref([])

async function run() {
  // Kill any existing requests (if changed while requesting)
  controllers.value.forEach((controller) =>
    controller.abort('A different user input was specified'),
  )

  colors.value = []
  seenColors.value = []

  // Hue requests in each batch (with GAP of 15):
  // 0, 15, 30, 45...
  // 1, 16, 31, 46...

  for (let offset = 0; offset < GAP_BETWEEN_HUES; offset++) {
    const requests = []
    for (let batch = 0; batch < 360 / GAP_BETWEEN_HUES; batch++) {
      const h = batch * GAP_BETWEEN_HUES + offset
      const controller = new AbortController()
      const req = fetch(`https://www.thecolorapi.com/id?hsl=${h},${s.value}%,${l.value}%`, {
        signal: controller.signal,
      })
        .then((rs) => rs.json())
        .then((data) => {
          if (seenColors.value.includes(data.name.value)) return
          seenColors.value.push(data.name.value)

          colors.value = [
            ...colors.value,
            {
              name: data.name.value,
              h: data.hsl.h,
              rgb: data.rgb.value,
              bg: data.hsl.value,
            },
          ]
        })
      controllers.value = [...controllers.value, controller]
      requests.push(req)
    }
    await Promise.all(requests)
  }

  controllers.value = []
}

onMounted(() => {
  run()
})

watch([s, l], () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
})
</script>

<template>
  <main
    class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-5 3xl:grid-cols-6 w-full relative"
  >
    <div
      v-for="color in colors"
      :key="color.bg"
      class="text-black flex px-2 justify-center gap-6 flex-1 min-w-[200px] h-[200px] relative"
      :style="{
        'background-color': color.bg,
        order: color.h,
      }"
    >
      <p class="bg-slate-700 px-2 py-1 w-fit h-fit text-center text-white rounded-b">
        {{ color.name }} - {{ color.rgb }}
      </p>
    </div>
    <div class="fixed z-10 bottom-0 flex justify-center w-full">
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
