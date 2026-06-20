<script setup lang="ts">
import { ref } from 'vue'
import type { GrowthStage, CareScheme, StageCareConfig } from '@/types/game'
import { STAGE_NAMES, STAGE_EMOJI } from '@/utils/constants'

const props = defineProps<{
  careScheme: CareScheme
  foodStock: number
}>()

const emit = defineEmits<{
  (e: 'toggle', enabled: boolean): void
  (e: 'updateConfig', stage: Exclude<GrowthStage, 'egg'>, config: Partial<StageCareConfig>): void
  (e: 'reset'): void
}>()

const expanded = ref(false)

const stages: Exclude<GrowthStage, 'egg'>[] = ['chick', 'juvenile', 'subadult', 'adult']

const hungerThresholdPresets = [30, 40, 50, 60, 70]
const feedAmountPresets = [5, 8, 10, 15, 20]
const calmThresholdPresets = [30, 40, 50, 60, 70, 80]

const handleStageToggle = (stage: Exclude<GrowthStage, 'egg'>, enabled: boolean) => {
  emit('updateConfig', stage, { enabled })
}

const handleHungerThreshold = (stage: Exclude<GrowthStage, 'egg'>, value: number) => {
  emit('updateConfig', stage, { hungerThreshold: value })
}

const handleFeedAmount = (stage: Exclude<GrowthStage, 'egg'>, value: number) => {
  emit('updateConfig', stage, { feedAmount: value })
}

const handleAutoCalm = (stage: Exclude<GrowthStage, 'egg'>, autoCalm: boolean) => {
  emit('updateConfig', stage, { autoCalm })
}

const handleCalmThreshold = (stage: Exclude<GrowthStage, 'egg'>, value: number) => {
  emit('updateConfig', stage, { calmThreshold: value })
}
</script>

