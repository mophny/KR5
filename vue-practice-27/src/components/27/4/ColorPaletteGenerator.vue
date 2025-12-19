<template>
  <div class="palette-container">
    <h2>Продвинутый Генератор Палитр</h2>

    <!-- Управление генерацией -->
    <div class="controls">
      <label>
        Базовый цвет:
        <input type="color" v-model="baseColor" />
      </label>

      <label>
        Тип палитры:
        <select v-model="paletteType">
          <option value="analogous">Аналогичная</option>
          <option value="complementary">Комплементарная</option>
          <option value="triad">Триада</option>
          <option value="monochromatic">Монохромная</option>
        </select>
      </label>

      <label>
        Настроение:
        <select v-model="mood">
          <option value="neutral">Нейтральное</option>
          <option value="calm">Спокойное</option>
          <option value="energetic">Энергичное</option>
          <option value="professional">Профессиональное</option>
        </select>
      </label>

      <label>
        Количество цветов:
        <select v-model="colorCount">
          <option :value="3">3</option>
          <option :value="5">5</option>
          <option :value="7">7</option>
        </select>
      </label>

      <label>
        Формат:
        <select v-model="colorFormat">
          <option value="hex">HEX</option>
          <option value="rgb">RGB</option>
        </select>
      </label>

      <button class="button primary" @click="generatePalette">
        🎲 Сгенерировать
      </button>
    </div>

    <!-- Палитра -->
    <div class="palette">
      <div
        v-for="(color, index) in palette"
        :key="index"
        class="color-card"
        :style="{ backgroundColor: color.hex }"
        @click="copyColor(color.hex)"
      >
        <div class="color-info">
          <p>{{ formatColor(color.hex) }} ({{ color.accessibility }})</p>
          <button class="lock-button" @click.stop="toggleLock(index)">
            {{ color.locked ? '🔒' : '🔓' }}
          </button>
        </div>
      </div>
    </div>

    <div v-if="copied" class="toast">Скопировано: {{ copied }}</div>

    <!-- Цветовой круг -->
    <div class="color-wheel-container">
      <h3>Цветовой круг</h3>
      <canvas ref="colorWheel" width="300" height="300"></canvas>
    </div>

    <!-- Preview интерфейса -->
    <div class="preview">
      <h3>Превью UI</h3>
      <button @click="darkMode = !darkMode" class="button small">
        {{ darkMode ? 'Светлый фон' : 'Тёмный фон' }}
      </button>

      <div class="preview-box" :class="{ dark: darkMode }">
        <button class="demo-btn" :style="{ backgroundColor: palette[0]?.hex }">
          Кнопка
        </button>
        <div class="demo-card" :style="{ backgroundColor: palette[1]?.hex }">
          Карточка
        </div>
        <h4 class="demo-title" :style="{ color: palette[2]?.hex }">
          Заголовок
        </h4>
      </div>
    </div>

    <!-- Сохранение и экспорт -->
    <div class="collections">
      <h3>Коллекции палитр</h3>

      <input
        type="text"
        v-model="collectionName"
        placeholder="Название коллекции"
      />
      <button @click="saveCollection" class="button">Сохранить палитру</button>

      <input
        type="text"
        v-model="searchQuery"
        placeholder="Поиск по названию..."
        class="search-input"
      />
      <div class="collection-filters">
        <button @click="showFavorites = !showFavorites" class="button small">
          {{ showFavorites ? 'Все палитры' : 'Только избранное' }}
        </button>
      </div>

      <div v-if="filteredCollections.length">
        <ul>
          <li
            v-for="(c, i) in filteredCollections"
            :key="i"
            class="collection-item"
          >
            <!-- Название и кнопки -->
            <div class="collection-header">
              <input type="text" v-model="c.name" @blur="editCollection(i)" />
              <span v-if="c.favorite">⭐</span>
              <button @click="toggleFavorite(c)">Избранное</button>
              <button @click="loadCollection(c)">Загрузить</button>
              <button @click="deleteCollection(c)">Удалить</button>
              <input v-model="c.name" @blur="saveToLocal()" />
            </div>

            <!-- Редактирование цветов -->
            <div class="edit-colors">
              <div
                v-for="(color, idx) in c.palette"
                :key="idx"
                class="edit-color"
              >
                <input
                  type="color"
                  v-model="color.hex"
                  @input="editCollection(i)"
                />
                <span>{{ formatColor(color.hex) }}</span>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="export">
      <h3>Экспорт палитры</h3>
      <div class="export-buttons">
        <button @click="exportCSS" class="button small">CSS Variables</button>
        <button @click="exportSCSS" class="button small">SCSS Variables</button>
        <button @click="exportTailwind" class="button small">
          Tailwind Config
        </button>
      </div>
      <pre v-if="exported">{{ exported }}</pre>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch, nextTick, computed } from 'vue'

