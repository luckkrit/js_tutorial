<script setup>
import { computed, onMounted, onBeforeUnmount, ref } from 'vue'

const props = defineProps({
  // slug ของโปรเจกต์ เช่น "web-static-html-4m8kxmfe"
  // (ส่วนที่อยู่หลัง stackblitz.com/edit/ ใน URL ของโปรเจกต์)
  project: {
    type: String,
    required: true,
  },
  // path ของไฟล์ที่จะเปิด เช่น "timer_interval/countdowntimer.html"
  file: {
    type: String,
    default: '',
  },
  // path เริ่มต้นที่ preview จะไปเปิด (ปกติเซ็ตให้เหมือน file)
  initialPath: {
    type: String,
    default: '',
  },
  // 'both' | 'editor' | 'preview'
  view: {
    type: String,
    default: 'both',
  },
  height: {
    type: [String, Number],
    default: 500,
  },
  devToolsHeight: {
    type: [String, Number],
    default: 33,
  },
  // ซ่อนแถบเมนูด้านบนของ StackBlitz ให้ดูสะอาดขึ้นบนสไลด์
  hideNavigation: {
    type: Boolean,
    default: false,
  },
  // แสดงปุ่มลิงก์ "เปิดในแท็บใหม่" ใต้ iframe หรือไม่
  showOpenLink: {
    type: Boolean,
    default: true,
  },
  theme: {
    type: String,
    default: 'dark', // 'dark' | 'light'
  },
})

const heightPx = computed(() =>
  typeof props.height === 'number' ? `${props.height}px` : props.height
)

// path ที่ใช้เปิด preview — ถ้าไม่ส่ง initialPath มา ให้ใช้ค่าเดียวกับ file
const resolvedInitialPath = computed(() => props.initialPath || props.file)

const embedUrl = computed(() => {
  const params = new URLSearchParams()
  if (props.file) params.set('file', props.file)
  if (resolvedInitialPath.value) params.set('initialpath', resolvedInitialPath.value)
  params.set('view', props.view)
  params.set('devToolsHeight', String(props.devToolsHeight))
  params.set('theme', props.theme)
  if (props.hideNavigation) params.set('hideNavigation', '1')
  params.set('embed', '1')
  return `https://stackblitz.com/edit/${props.project}?${params.toString()}`
})

// ลิงก์เปิดโปรเจกต์เต็มจอในแท็บใหม่ (ไม่ต้องมี embed=1)
const openUrl = computed(() => {
  const params = new URLSearchParams()
  if (props.file) params.set('file', props.file)
  return `https://stackblitz.com/edit/${props.project}?${params.toString()}`
})

// ===== แก้ปัญหา: StackBlitz คำนวณ layout (editor/preview split) ผิด =====
// ถ้า iframe ถูก mount ตอน slide ยังไม่ active (container กว้าง/สูง = 0 หรือ
// อยู่นอกจอ) ตัว embed จะ boot ด้วยขนาดที่ผิด แล้วค้างอยู่แบบนั้น
// (เหมือนที่ต้องคลิก "Both" เองถึงจะ re-layout ใหม่ให้ถูก)
//
// วิธีแก้: ไม่เซ็ต src ให้ iframe ทันทีตอน mount แต่รอจนกว่า wrapper
// จะ "มองเห็นได้จริง" บนจอก่อน (ผ่าน IntersectionObserver) ค่อยเซ็ต src
// รับประกันว่า StackBlitz boot ตอนขนาด container ถูกต้องแล้วเสมอ

const wrapperRef = ref(null)
const iframeSrc = ref('')
let observer = null

function loadIframe() {
  if (iframeSrc.value) return // โหลดไปแล้ว ไม่ต้องโหลดซ้ำ
  iframeSrc.value = embedUrl.value
}

onMounted(() => {
  if (!wrapperRef.value) return

  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          // หน่วงเล็กน้อยให้ Slidev ทำ transition/layout ของ slide เสร็จก่อน
          // (กัน edge case ที่ slide เพิ่งเริ่ม fade-in ขนาดยังไม่นิ่ง)
          requestAnimationFrame(() => {
            setTimeout(loadIframe, 50)
          })
        }
      })
    },
    { threshold: 0.1 }
  )

  observer.observe(wrapperRef.value)
})

onBeforeUnmount(() => {
  observer?.disconnect()
})
</script>

<template>
  <div ref="wrapperRef" class="stackblitz-embed-wrap">
    <div
      class="stackblitz-embed-frame"
      :style="{ width: '100%', height: heightPx }"
    >
      <!-- โชว์ placeholder จนกว่าจะเริ่มโหลด iframe จริง (ตอน wrapper มองเห็นได้) -->
      <div v-if="!iframeSrc" class="stackblitz-embed-placeholder">
        กำลังเตรียม StackBlitz…
      </div>
      <iframe
        v-else
        :src="iframeSrc"
        :style="{ width: '100%', height: '100%', border: 0, borderRadius: '8px' }"
        loading="eager"
        allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
        sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
      ></iframe>
    </div>

    <div v-if="showOpenLink" class="stackblitz-embed-footer">
      <a :href="openUrl" target="_blank" rel="noopener">เปิดใน StackBlitz (แท็บใหม่) ↗</a>
    </div>
  </div>
</template>

<style scoped>
.stackblitz-embed-wrap {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.stackblitz-embed-footer {
  font-size: 12px;
  text-align: right;
}
.stackblitz-embed-footer a {
  color: #5eead4;
  text-decoration: none;
}
.stackblitz-embed-footer a:hover {
  text-decoration: underline;
}
.stackblitz-embed-frame {
  position: relative;
  border-radius: 8px;
  overflow: hidden;
  background: #0f1115;
}
.stackblitz-embed-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #8a8f98;
  font-size: 14px;
}
</style>