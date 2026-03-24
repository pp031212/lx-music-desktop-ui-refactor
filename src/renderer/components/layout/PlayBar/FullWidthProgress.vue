<template>
  <div :class="$style.player">
    <div :class="$style.progress">
      <common-progress-bar v-if="!isShowPlayerDetail" :class-name="$style.progressBar" :progress="progress" :handle-transition-end="handleTransitionEnd" :is-active-transition="isActiveTransition" />
    </div>
    <div :class="$style.picContent" :aria-label="$t('player__pic_tip')" @contextmenu="handleToMusicLocation" @click="showPlayerDetail">
      <img v-if="musicInfo.pic" :src="musicInfo.pic" decoding="async" @error="imgError">
      <div v-else :class="$style.emptyPic">L<span>X</span></div>
    </div>
    <div :class="$style.infoContent">
      <div :class="$style.title" :aria-label="title + $t('copy_tip')" @click="handleCopy(title)">
        {{ title }}
      </div>
      <div :class="$style.status">{{ statusText }}</div>
    </div>
    <div :class="$style.timeContent">
      <span>{{ nowPlayTimeStr }}</span>
      <span style="margin: 0 1px;">/</span>
      <span>{{ maxPlayTimeStr }}</span>
    </div>
    <!-- <play-progress /> -->
    <control-btns />
    <div :class="$style.playBtnContent">
      <div :class="$style.playBtn" :aria-label="$t('player__prev')" @click="playPrev()">
        <svg version="1.1" xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" height="100%" viewBox="0 0 1024 1024" space="preserve">
          <use xlink:href="#icon-prevMusic" />
        </svg>
      </div>
      <div :class="$style.playBtn" :aria-label="isPlay ? $t('player__pause') : $t('player__play')" @click="togglePlay">
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
  </div>
</template>

<script>
import { computed } from '@common/utils/vueTools'
import { useRouter } from '@common/utils/vueRouter'
import { clipboardWriteText } from '@common/utils/electron'
import ControlBtns from './ControlBtns.vue'
// import PlayProgress from './PlayProgress'
import usePlayProgress from '@renderer/utils/compositions/usePlayProgress'
// import { lyric } from '@renderer/core/share/lyric'
import {
  statusText,
  musicInfo,
  isShowPlayerDetail,
  isPlay,
  playInfo,
  playMusicInfo,
} from '@renderer/store/player/state'
import {
  setMusicInfo,
  setShowPlayerDetail,
} from '@renderer/store/player/action'
import { appSetting } from '@renderer/store/setting'
import { togglePlay, playNext, playPrev } from '@renderer/core/player'
import { LIST_IDS } from '@common/constants'

export default {
  name: 'CorePlayBar',
  components: {
    ControlBtns,
    // PlayProgress,
  },
  setup() {
    const router = useRouter()

    const {
      nowPlayTimeStr,
      maxPlayTimeStr,
      progress,
      isActiveTransition,
      handleTransitionEnd,
    } = usePlayProgress()

    const showPlayerDetail = () => {
      if (!playMusicInfo.musicInfo) return
      setShowPlayerDetail(true)
    }
    const handleCopy = (text) => {
      clipboardWriteText(text)
    }

    const imgError = () => {
      // console.log(e)
      setMusicInfo({ pic: null })
    }

    const handleToMusicLocation = () => {
      const listId = playMusicInfo.listId
      if (!listId || listId == LIST_IDS.DOWNLOAD || !playMusicInfo.musicInfo) return
      if (playInfo.playIndex == -1) return
      void router.push({
        path: '/list',
        query: {
          id: listId,
          scrollIndex: playInfo.playIndex,
        },
      })
    }

    const title = computed(() => {
      return musicInfo.name
        ? appSetting['download.fileName'].replace('歌名', musicInfo.name).replace('歌手', musicInfo.singer)
        : ''
    })

    // onBeforeUnmount(() => {
    // window.eventHub.emit(eventPlayerNames.setTogglePlay)
    // })

    return {
      musicInfo,
      nowPlayTimeStr,
      maxPlayTimeStr,
      progress,
      isActiveTransition,
      handleTransitionEnd,
      handleCopy,
      imgError,
      statusText,
      title,
      showPlayerDetail,
      isPlay,
      togglePlay,
      playNext,
      playPrev,
      handleToMusicLocation,
      isShowPlayerDetail,
    }
  },
}
</script>


