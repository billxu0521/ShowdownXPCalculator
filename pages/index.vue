<template>
  <div class="container mx-auto py-8">
    <h1 class="text-2xl font-bold mb-4">經驗值計算器</h1>
    <!-- 玩家經驗值輸入區 -->
    <div class="mb-6">
      <label for="level">等級 (1-50)：</label>
      <input v-model.number="playerLevel" type="number" min="1" max="50" id="level" class="w-32 border-2 border-gray-300 rounded-md px-2 py-1 focus:border-primary-500 focus:ring-2 focus:ring-primary-200 hover:border-primary-400 transition-colors" />
      <label for="exp">目前經驗值：</label>
      <input v-model.number="playerExp" type="number" min="0" :max="levelExp(playerLevel)" id="exp" class="w-32 border-2 border-gray-300 rounded-md px-2 py-1 focus:border-primary-500 focus:ring-2 focus:ring-primary-200 hover:border-primary-400 transition-colors" />
    </div>
    <!-- 經驗值顯示 -->
    <div class="sticky top-0 bg-white z-10 pb-4">
      <div class="flex flex-col gap-4">
        <!-- 當前等級經驗值進度 -->
        <div class="flex flex-col gap-1">
          <div class="flex justify-between text-sm">
            <span>等級 {{ playerLevel }} 經驗值進度</span>
            <span>{{ playerExp }} / {{ levelExp(playerLevel) }}</span>
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2.5 relative">
            <!-- 目前經驗值進度 -->
            <div class="bg-green-500 h-2.5 rounded-full transition-all duration-300 absolute top-0 left-0"
              :style="{ width: `${(playerExp / levelExp(playerLevel)) * 100}%` }">
            </div>
            <!-- 獲得經驗值進度 -->
            <div class="bg-blue-500 h-2.5 rounded-full transition-all duration-300 absolute top-0 left-0"
              :style="{ 
                width: `${Math.min((totalExp / levelExp(playerLevel)) * 100, 100 - (playerExp / levelExp(playerLevel)) * 100)}%`,
                left: `${(playerExp / levelExp(playerLevel)) * 100}%`
              }">
            </div>
          </div>
          <div class="flex justify-between text-sm">
            <div class="flex gap-4">
              <span class="text-primary-500">目前經驗值</span>
              <span class="text-blue-500">獲得經驗值</span>
            </div>
            <span class="text-blue-500">+{{ totalExp }}</span>
          </div>
        </div>

        <!-- 總經驗值進度 -->
        <div class="flex flex-col gap-1">
          <div class="flex justify-between text-sm">
            <span>總經驗值進度</span>
            <span>{{ getTotalExpForLevel(playerLevel) + playerExp }} / {{ getTotalExpForLevel(50) }}</span>
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2.5">
            <div class="bg-blue-500 h-2.5 rounded-full transition-all duration-300"
              :style="{ width: `${((getTotalExpForLevel(playerLevel) + playerExp) / getTotalExpForLevel(50)) * 100}%` }">
            </div>
          </div>
        </div>

        <!-- 其他資訊 -->
        <div class="flex flex-col gap-2 text-sm">
          <span>當前等級獎勵經驗值：{{ getBonusExpForLevel(playerLevel) }}</span>
          <span>預期到達等級：{{ getExpectedLevel(playerLevel, playerExp, totalExp) }}</span>
          <span v-if="playerExp + totalExp >= levelExp(playerLevel)" class="text-green-600 font-bold">
            可以升級！
          </span>
        </div>

        <button v-if="totalExp > 0" class="mt-2 w-32 bg-red-500 text-white px-4 py-2 rounded" @click="resetCounts">
          重置點擊
        </button>
      </div>
    </div>

       <!-- 怪物/事件列表 -->
       <div class="mb-6">
      <h2 class="text-xl font-semibold mb-2">怪物/事件列表</h2>
      <div class="grid grid-cols-3 gap-2">
        <div v-for="(item, idx) in monsterList" :key="item.id" 
          @click="addCount(idx)"
          class="cursor-pointer hover:ring-2 hover:ring-primary-500 transition aspect-square flex items-center justify-center">
          <div class="flex flex-col gap-1 items-center text-center">
            <span class="font-bold">{{ item.name }}</span>
            <span>經驗值：{{ item.exp }}</span>
            <span>點擊次數：{{ item.count }}</span>
          </div>
        </div>
      </div>
    </div>

    
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

// 玩家等級與經驗值
const playerLevel = ref(1)
const playerExp = ref(0)

