<script setup>
import { onMounted, ref } from 'vue'
const delay = async () => new Promise((r) => setTimeout(() => r(), 500))
const colors = ref([])
async function run() {
  let lastName = ''
  for (let i = 0; i < 360; i++) {
    await fetch(`https://www.thecolorapi.com/id?hsl=${i},100%,50%`)
      .then((rs) => rs.json())
      .then((data) => {
        if (data.name.value === lastName) return
        lastName = data.name.value
        colors.value = [
          ...colors.value,
          {
            name: data.name.value,
            bg: data.hsl.value,
          },
        ]
        // const newDiv = document.createElement('div')
        // newDiv.innerText = data.name.value
        // newDiv.style.background = data.hsl.value
        // document.body.append(newDiv)
      })
    // await delay()
  }
}

onMounted(() => {
  run()
})
</script>
<template>
  <main>
    <div v-for="color in colors" :key="color.name" :style="{ 'background-color': color.bg }">
      {{ color.name }}
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