<style lang="less" module>
@import '@renderer/assets/styles/layout.less';

.player {
  position: relative;
  height: @height-player;
  box-sizing: border-box;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  contain: strict;
  padding: 0 20px 8px; // 增加底部 padding，让内容视觉上浮
  z-index: 10;
  
  // 稳重的厚面板风格
  backdrop-filter: blur(@ui-glass-blur);
  background-color: @ui-glass-background;
  border: @ui-glass-border;
  border-radius: @ui-radius-component;
  box-shadow: @ui-shadow-soft;
  transition: @transition-normal;
  transition-property: background-color, box-shadow;

  * {
    box-sizing: border-box;
  }

  &:before {
    display: none;
  }
}
.progress {
  position: absolute;
  top: -1px; // 稍微往上提
  left: 12px;
  right: 12px;
  padding-bottom: 6px;
  .progressBar {
    height: 3px; 
    border-radius: 4px;
    background-color: var(--color-primary-alpha-200);
  }
}

.picContent {
  height: 75%; // 在 76px 高度下展现更多细节
  aspect-ratio: 1 / 1;
  flex: none;
  transition: transform @transition-fast;
  display: flex;
  justify-content: center;
  cursor: pointer;
  margin-top: -4px; // 向上微调重心

  &:hover {
    transform: scale(1.05);
  }

  img {
    box-shadow: @ui-shadow-glass;
    max-width: 100%;
    max-height: 100%;
    border-radius: @ui-radius-component;
    object-fit: cover;
  }
}

.infoContent {
  padding-left: 20px; 
  flex: auto;
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: flex-start;
  min-width: 0;
  gap: 1px;
  margin-top: -6px; // 向上微调重心
}

.title {
  max-width: 100%;
  font-size: 14px; // 稍微下调至更平衡的值
  font-weight: @ui-font-weight-title;
  color: var(--color-font);
  .mixin-ellipsis-1();
}
.status {
  padding-top: 0px;
  font-size: 11px;
  color: var(--color-font-label);
  .mixin-ellipsis-1();
  max-width: 100%;
}

.playBtnContent {
  height: 100%;
  flex: none;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  padding-left: 10px;
  padding-right: 15px;
  gap: 20px; // 增加间距
  margin-top: -4px; // 向上微调重心
}

.playBtn {
  flex: none;
  height: 32px; // 强制固定高度，不再跟随百分比变大
  width: 32px;
  transition: @transition-fast;
  transition-property: color, opacity, transform;
  color: var(--color-button-font);
  opacity: 1;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;

  svg {
    fill: currentColor;
    filter: drop-shadow(0 0 1px rgba(0, 0, 0, 0.2));
  }
  
  // 针对中间主播放按钮的特殊样式（第二个子元素）
  &:nth-child(2) {
    height: 38px;
    width: 38px;
  }

  &:hover {
    opacity: 0.8;
    transform: scale(1.1);
  }
  &:active {
    opacity: 0.6;
    transform: scale(0.95);
  }
}

.timeContent {
  flex: none;
  color: var(--color-550);
  font-size: 13px;
  padding-left: 10px;
}

.playBtnContent {
  height: 100%;
  flex: none;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  padding-left: 10px;
  padding-right: 15px;
  gap: 18px;
}

.playBtn {
  flex: none;
  height: 52%;
  // margin-top: -2px;
  transition: @transition-fast;
  transition-property: color, opacity;
  color: var(--color-button-font);
  opacity: 1;
  cursor: pointer;

  svg {
    fill: currentColor;
    filter: drop-shadow(0 0 1px rgba(0, 0, 0, 0.2));
  }
  &:hover {
    opacity: 0.8;
  }
  &:active {
    opacity: 0.6;
  }
}

</style>