// 怪物/事件列表
const monsterList = ref([
  // 一般怪物
  { id: 1, name: 'Leech', exp: 10, count: 0, type: 'monster' },
  { id: 2, name: 'Grunt', exp: 20, count: 0, type: 'monster' },
  { id: 3, name: 'Hellhound', exp: 40, count: 0, type: 'monster' },
  { id: 4, name: 'Waterdevil', exp: 20, count: 0, type: 'monster' },
  { id: 5, name: 'Hive', exp: 60, count: 0, type: 'monster' },
  { id: 6, name: 'Immolator', exp: 60, count: 0, type: 'monster' },
  { id: 7, name: 'Armoured', exp: 60, count: 0, type: 'monster' },
  { id: 8, name: 'Meathead', exp: 300, count: 0, type: 'monster' },

  // 事件
  { id: 9, name: 'Clue Pickup', exp: 200, count: 0, type: 'event' },
  { id: 10, name: 'Lair Discovered', exp: 100, count: 0, type: 'event' },
  { id: 11, name: 'Boss Kill', exp: 400, count: 0, type: 'event' },
  { id: 12, name: 'Solo Boss Kill', exp: 400, count: 0, type: 'event' },
  { id: 13, name: 'Boss Banish', exp: 350, count: 0, type: 'event' },
  { id: 14, name: 'Bounty Extract', exp: 400, count: 0, type: 'event', isSpecial: true },
  { id: 15, name: 'Partner Extract', exp: 800, count: 0, type: 'event', isSpecial: true },
  { id: 16, name: 'Underdog Bonus', exp: 500, count: 0, type: 'event', isSpecial: true },
  { id: 17, name: 'Clean Sweep', exp: 100, count: 0, type: 'event' },
  { id: 18, name: 'Extraction Time', exp: 4, count: 0, type: 'event' },
  { id: 19, name: 'Revive Partner', exp: 100, count: 0, type: 'event' },
  { id: 20, name: 'Kill Teammate', exp: -500, count: 0, type: 'event' },

  // Soul Survivor 事件
  { id: 21, name: 'Rift Pickup', exp: 100, count: 0, type: 'soul' },
  { id: 22, name: 'Wellspring Activation', exp: 100, count: 0, type: 'soul' },
  { id: 23, name: 'Energy Absorbed', exp: 1, count: 0, type: 'soul', isSpecial: true },
  { id: 24, name: 'Final Survivor Bonus', exp: 0, count: 0, type: 'soul', isSpecial: true },
])

// 經驗值門檻表
const levelExpTable = [
  { level: 1, expRequired: 100, totalExp: 0, bonusExp: 0 },
  { level: 2, expRequired: 100, totalExp: 100, bonusExp: 0 },
  { level: 3, expRequired: 100, totalExp: 200, bonusExp: 0 },
  { level: 4, expRequired: 200, totalExp: 400, bonusExp: 250 },
  { level: 5, expRequired: 200, totalExp: 600, bonusExp: 0 },
  { level: 6, expRequired: 200, totalExp: 800, bonusExp: 0 },
  { level: 7, expRequired: 200, totalExp: 1000, bonusExp: 0 },
  { level: 8, expRequired: 300, totalExp: 1300, bonusExp: 0 },
  { level: 9, expRequired: 300, totalExp: 1600, bonusExp: 250 },
  { level: 10, expRequired: 300, totalExp: 1900, bonusExp: 0 },
  { level: 11, expRequired: 300, totalExp: 2200, bonusExp: 0 },
  { level: 12, expRequired: 300, totalExp: 2500, bonusExp: 0 },
  { level: 13, expRequired: 300, totalExp: 2800, bonusExp: 250 },
  { level: 14, expRequired: 300, totalExp: 3100, bonusExp: 0 },
  { level: 15, expRequired: 300, totalExp: 3400, bonusExp: 0 },
  { level: 16, expRequired: 300, totalExp: 3700, bonusExp: 0 },
  { level: 17, expRequired: 300, totalExp: 4000, bonusExp: 0 },
  { level: 18, expRequired: 300, totalExp: 4300, bonusExp: 0 },
  { level: 19, expRequired: 400, totalExp: 4700, bonusExp: 500 },
  { level: 20, expRequired: 400, totalExp: 5100, bonusExp: 0 },
  { level: 21, expRequired: 400, totalExp: 5500, bonusExp: 0 },
  { level: 22, expRequired: 400, totalExp: 5900, bonusExp: 0 },
  { level: 23, expRequired: 400, totalExp: 6300, bonusExp: 0 },
  { level: 24, expRequired: 400, totalExp: 6700, bonusExp: 0 },
  { level: 25, expRequired: 400, totalExp: 7100, bonusExp: 0 },
  { level: 26, expRequired: 500, totalExp: 7600, bonusExp: 500 },
  { level: 27, expRequired: 500, totalExp: 8100, bonusExp: 0 },
  { level: 28, expRequired: 500, totalExp: 8600, bonusExp: 0 },
  { level: 29, expRequired: 500, totalExp: 9100, bonusExp: 0 },
  { level: 30, expRequired: 500, totalExp: 9600, bonusExp: 0 },
  { level: 31, expRequired: 500, totalExp: 10100, bonusExp: 0 },
  { level: 32, expRequired: 500, totalExp: 10600, bonusExp: 0 },
  { level: 33, expRequired: 500, totalExp: 11100, bonusExp: 0 },
  { level: 34, expRequired: 500, totalExp: 11600, bonusExp: 500 },
  { level: 35, expRequired: 500, totalExp: 12100, bonusExp: 0 },
  { level: 36, expRequired: 600, totalExp: 12700, bonusExp: 0 },
  { level: 37, expRequired: 600, totalExp: 13300, bonusExp: 0 },
  { level: 38, expRequired: 600, totalExp: 13900, bonusExp: 0 },
  { level: 39, expRequired: 600, totalExp: 14500, bonusExp: 0 },
  { level: 40, expRequired: 600, totalExp: 15100, bonusExp: 0 },
  { level: 41, expRequired: 600, totalExp: 15700, bonusExp: 0 },
  { level: 42, expRequired: 600, totalExp: 16300, bonusExp: 750 },
  { level: 43, expRequired: 600, totalExp: 16900, bonusExp: 0 },
  { level: 44, expRequired: 600, totalExp: 17500, bonusExp: 0 },
  { level: 45, expRequired: 600, totalExp: 18100, bonusExp: 0 },
  { level: 46, expRequired: 600, totalExp: 18700, bonusExp: 0 },
  { level: 47, expRequired: 600, totalExp: 19300, bonusExp: 0 },
  { level: 48, expRequired: 600, totalExp: 19900, bonusExp: 0 },
  { level: 49, expRequired: 600, totalExp: 20500, bonusExp: 0 },
  { level: 50, expRequired: 600, totalExp: 21100, bonusExp: 1000 },
]

