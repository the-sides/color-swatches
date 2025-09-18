<script setup>
import { onMounted, ref, watch } from 'vue'
const colors = ref([])
const s = ref(100)
const l = ref(50)

const DIST = 15
const seenColors = ref([])
const controllers = ref([])

async function run() {
  // Kill any existing requests
  controllers.value.forEach((controller) =>
    controller.abort('A different user input was specified'),
  )

  colors.value = []
  seenColors.value = []
  // 0, 15, 30, 45...
  // 1, 16, 31, 46...

  // 0, 1, 2
  for (let offset = 0; offset < DIST; offset++) {
    const requests = []
    for (let b = 0; b < 360 / DIST; b++) {
      let h = b * DIST + offset
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
              name: `${data.name.value}`, // <br> (i:${i}) <br> (dist:${data.name.distance})<br> (hsl:${data.hsl.value})`,
              h: data.hsl.h,
              rgb: data.hsl.value,
              bg: data.hsl.value,
              batch: offset,
              data,
            },
          ]
        })
      controllers.value = [...controllers.value, controller]
      requests.push(req)
    }
    await Promise.all(requests)
  }
  const score = colors.value.reduce((accu, c) => c.batch + accu, 0)
  window.alert(score)
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
    class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-5 3xl:grid-cols-6 w-screen relative [&_*]:text-white"
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
      <p class="bg-slate-700 px-2 py-1 w-fit h-fit text-center rounded-b">
        {{ color.name }} - {{ color.batch }} - {{ color.rgb }}
      </p>
    </div>
    <div class="fixed z-10 bottom-0 flex justify-center w-full">
      <div
        class="mx-auto w-fit px-2 py-1 drop-shadow-black drop-shadow-2xl rounded-t bg-white [&_*]:!text-black"
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
