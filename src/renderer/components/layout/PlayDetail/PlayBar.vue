<template>
  <div :class="$style.footer">
    <div :class="$style.footerContent">
      <div :class="$style.controlRow">
        <div :class="$style.side"></div>
        <div :class="$style.playControl">
          <div :class="$style.playBtn" :aria-label="$t('player__prev')" @click="playPrev()">
            <svg version="1.1" xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" height="100%" viewBox="0 0 1024 1024" space="preserve">
              <use xlink:href="#icon-prevMusic" />
            </svg>
          </div>
          <div :class="[$style.playBtn, $style.mainPlayBtn]" :aria-label="isPlay ? $t('player__pause') : $t('player__play')" @click="togglePlay">
            <svg v-if="isPlay" version="1.1" xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" height="100%" viewBox="0 0 1024 1024" space="preserve">
              <use xlink:href="#icon-pause" />
            </svg>
            <svg v-else version="1.1" xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" height="100%" viewBox="0 0 1024 1024" space="preserve">
              <use xlink:href="#icon-play" />
            </svg>
          </div>
          <div :class="$style.playBtn" :aria-label="$t('player__next')" @click="playNext()">
            <svg version="1.1" xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" height="100%" viewBox="0 0 1024 1024" space="preserve">
              <use xlink:href="#icon-nextMusic" />
            </svg>
          </div>
        </div>
        <div :class="[$style.side, $style.rightSide]">
          <control-btns />
        </div>
      </div>
      <div :class="$style.progressContainer">
        <div :class="$style.progressContent">
          <common-progress-bar
            :class-name="$style.progress"
            :progress="progress"
            :handle-transition-end="handleTransitionEnd"
            :is-active-transition="isActiveTransition"
          />
        </div>
      </div>
      <div :class="$style.timeLabel">
        <span :class="$style.status">{{ status }}</span>
        <span>{{ nowPlayTimeStr }}</span>
        <span :class="$style.timeSlash">/</span>
        <span>{{ maxPlayTimeStr }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { playNext, playPrev, togglePlay } from '@renderer/core/player'
import { status, isPlay } from '@renderer/store/player/state'
import usePlayProgress from '@renderer/utils/compositions/usePlayProgress'

import ControlBtns from './components/ControlBtns.vue'

const {
  nowPlayTimeStr,
  maxPlayTimeStr,
  progress,
  isActiveTransition,
  handleTransitionEnd,
} = usePlayProgress()

</script>

<style lang="less" module>
@import '@renderer/assets/styles/layout.less';

.footer {
  flex: none;
  height: 112px;
  contain: strict;
}

.footerContent {
  display: flex;
  flex-flow: column nowrap;
  height: 100%;
  padding: 0 30px 16px;
}

.controlRow {
  display: flex;
  flex: auto;
  align-items: center;
}

.side {
  flex: 1;
  display: flex;
  align-items: center;
  min-width: 0;
}

.rightSide {
  justify-content: flex-end;
}

.progressContainer {
  width: 100%;
  position: relative;
  padding: 2px 0 0;
}

.progressContent {
  position: relative;
  height: 18px;
  padding: 5px 0;
  width: 100%;
}
.progress {
  height: 100%;
}

.barTransition {
  transition-property: transform;
  transition-timing-function: ease-out;
  transition-duration: 0.2s;
}
.timeLabel {
  width: 100%;
  display: flex;
  align-items: center;
  color: var(--color-font-label);
  span {
    font-size: 13px;
  }
}
.status {
  flex: auto;
  color: var(--color-font);
}

.timeSlash {
  margin: 0 6px;
}

.playControl {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  color: var(--color-button-font);
}
.playBtn {
  flex: none;
  width: 42px;
  height: 42px;
  padding: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-button-font);
  background: transparent;
  border-radius: 0;
  box-shadow: none;
  transition: opacity 0.2s ease, transform 0.2s ease, color 0.2s ease;
  cursor: pointer;

  svg {
    fill: currentColor;
    filter: none;
  }
  &:hover {
    color: var(--color-primary-dark-200);
    transform: translateY(-1px);
  }
  &:active {
    opacity: 0.7;
    transform: scale(0.96);
  }
}

.mainPlayBtn {
  width: 56px;
  height: 56px;
  padding: 10px;
}

</style>