<template>
  <div class="glass rounded-2xl overflow-hidden">
    <div
      class="px-4 py-3 flex items-center justify-between cursor-pointer hover:bg-white/10 transition-colors"
      @click="expanded = !expanded"
    >
      <div class="flex items-center gap-2">
        <span class="text-xl">🤖</span>
        <span class="font-display text-amber-300 font-bold">批量照料</span>
        <span
          :class="[
            'px-2 py-0.5 rounded-full text-xs font-bold',
            careScheme.enabled
              ? 'bg-green-500/30 text-green-300'
              : 'bg-gray-500/30 text-gray-400',
          ]"
        >
          {{ careScheme.enabled ? '运行中' : '已暂停' }}
        </span>
      </div>
      <div class="flex items-center gap-2">
        <button
          :class="[
            'relative w-12 h-6 rounded-full transition-all duration-300',
            careScheme.enabled ? 'bg-green-500' : 'bg-gray-600',
          ]"
          @click.stop="emit('toggle', !careScheme.enabled)"
        >
          <div
            :class="[
              'absolute top-0.5 w-5 h-5 rounded-full bg-white shadow transition-all duration-300',
              careScheme.enabled ? 'left-6' : 'left-0.5',
            ]"
          />
        </button>
        <span
          :class="[
            'text-sm transition-transform duration-300',
            expanded ? 'rotate-180' : '',
          ]"
        >
          ▼
        </span>
      </div>
    </div>

    <div
      v-if="expanded"
      class="border-t border-white/10 p-4 space-y-4"
    >
      <div class="text-xs text-white/60">
        自动照料不同成长阶段的小鸟，手动喂食优先，不冲突。
      </div>

      <div
        v-for="stage in stages"
        :key="stage"
        :class="[
          'rounded-xl p-3 space-y-3 transition-all',
          careScheme.stageConfigs[stage].enabled
            ? 'bg-white/8 border border-white/10'
            : 'bg-white/5 border border-white/5 opacity-60',
        ]"
      >
        <div class="flex items-center justify-between">
          <div class="flex items-center gap-2">
            <span class="text-lg">{{ STAGE_EMOJI[stage] }}</span>
            <span class="font-bold text-white text-sm">{{ STAGE_NAMES[stage] }}</span>
          </div>
          <button
            :class="[
              'relative w-10 h-5 rounded-full transition-all duration-300',
              careScheme.stageConfigs[stage].enabled ? 'bg-green-500' : 'bg-gray-600',
            ]"
            @click="handleStageToggle(stage, !careScheme.stageConfigs[stage].enabled)"
          >
            <div
              :class="[
                'absolute top-0.5 w-4 h-4 rounded-full bg-white shadow transition-all duration-300',
              ]"
              :style="{ left: careScheme.stageConfigs[stage].enabled ? '22px' : '2px' }"
            />
          </button>
        </div>

        <div
          v-if="careScheme.stageConfigs[stage].enabled"
          class="space-y-3"
        >
          <div class="space-y-1.5">
            <div class="flex items-center justify-between text-xs">
              <span class="text-white/70">饥饿阈值</span>
              <span class="text-amber-300 font-bold">{{ careScheme.stageConfigs[stage].hungerThreshold }}%</span>
            </div>
            <div class="flex gap-1.5 flex-wrap">
              <button
                v-for="val in hungerThresholdPresets"
                :key="val"
                :class="[
                  'px-2 py-1 rounded-lg text-xs font-medium transition-all',
                  careScheme.stageConfigs[stage].hungerThreshold === val
                    ? 'bg-amber-500 text-white'
                    : 'bg-white/10 text-white/70 hover:bg-white/20',
                ]"
                @click="handleHungerThreshold(stage, val)"
              >
                {{ val }}%
              </button>
            </div>
          </div>

          <div class="space-y-1.5">
            <div class="flex items-center justify-between text-xs">
              <span class="text-white/70">喂食量</span>
              <span class="text-green-300 font-bold">{{ careScheme.stageConfigs[stage].feedAmount }}🍒</span>
            </div>
            <div class="flex gap-1.5 flex-wrap">
              <button
                v-for="val in feedAmountPresets"
                :key="val"
                :class="[
                  'px-2 py-1 rounded-lg text-xs font-medium transition-all',
                  foodStock < val
                    ? 'bg-red-900/30 text-red-400 cursor-not-allowed'
                    : careScheme.stageConfigs[stage].feedAmount === val
                      ? 'bg-green-500 text-white'
                      : 'bg-white/10 text-white/70 hover:bg-white/20',
                ]"
                :disabled="foodStock < val"
                @click="handleFeedAmount(stage, val)"
              >
                {{ val }}
              </button>
            </div>
          </div>

          <div class="flex items-center justify-between">
            <span class="text-xs text-white/70">自动安抚</span>
            <button
              :class="[
                'relative w-10 h-5 rounded-full transition-all duration-300',
                careScheme.stageConfigs[stage].autoCalm ? 'bg-purple-500' : 'bg-gray-600',
              ]"
              @click="handleAutoCalm(stage, !careScheme.stageConfigs[stage].autoCalm)"
            >
              <div
                :class="[
                  'absolute top-0.5 w-4 h-4 rounded-full bg-white shadow transition-all duration-300',
                ]"
                :style="{ left: careScheme.stageConfigs[stage].autoCalm ? '22px' : '2px' }"
              />
            </button>
          </div>

          <div
            v-if="careScheme.stageConfigs[stage].autoCalm"
            class="space-y-1.5"
          >
            <div class="flex items-center justify-between text-xs">
              <span class="text-white/70">恐惧阈值</span>
              <span class="text-purple-300 font-bold">{{ careScheme.stageConfigs[stage].calmThreshold }}%</span>
            </div>
            <div class="flex gap-1.5 flex-wrap">
              <button
                v-for="val in calmThresholdPresets"
                :key="val"
                :class="[
                  'px-2 py-1 rounded-lg text-xs font-medium transition-all',
                  careScheme.stageConfigs[stage].calmThreshold === val
                    ? 'bg-purple-500 text-white'
                    : 'bg-white/10 text-white/70 hover:bg-white/20',
                ]"
                @click="handleCalmThreshold(stage, val)"
              >
                {{ val }}%
              </button>
            </div>
          </div>
        </div>
      </div>

      <button
        class="w-full py-2 rounded-xl text-xs font-medium bg-white/10 text-white/70 hover:bg-white/20 transition-all"
        @click="emit('reset')"
      >
        🔄 恢复默认配置
      </button>
    </div>
  </div>
</template>