export default {
  name: 'AdvancedColorPalette',
  setup () {
    const baseColor = ref('#3498db')
    const paletteType = ref('analogous')
    const mood = ref('neutral')
    const colorCount = ref(5)
    const darkMode = ref(false)
    const colorFormat = ref('hex')
    const copied = ref('')
    const palette = ref([])
    const collectionName = ref('')
    const collections = ref([])
    const exported = ref('')
    const colorWheel = ref(null)
    const searchQuery = ref('')

    const hexToRgb = hex => {
      const r = parseInt(hex.slice(1, 3), 16)
      const g = parseInt(hex.slice(3, 5), 16)
      const b = parseInt(hex.slice(5, 7), 16)
      return { r, g, b }
    }

    const rgbToHex = ({ r, g, b }) =>
      '#' +
      [r, g, b]
        .map(x => x.toString(16).padStart(2, '0'))
        .join('')
        .toUpperCase()

    const formatColor = hex => {
      if (colorFormat.value === 'hex') return hex
      const { r, g, b } = hexToRgb(hex)
      return `rgb(${r}, ${g}, ${b})`
    }

    const shiftColor = (rgb, rShift = 0, gShift = 0, bShift = 0) => ({
      r: Math.min(255, Math.max(0, rgb.r + rShift)),
      g: Math.min(255, Math.max(0, rgb.g + gShift)),
      b: Math.min(255, Math.max(0, rgb.b + bShift))
    })

    const contrastRatio = (rgb1, rgb2 = { r: 255, g: 255, b: 255 }) => {
      const luminance = ({ r, g, b }) => {
        const a = [r, g, b].map(v => {
          v /= 255
          return v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4)
        })
        return 0.2126 * a[0] + 0.7152 * a[1] + 0.0722 * a[2]
      }
      const L1 = luminance(rgb1)
      const L2 = luminance(rgb2)
      return L1 > L2 ? (L1 + 0.05) / (L2 + 0.05) : (L2 + 0.05) / (L1 + 0.05)
    }

    const accessibilityLevel = ratio => {
      if (ratio >= 7) return 'AAA'
      if (ratio >= 4.5) return 'AA'
      return 'Недостаточно'
    }

    const generatePalette = () => {
      const base = hexToRgb(baseColor.value)
      const newPalette = []

      for (let i = 0; i < colorCount.value; i++) {
        const prev = palette.value[i]
        if (prev && prev.locked) {
          newPalette.push(prev)
          continue
        }

        let c
        switch (paletteType.value) {
          case 'analogous':
            c = shiftColor(base, i * 20, i * 10, -i * 5)
            break
          case 'complementary':
            c = shiftColor(base, i % 2 === 0 ? 180 : 0, i * 10, -i * 10)
            break
          case 'triad':
            c = shiftColor(base, 0, i * 120, 0)
            break
          case 'monochromatic':
            c = shiftColor(base, i * 10, i * 10, i * 10)
            break
        }

        if (mood.value === 'calm') c = shiftColor(c, -10, -5, 0)
        if (mood.value === 'energetic') c = shiftColor(c, 20, 10, 5)
        if (mood.value === 'professional') c = shiftColor(c, -5, -5, -5)

        const hex = rgbToHex(c)
        newPalette.push({
          hex,
          locked: false,
          accessibility: accessibilityLevel(contrastRatio(c))
        })
      }

      palette.value = newPalette
      saveToLocal()
    }

    const toggleLock = index => {
      palette.value[index].locked = !palette.value[index].locked
      saveToLocal()
    }

    const copyColor = async hex => {
      await navigator.clipboard.writeText(formatColor(hex))
      copied.value = formatColor(hex)
      setTimeout(() => (copied.value = ''), 1200)
    }

    const saveToLocal = () => {
      localStorage.setItem(
        'savedPalette',
        JSON.stringify({
          palette: palette.value,
          format: colorFormat.value
        })
      )
      localStorage.setItem('collections', JSON.stringify(collections.value))
    }

    const drawColorWheel = () => {
      if (!colorWheel.value || !palette.value.length) return
      const ctx = colorWheel.value.getContext('2d')
      const cx = colorWheel.value.width / 2
      const cy = colorWheel.value.height / 2
      const radius = 120
      ctx.clearRect(0, 0, colorWheel.value.width, colorWheel.value.height)

      palette.value.forEach((c, i) => {
        const start = (i / palette.value.length) * 2 * Math.PI
        const end = ((i + 1) / palette.value.length) * 2 * Math.PI
        ctx.beginPath()
        ctx.moveTo(cx, cy)
        ctx.arc(cx, cy, radius, start, end)
        ctx.closePath()
        ctx.fillStyle = c.hex
        ctx.fill()
      })
    }

    watch(palette, () => nextTick(drawColorWheel))
    watch(colorFormat, saveToLocal)

    // --- Управление коллекциями ---
    const saveCollection = () => {
      if (!collectionName.value.trim()) return
      collections.value.push({
        name: collectionName.value.trim(),
        palette: JSON.parse(JSON.stringify(palette.value)),
        favorite: false
      })
      collectionName.value = ''
      saveToLocal()
    }

    const loadCollection = collection => {
      palette.value = JSON.parse(JSON.stringify(collection.palette))
    }

    const deleteCollection = collection => {
      const index = collections.value.indexOf(collection)
      if (index !== -1) {
        collections.value.splice(index, 1)
        saveToLocal()
      }
    }

    const editCollection = i => {
      saveToLocal()
    }

    const toggleFavorite = collection => {
      collection.favorite = !collection.favorite
      saveToLocal()
    }

    const showFavorites = ref(false)

    const filteredCollections = computed(() => {
      return collections.value
        .filter(c =>
          c.name.toLowerCase().includes(searchQuery.value.toLowerCase())
        )
        .filter(c => !showFavorites.value || c.favorite) // фильтр по избранным
    })

    const exportCSS = () => {
      exported.value = palette.value
        .map((c, i) => `--color-${i + 1}: ${formatColor(c.hex)};`)
        .join('\n')
    }
    const exportSCSS = () => {
      exported.value = palette.value
        .map((c, i) => `$color-${i + 1}: ${formatColor(c.hex)};`)
        .join('\n')
    }
    const exportTailwind = () => {
      exported.value = palette.value
        .map((c, i) => `c${i + 1}: '${formatColor(c.hex)}',`)
        .join('\n')
    }

    onMounted(() => {
      const saved = localStorage.getItem('savedPalette')
      if (saved) {
        const parsed = JSON.parse(saved)
        palette.value = parsed.palette || []
        colorFormat.value = parsed.format || 'hex'
      } else {
        generatePalette()
      }

      const savedCollections = localStorage.getItem('collections')
      if (savedCollections) collections.value = JSON.parse(savedCollections)
    })

    return {
      baseColor,
      paletteType,
      mood,
      colorCount,
      darkMode,
      colorFormat,
      copied,
      palette,
      collectionName,
      collections,
      exported,
      colorWheel,
      searchQuery,
      filteredCollections,
      generatePalette,
      toggleLock,
      copyColor,
      formatColor,
      saveCollection,
      loadCollection,
      deleteCollection,
      editCollection,
      showFavorites,
      toggleFavorite,
      exportCSS,
      exportSCSS,
      exportTailwind
    }
  }
}
</script>

