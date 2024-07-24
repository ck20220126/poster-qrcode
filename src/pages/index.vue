<script setup lang="ts" generic="T extends any, O extends any">
import html2canvas from 'html2canvas'
import * as changedpi from 'changedpi'
import download from 'downloadjs'
import { type FormInst, NButton, NDivider, NDrawer, NDrawerContent, NFloatButton, NForm, NFormItem, NIcon, NInputNumber, NProgress, NSpace, NUpload, type UploadFileInfo } from 'naive-ui'
import { CogSharp } from '@vicons/ionicons5'

defineOptions({
  name: 'IndexPage',
})

const posterBg = ref()
const posterFileList = ref<Array<UploadFileInfo>>([])

function handleChange(options: { fileList: UploadFileInfo[] }) {
  const file = options.fileList[0]
  const reader = new FileReader()

  if (file?.file) {
    reader.onload = function (e) {
      posterBg.value = e.target?.result
    }

    reader.readAsDataURL(file.file)
  }
}

const qrcodeImage = ref()
const qrcodeFileList = ref<Array<UploadFileInfo>>([])
const showQrcodeFileList = ref(false)

const currentTaskIndex = ref(0)
const percentage = computed(() => {
  return Number.parseInt((((currentTaskIndex.value + 1) / qrcodeFileList.value.length) * 100).toString())
})
function handleCodeChange({ fileList }: { fileList: UploadFileInfo[] }) {
  qrcodeFileList.value = fileList
  const file = fileList[0]
  const reader = new FileReader()

  if (file?.file) {
    reader.onload = function (e) {
      qrcodeImage.value = e.target?.result
    }

    reader.readAsDataURL(file.file)
  }
}

const timer = ref<ReturnType<typeof setTimeout>>()
async function renderQrcode(file: File) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = function (e) {
      qrcodeImage.value = e.target?.result
      timer.value = setTimeout(() => {
        html2canvas(document.querySelector('#capture')!).then(async (canvas: any) => {
          const fileName = file.name.split('.')
          fileName.pop()
          await downloadCanvas(canvas, fileName.join('.'))
          resolve(true)
        })
      }, 300)
    }
    reader.onerror = reject
    reader.readAsDataURL(file)
  })
}

const showConfigDrawer = ref(true)
const poster = ref<any>()

const generation = ref(false)
async function submit() {
  if (!qrcodeFileList.value?.length)
    return

  currentTaskIndex.value = 0
  generation.value = true
  for (let i = 0; i < qrcodeFileList.value.length; i++) {
    if (qrcodeFileList.value[i].file) {
      await renderQrcode(qrcodeFileList.value[i].file!)
      currentTaskIndex.value++
    }
  }

  generation.value = false
}

async function downloadCanvas(canvas: any, fileName?: string) {
  let imgData = canvas.toDataURL({ format: 'png', quality: 1 })
  imgData = changedpi.changeDpiDataUrl(imgData, 300)
  await download(imgData, fileName, 'image/png')
}

const formData = ref({
  size: 136,
  x: 44,
  y: 1132,
})
const formRef = ref<FormInst | null>(null)
function clearForm() {
  // formRef.value?.restoreValidation()
  posterFileList.value = []
  posterBg.value = ''

  qrcodeFileList.value = []
  qrcodeImage.value = ''
}
</script>

<template>
  <div class="min-h-screen">
    <div class="- h-full w-full flex items-center justify-center">
      <div id="capture" :ref="poster" class="relative">
        <img :src="posterBg">
        <img
          :src="qrcodeImage" class="absolute" :style="{
            width: `${formData.size}px`,
            left: `${formData.x}px`,
            top: `${formData.y}px`,
          }"
        >

        <NDrawer v-model:show="showConfigDrawer" :default-width="502" resizable>
          <NDrawerContent title="配置" closable>
            <NForm ref="formRef">
              <NFormItem path="poster" label="背景">
                <NUpload :default-upload="false" :file-list="posterFileList" :show-file-list="false" :max="1" @change="handleChange">
                  <NButton>选择文件</NButton>
                </NUpload>
              </NFormItem>
              <NDivider />
              <NFormItem path="qrcode" label="二维码">
                <NUpload :default-upload="false" :file-list="qrcodeFileList" :show-file-list="showQrcodeFileList" multiple @change="handleCodeChange">
                  <NButton>选择文件</NButton>
                </NUpload>
              </NFormItem>
              <NFormItem path="size" label="大小">
                <NInputNumber v-model:value="formData.size" />
              </NFormItem>
              <NFormItem path="x" label="x">
                <NInputNumber v-model:value="formData.x" />
              </NFormItem>
              <NFormItem path="y" label="y">
                <NInputNumber v-model:value="formData.y" />
              </NFormItem>
            </NForm>

            <NDivider />

            <NProgress v-if="generation" type="line" :percentage="percentage" />
            <p v-if="qrcodeFileList.length">
              共 {{ qrcodeFileList.length }}个任务
              <template>
                ，当前：{{ currentTaskIndex + (generation ? 1 : 0) }}
              </template>
            </p>
            <template #footer>
              <NSpace>
                <NButton @click="clearForm">
                  清除
                </NButton>
                <NButton @click="submit">
                  生成
                </NButton>
              </NSpace>
            </template>
          </NDrawerContent>
        </NDrawer>
        <NFloatButton :right="20" :bottom="20" type="primary" @click="showConfigDrawer = true">
          <NIcon>
            <CogSharp />
          </NIcon>
        </NFloatButton>
      </div>
    </div>
  </div>
</template>