// 點擊卡片增加次數
function addCount(idx: number) {
  monsterList.value[idx].count++
}

// 重置所有點擊次數
function resetCounts() {
  monsterList.value.forEach(item => (item.count = 0))
}

// 計算總共預期獲得的經驗值
const totalExp = computed(() =>
  monsterList.value.reduce((sum: number, m: { exp: number; count: number; isSpecial?: boolean; type: string; name: string }) => {
    if (m.isSpecial) {
      switch (m.type) {
        case 'event':
          if (m.name === 'Bounty Extract') {
            // Bounty Extract: 400-700 XP
            return sum + (400 + Math.floor(Math.random() * 301)) * m.count;
          } else if (m.name === 'Partner Extract') {
            // Partner Extract: x2 Bounty Extract
            return sum + (800 + Math.floor(Math.random() * 601)) * m.count;
          } else if (m.name === 'Underdog Bonus') {
            // Underdog Bonus: 500-2000 XP
            return sum + (500 + Math.floor(Math.random() * 1501)) * m.count;
          }
          break;
        case 'soul':
          if (m.name === 'Energy Absorbed') {
            // Energy Absorbed: 1-400 XP
            return sum + (1 + Math.floor(Math.random() * 400)) * m.count;
          } else if (m.name === 'Final Survivor Bonus') {
            // Final Survivor Bonus: 4x Energy Absorbed
            const energyAbsorbed = monsterList.value.find(item => item.name === 'Energy Absorbed');
            if (energyAbsorbed) {
              const energyExp = (1 + Math.floor(Math.random() * 400)) * energyAbsorbed.count;
              return sum + energyExp * 4 * m.count;
            }
          }
          break;
      }
    }
    return sum + m.exp * m.count;
  }, 0)
)

// 修改經驗值計算函數
function levelExp(level: number) {
  const levelData = levelExpTable.find(l => l.level === level)
  return levelData ? levelData.expRequired : 0
}

// 計算總經驗值
function getTotalExpForLevel(level: number) {
  const levelData = levelExpTable.find(l => l.level === level)
  return levelData ? levelData.totalExp : 0
}

// 計算獎勵經驗值
function getBonusExpForLevel(level: number) {
  const levelData = levelExpTable.find(l => l.level === level)
  return levelData ? levelData.bonusExp : 0
}

// 計算預期到達的等級
function getExpectedLevel(currentLevel: number, currentExp: number, additionalExp: number) {
  const totalExp = getTotalExpForLevel(currentLevel) + currentExp + additionalExp;
  
  // 如果總經驗值小於當前等級的總經驗值，返回當前等級
  if (totalExp < getTotalExpForLevel(currentLevel)) {
    return currentLevel;
  }
  
  // 從當前等級開始往後找，找到第一個總經驗值超過目標的等級
  for (let level = currentLevel + 1; level <= 50; level++) {
    const levelData = levelExpTable.find(l => l.level === level);
    if (!levelData) continue;
    
    if (totalExp < levelData.totalExp) {
      return level - 1;
    }
  }
  
  return 50; // 如果超過所有等級的總經驗值，返回最高等級
}
</script>

<style scoped>
.container {
  max-width: 800px;
}
/* 可根據需要補充原本 Nuxt UI 的樣式 */
</style> 