<style scoped>
.palette-container {
  max-width: 900px;
  margin: 20px auto;
  padding: 20px;
}
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 20px;
}
.palette {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
}
.color-card {
  flex: 1;
  height: 120px;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: end;
  padding: 10px;
  color: white;
  font-weight: bold;
  position: relative;
}
.color-info {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.lock-button {
  background: rgba(0, 0, 0, 0.4);
  border: none;
  font-size: 20px;
  cursor: pointer;
  border-radius: 5px;
  padding: 5px;
  color: white;
}
.toast {
  background-color: #28a745;
  color: white;
  padding: 10px 15px;
  border-radius: 6px;
  display: inline-block;
  margin-bottom: 20px;
  animation: fade 1.2s ease-out forwards;
}
@keyframes fade {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}
.preview {
  margin-top: 20px;
}
.preview-box {
  padding: 20px;
  border-radius: 10px;
  background: #f2f2f2;
}
.preview-box.dark {
  background: #222;
}
.demo-btn {
  padding: 10px 20px;
  border-radius: 5px;
  color: white;
  border: none;
  margin-bottom: 10px;
}
.demo-card {
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 10px;
  color: white;
}
.demo-title {
  font-size: 20px;
}
.collections {
  margin-top: 20px;
}
.collections ul {
  list-style: none;
  padding-left: 0;
}
.collections li {
  margin-bottom: 5px;
}
.export {
  margin-top: 10px;
}
.export pre {
  background: #eee;
  padding: 10px;
  border-radius: 5px;
  white-space: pre-wrap;
}
.color-wheel-container {
  margin-top: 30px;
  text-align: center;
}
canvas {
  border: 1px solid #ccc;
  border-radius: 50%;
}
.collection-item {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 6px;
}

.collection-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}

.edit-colors {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.edit-color {
  display: flex;
  align-items: center;
  gap: 5px;
}
.export-buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.collection-filters {
  margin-bottom: 10px;
}
</style>
