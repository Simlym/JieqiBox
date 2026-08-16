<template>
  <div class="pool-strip" :class="`pool-${side}`">
    <div v-for="item in items" :key="item.char" class="pool-entry">
      <v-menu location="bottom" :close-on-content-click="false">
        <template #activator="{ props }">
          <div
            v-bind="props"
            class="pool-piece-group"
            :title="$t('analysis.darkPiecePool')"
          >
            <!-- Remaining unrevealed pieces of this type -->
            <div v-if="item.count > 0" class="pool-piece">
              <img
                :src="pieceImg(item.name)"
                class="pool-img"
                :draggable="false"
              />
              <span class="badge count-badge">{{ item.count }}</span>
            </div>
            <!-- Captured unrevealed pieces of this type (greyed out) -->
            <div v-if="item.capturedCount > 0" class="pool-piece">
              <img
                :src="pieceImg(item.name)"
                class="pool-img captured"
                :draggable="false"
              />
              <span class="badge captured-badge">{{ item.capturedCount }}</span>
            </div>
            <!-- Placeholder so users can still open the adjuster when both are 0 -->
            <div
              v-if="item.count === 0 && item.capturedCount === 0"
              class="pool-piece"
            >
              <img
                :src="pieceImg(item.name)"
                class="pool-img empty"
                :draggable="false"
              />
              <span class="badge count-badge zero">0</span>
            </div>
          </div>
        </template>
        <v-card class="adjust-card" elevation="8">
          <div class="adjust-row">
            <span class="adjust-label">{{ $t('analysis.poolRemaining') }}</span>
            <v-btn
              density="compact"
              icon="mdi-minus"
              size="x-small"
              variant="text"
              :disabled="item.count <= 0"
              @click="adjustUnrevealedCount(item.char, -1)"
            />
            <span class="adjust-value">{{ item.count }}</span>
            <v-btn
              density="compact"
              icon="mdi-plus"
              size="x-small"
              variant="text"
              :disabled="item.count >= item.max"
              @click="adjustUnrevealedCount(item.char, 1)"
            />
          </div>
          <div class="adjust-row">
            <span class="adjust-label">{{ $t('analysis.poolCaptured') }}</span>
            <v-btn
              density="compact"
              icon="mdi-minus"
              size="x-small"
              variant="text"
              :disabled="item.capturedCount <= 0"
              @click="adjustCapturedUnrevealedCount(item.char, -1)"
            />
            <span class="adjust-value">{{ item.capturedCount }}</span>
            <v-btn
              density="compact"
              icon="mdi-plus"
              size="x-small"
              variant="text"
              :disabled="item.count <= 0"
              @click="adjustCapturedUnrevealedCount(item.char, 1)"
            />
          </div>
        </v-card>
      </v-menu>
    </div>

    <!-- Pool validation warning -->
    <v-icon
      v-if="hasValidationError"
      icon="mdi-alert-circle-outline"
      color="error"
      size="18"
      class="pool-warning"
      :title="validationMessage"
    />
  </div>
</template>

<script setup lang="ts">
  import { computed, inject } from 'vue'
  import { resolvePieceImage } from '@/utils/pieceImages'
  import { useInterfaceSettings } from '../composables/useInterfaceSettings'

  const props = defineProps<{
    side: 'red' | 'black'
  }>()

  const gameState: any = inject('game-state')
  const {
    unrevealedPieceCounts,
    capturedUnrevealedPieceCounts,
    validationStatus,
    adjustUnrevealedCount,
    adjustCapturedUnrevealedCount,
    getPieceNameFromChar,
  } = gameState

  const { pieceStyle } = useInterfaceSettings()

  // Initial counts per piece type (揭棋暗子初始数量)
  const INITIAL_PIECE_COUNTS: { [k: string]: number } = {
    r: 2,
    n: 2,
    b: 2,
    a: 2,
    c: 2,
    p: 5,
    k: 1,
  }

  const items = computed(() => {
    const chars =
      props.side === 'red' ? 'RNBACP'.split('') : 'rnbacp'.split('')
    return chars.map(char => ({
      char,
      name: getPieceNameFromChar(char),
      count: unrevealedPieceCounts?.value?.[char] || 0,
      capturedCount: capturedUnrevealedPieceCounts?.value?.[char] || 0,
      max: INITIAL_PIECE_COUNTS[char.toLowerCase()],
    }))
  })

  function pieceImg(pieceName: string): string {
    const style =
      pieceStyle?.value === 'internationalized' ? 'internationalized' : 'default'
    return resolvePieceImage(pieceName, style)
  }

  const hasValidationError = computed(() => {
    const status = validationStatus?.value
    if (!status) return false
    return !(
      status.includes('正常') || status.toLowerCase().includes('normal')
    )
  })

  const validationMessage = computed(() => validationStatus?.value || '')
</script>

<style lang="scss" scoped>
  .pool-strip {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    min-height: 44px;
    padding: 2px 8px;
    flex-wrap: wrap;
  }

  .pool-piece-group {
    display: flex;
    align-items: center;
    gap: 2px;
    cursor: pointer;
    border-radius: 6px;
    padding: 1px 2px;
    transition: background-color 0.15s ease;

    &:hover {
      background-color: rgba(var(--v-theme-on-surface), 0.06);
    }
  }

  .pool-piece {
    position: relative;
    width: 38px;
    height: 38px;
  }

  .pool-img {
    width: 100%;
    height: 100%;
    display: block;
    user-select: none;

    &.captured {
      filter: grayscale(0.9) brightness(0.75);
      opacity: 0.75;
    }

    &.empty {
      opacity: 0.3;
      filter: grayscale(1);
    }
  }

  .badge {
    position: absolute;
    top: -4px;
    right: -6px;
    min-width: 16px;
    height: 16px;
    padding: 0 4px;
    border-radius: 8px;
    font-size: 11px;
    font-weight: 700;
    line-height: 16px;
    text-align: center;
    color: #fff;
    pointer-events: none;
  }

  .count-badge {
    background-color: #d32f2f;

    &.zero {
      background-color: #9e9e9e;
    }
  }

  .captured-badge {
    background-color: #616161;
  }

  .pool-warning {
    margin-left: 4px;
  }

  .adjust-card {
    padding: 8px 12px;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .adjust-row {
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .adjust-label {
    width: 40px;
    font-size: 12px;
    opacity: 0.75;
  }

  .adjust-value {
    min-width: 20px;
    text-align: center;
    font-weight: 600;
  }

  // Smaller pieces on narrow screens
  @media (max-width: 768px) {
    .pool-strip {
      gap: 6px;
      min-height: 38px;
    }

    .pool-piece {
      width: 32px;
      height: 32px;
    }
  }
</style>
