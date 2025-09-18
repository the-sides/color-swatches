<script setup>
import { computed, onMounted, ref } from 'vue'
const colors = ref([])
const colorSlots = ref([])
const s = ref(100)
const l = ref(50)

const BATCH_SIZE = 360
const seenColors = ref([])
const controllers = ref([])

async function run() {
  // Kill any existing requests
  controllers.value.forEach((controller) =>
    controller.abort('A different user input was specified'),
  )

  colors.value = []
  seenColors.value = []
  for (let b = 0; b < 360 / BATCH_SIZE; b++) {
    const requests = []
    for (let i = b * BATCH_SIZE; i < (b + 1) * BATCH_SIZE && i <= 360; i++) {
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
              name: `${data.name.value}`, // <br> (i:${i}) <br> (dist:${data.name.distance})<br> (hsl:${data.hsl.value})`,
              h: data.hsl.h,
              rgb: data.rgb.value,
              bg: data.hsl.value,
              data,
              i,
            },
          ]
        })
      controllers.value = [...controllers.value, controller]
      colorSlots.value = [...colorSlots.value, true]
      requests.push(req)
    }
    await Promise.all(requests)
  }
  colorSlots.value = colors.value.map((c) => c.bg)
  controllers.value = []
}

onMounted(() => {
  run()
})

const optimisticSpacing = computed(() => {
  if (s.value < 5 || 95 < s.value) {
    return 100
  }
  if (s.value < 15 || 85 < s.value) {
    return 50
  }
  return 80
})
</script>

<template>
  <main class="grid grid-cols-4 w-screen relative [&_*]:text-white">
    <!-- {{ colors?.[0].data }} -->
    <div
      v-for="(prevColor, i) in colorSlots"
      :key="i"
      class="text-black flex px-2 justify-center gap-6 flex-1 min-w-[200px] h-[200px] relative"
      :style="{
        // How do I make this update when s and l are updated?
        'background-color': colors[i]?.bg ?? prevColor, // `hsl(${i * optimisticSpacing}, ${s}%, ${l}%)`,
        order: colors[i]?.h ?? 999,
      }"
    >
      <p class="bg-slate-700 px-2 py-1 w-fit h-fit text-center rounded-b" v-if="colors[i]">
        {{ colors[i]?.name }} - {{ colors[i]?.rgb }}
      </p>
      <p
        class="bg-slate-700 mto justify-self-end px-2 w-fit mb-auto h-fit text-center rounded-b"
        v-if="colors[i]"
      ></p>
      <div
        v-else
        class="justify-self-center self-center m-auto h-24 w-24 animate-spin rounded-full border-2 border-gray-300 border-t-transparent align-[-0.125em]"
      ></div>
    </div>
    <!-- <div
      v-for="(color, i) in colors"
      :key="i + 'c'"
      class="text-black flex-1 min-w-[200px] h-[200px]"
      :style="{ 'background-color': color.bg ?? 'navy' }"
      v-html="color?.name ?? i"
    ></div> -->
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
