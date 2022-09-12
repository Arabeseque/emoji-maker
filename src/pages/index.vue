<script setup lang="ts">
type ImportModuleFunc = () => Promise<any>

interface imageType {
  head: string[]
  eyes: string[]
  eyesBrows: string[]
  mouse: string[]
}

const image = ref<imageType>({
  head: [],
  eyes: [],
  eyesBrows: [],
  mouse: [],
})

const tabs = ['head', 'mouse', 'eyes', 'eyesBrows']
const resolveImportGlobModule = async (modules: Record<string, ImportModuleFunc>) => {
  const imports = Object.values(modules).map(importFuc => importFuc())
  const modulesList = await Promise.all(imports)
  return modulesList.map(module => module.default)
}

const loadImage = async () => {
  const headModules = import.meta.glob('./assets/Body/*.svg')
  const loadedHeadModules = await resolveImportGlobModule(headModules)
  const eyesModules = import.meta.glob('./assets/Eyes/*.svg')
  const loadedEyesModules = await resolveImportGlobModule(eyesModules)
  const EyesBrowsModules = import.meta.glob('./assets/EyesBrows/*.svg')
  const loadedEyesBrowsModules = await resolveImportGlobModule(EyesBrowsModules)
  const mouthModules = import.meta.glob('./assets/Mouth/*.svg')
  const loadedMuuthModules = await resolveImportGlobModule(mouthModules)

  image.value = {
    head: [...loadedHeadModules],
    eyes: [...loadedEyesModules],
    eyesBrows: [...loadedEyesBrowsModules],
    mouse: [...loadedMuuthModules],
  }

  randomIndex()
}

const selectedTab = ref('head')
function handleSelectTab(idx: any) {
  selectedTab.value = tabs[idx]
}

interface selectedIndexType {
  [head: string]: number
  eyes: number
  eyesBrows: number
  mouse: number
}

const selectedIndex = ref<selectedIndexType>({
  head: 0,
  eyes: 0,
  eyesBrows: 0,
  mouse: 0,
})

const selectedImage = computed(() => {
  return {
    head: image.value.head[selectedIndex.value.head],
    eyes: image.value.eyes[selectedIndex.value.eyes],
    eyesBrows: image.value.eyesBrows[selectedIndex.value.eyesBrows],
    mouse: image.value.mouse[selectedIndex.value.mouse],
  }
})

function handleSelectItem(tab: string, index: number) {
  selectedIndex.value[tab] = index
}

const pathToImage = async (path: string) => {
  return new Promise<HTMLImageElement | null>((resolve, reject) => {
    if (path === 'null')
      resolve(null)

    const img = new Image()
    img.src = path
    img.onload = () => {
      resolve(img)
    }
    img.onerror = reject
  })
}
const refCanvas = ref<HTMLCanvasElement>()
const imageSize = 160
watchSyncEffect(() => {
  const [head, eyes, eyesBrows, mouse] = Object.values(selectedImage.value)
  Promise.all([pathToImage(head), pathToImage(eyes), pathToImage(eyesBrows), pathToImage(mouse)]).then(([head, eyes, eyesBrows, mouse]) => {
    const ctx = refCanvas.value!.getContext('2d')
    if (ctx) {
      ctx.clearRect(0, 0, imageSize, imageSize)
      if (head)
        ctx.drawImage(head, 0, 0, imageSize, imageSize)

      if (eyes)
        ctx.drawImage(eyes, 0, 0, imageSize, imageSize)

      if (eyesBrows)
        ctx.drawImage(eyesBrows, 0, 0, imageSize, imageSize)

      if (mouse)
        ctx.drawImage(mouse, 0, 0, imageSize, imageSize)
    }
  })
})

function handleImageDownload() {
  const canvas = refCanvas.value
  if (canvas) {
    const link = document.createElement('a')
    link.download = 'image.png'
    link.href = canvas.toDataURL()
    link.click()
  }
}

function randomIndex() {
  selectedIndex.value.head = Math.floor(Math.random() * image.value.head.length)
  selectedIndex.value.eyes = Math.floor(Math.random() * image.value.eyes.length)
  selectedIndex.value.eyesBrows = Math.floor(Math.random() * image.value.eyesBrows.length)
  selectedIndex.value.mouse = Math.floor(Math.random() * image.value.mouse.length)
}
onMounted(() => {
  loadImage()
})
</script>

<template>
  <div px-6 pt-3>
    <div flex justify-between>
      <div text-2xl font-bold>
        Fluent Emoji Maker
      </div>
      <button icon-btn @click="toggleDark()">
        <div text-lg dark:i-carbon-moon i-carbon-sun />
      </button>
    </div>

    <div bg-white rounded-md mx-4 mt-5 shadow-lg>
      <!-- canvas -->
      <div flex items-center justify-center p-t-8 transition-all>
        <canvas ref="refCanvas" :height="imageSize" :width="imageSize" border rounded-lg transition-all />
      </div>

      <div flex items-center justify-center>
        <button
          border rounded-full mt-5 px-2 py-1 hover:bg-gray-100
          font-bold text-lg text-center
          @click="handleImageDownload"
        >
          export
        </button>
        <div

          hover:bg-gray-100
          border rounded-full p-2 flex items-center justify-center ml-2 mt-5
        >
          <div
            class="cycle"
            relative
            i-material-symbols-cycle
            cursor-pointer
            @click="randomIndex"
          />
        </div>
      </div>

      <!-- tabs -->
      <div flex gap-2 items-center justify-center py-6>
        <div v-for="(tab, idx) in tabs" :key="tab" @click="handleSelectTab(idx)">
          <img
            :src="selectedImage[tab]"
            h-16 w-16
            cursor-pointer border rounded-md hover:bg-gray-100 transition-color shadow
          >
        </div>
      </div>

      <!-- 分割线 -->
      <div border-b mx-8 />

      <!-- 展示 -->
      <div
        v-for="item in Object.keys(image)"
        :key="item"
        flex justify-center
      >
        <div
          v-show="item === selectedTab"
          flex flex-wrap p-5 justify-center gap-2
        >
          <div
            v-for="pic in image[item]"
            :key="pic"
            @click="handleSelectItem(item, image[item].indexOf(pic))"
          >
            <img
              :src="pic"
              h-16
              cursor-pointer border rounded-md hover:bg-gray-100 transition-color shadow
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
  :root {
    background-color: #ffe4e6;
  }

  .cycle {
    top: 2px;
  }
</style>
