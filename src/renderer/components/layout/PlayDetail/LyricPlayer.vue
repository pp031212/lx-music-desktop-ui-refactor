<template>
  <div :class="['right', $style.right]" :style="lrcFontSize">
    <transition enter-active-class="animated fadeIn" leave-active-class="animated fadeOut">
      <div
        v-show="!isShowLrcSelectContent"
        ref="dom_lyric"
        :class="['lyric', $style.lyric, { [$style.draging]: isMsDown }, { [$style.lrcActiveZoom]: isZoomActiveLrc }, { [$style.fontWeight]: isFontWeight }]" :style="lrcStyles"
        @wheel="handleWheel" @mousedown="handleLyricMouseDown" @touchstart="handleLyricTouchStart"
        @contextmenu.stop="handleShowLyricMenu"
      >
        <div :class="['pre', $style.lyricSpace]" />
        <div ref="dom_lyric_text" />
        <div :class="$style.lyricSpace" />
      </div>
    </transition>
    <transition enter-active-class="animated fadeIn" leave-active-class="animated fadeOut">
      <div v-if="isShowLyricProgressSetting" v-show="isStopScroll && !isShowLrcSelectContent" :class="$style.skip">
        <div ref="dom_skip_line" :class="$style.line" />
        <span :class="$style.label">{{ timeStr }}</span>
        <base-btn :class="$style.skipBtn" @mouseenter="handleSkipMouseEnter" @mouseleave="handleSkipMouseLeave" @click="handleSkipPlay">
          <svg version="1.1" xmlns="http://www.w3.org/2000/svg" xlink="http://www.w3.org/1999/xlink" height="50%" viewBox="0 0 1024 1024" space="preserve">
            <use xlink:href="#icon-play" />
          </svg>
        </base-btn>
      </div>
    </transition>
    <transition enter-active-class="animated fadeIn" leave-active-class="animated fadeOut">
      <div v-if="isShowLrcSelectContent" ref="dom_lrc_select_content" tabindex="-1" :class="[$style.lyricSelectContent, 'select', 'scroll', 'lyricSelectContent', { [$style.fontWeight]: isFontWeight }]" @contextmenu="handleCopySelectText">
        <div v-for="(info, index) in lyric.lines" :key="index" :class="[$style.lyricSelectline, { [$style.lrcActive]: lyric.line == index }]">
          <span>{{ info.text }}</span>
          <template v-for="(lrc, i) in info.extendedLyrics" :key="i">
            <br>
            <span :class="$style.lyricSelectlineExtended">{{ lrc }}</span>
          </template>
        </div>
      </div>
    </transition>
    <LyricMenu v-model="lyricMenuVisible" :xy="lyricMenuXY" :lyric-info="lyricInfo" @update-lyric="handleUpdateLyric" />
  </div>
</template>

<script>
import { clipboardWriteText } from '@common/utils/electron'
import { lyric } from '@renderer/store/player/lyric'
import { playProgress } from '@renderer/store/player/playProgress'
import { isFullscreen } from '@renderer/store'
import {
  isPlay,
  isShowLrcSelectContent,
  isShowPlayComment,
  musicInfo as playerMusicInfo,
  playMusicInfo,
} from '@renderer/store/player/state'
import {
  setMusicInfo,
} from '@renderer/store/player/action'
import { onMounted, onBeforeUnmount, computed, reactive, ref, nextTick, watch } from '@common/utils/vueTools'
import useLyric from '@renderer/utils/compositions/useLyric'
import LyricMenu from './components/LyricMenu.vue'
import { appSetting } from '@renderer/store/setting'
import { setLyricOffset } from '@renderer/core/lyric'
import useSelectAllLrc from './useSelectAllLrc'

export default {
  components: {
    LyricMenu,
  },
  setup() {
    const isZoomActiveLrc = computed(() => appSetting['playDetail.isZoomActiveLrc'])
    const isShowLyricProgressSetting = computed(() => appSetting['playDetail.isShowLyricProgressSetting'])
    const isFontWeight = computed(() => appSetting['playDetail.style.fontWeight'])

    const {
      dom_lyric,
      dom_lyric_text,
      dom_skip_line,
      isMsDown,
      isStopScroll,
      timeStr,
      handleLyricMouseDown,
      handleLyricTouchStart,
      handleWheel,
      handleSkipPlay,
      handleSkipMouseEnter,
      handleSkipMouseLeave,
      handleScrollLrc,
    } = useLyric({ isPlay, lyric, playProgress, isShowLyricProgressSetting })

    const dom_lrc_select_content = useSelectAllLrc()

    watch([isFullscreen, isShowPlayComment], () => {
      setTimeout(handleScrollLrc, 400)
    })

    const lyricMenuVisible = ref(false)
    const lyricMenuXY = reactive({
      x: 0,
      y: 0,
    })
    const lyricInfo = reactive({
      lyric: '',
      tlyric: '',
      rlyric: '',
      lxlyric: '',
      rawlyric: '',
      musicInfo: null,
    })
    const updateMusicInfo = () => {
      lyricInfo.lyric = playerMusicInfo.lrc
      lyricInfo.tlyric = playerMusicInfo.tlrc
      lyricInfo.rlyric = playerMusicInfo.rlrc
      lyricInfo.lxlyric = playerMusicInfo.lxlrc
      lyricInfo.rawlyric = playerMusicInfo.rawlrc
      lyricInfo.musicInfo = playMusicInfo.musicInfo
    }
    const handleShowLyricMenu = event => {
      updateMusicInfo()
      lyricMenuXY.x = event.pageX
      lyricMenuXY.y = event.pageY
      if (lyricMenuVisible.value) return
      void nextTick(() => {
        lyricMenuVisible.value = true
      })
    }
    const handleUpdateLyric = ({ lyric, tlyric, rlyric, lxlyric, offset }) => {
      setMusicInfo({
        lrc: lyric,
        tlrc: tlyric,
        rlrc: rlyric,
        lxlrc: lxlyric,
      })
      console.log(offset)
      setLyricOffset(offset)
    }

    const lrcStyles = computed(() => {
      const align = appSetting['playDetail.style.align']
      const activePaddingLeft = align == 'right'
        ? 'calc(8% + 14px)'
        : align == 'center'
          ? 'calc(8% + 14px)'
          : '1px'
      const activePaddingRight = align == 'left'
        ? 'calc(8% + 14px)'
        : align == 'center'
          ? 'calc(8% + 14px)'
          : '8%'
      return {
        textAlign: align,
        '--playDetail-lrc-transform-origin': align == 'left'
          ? 'left center'
          : align == 'right'
            ? 'right center'
            : 'center center',
        '--playDetail-lrc-active-padding-left': activePaddingLeft,
        '--playDetail-lrc-active-padding-right': activePaddingRight,
      }
    })
    const lrcFontSize = computed(() => {
      let size = appSetting['playDetail.style.fontSize'] / 100
      if (isFullscreen.value) size = size *= 1.4
      return {
        '--playDetail-lrc-font-size': (isShowPlayComment.value ? size * 0.82 : size) + 'rem',
      }
    })

    onMounted(() => {
      window.app_event.on('musicToggled', updateMusicInfo)
      window.app_event.on('lyricUpdated', updateMusicInfo)
    })
    onBeforeUnmount(() => {
      window.app_event.off('musicToggled', updateMusicInfo)
      window.app_event.off('lyricUpdated', updateMusicInfo)
    })

    return {
      dom_lyric,
      dom_lyric_text,
      dom_skip_line,
      dom_lrc_select_content,
      isMsDown,
      timeStr,
      handleLyricMouseDown,
      handleLyricTouchStart,
      handleWheel,
      handleSkipPlay,
      handleSkipMouseEnter,
      handleSkipMouseLeave,
      lyric,
      lrcStyles,
      lrcFontSize,
      isShowLrcSelectContent,
      isShowLyricProgressSetting,
      isZoomActiveLrc,
      isFontWeight,
      isStopScroll,
      lyricMenuVisible,
      lyricMenuXY,
      handleShowLyricMenu,
      handleUpdateLyric,
      lyricInfo,
    }
  },
  methods: {
    handleCopySelectText() {
      let str = window.getSelection().toString()
      str = str.trim()
      if (!str.length) return
      clipboardWriteText(str)
    },
  },
}
</script>


<style lang="less" module>
@import '@renderer/assets/styles/layout.less';

.right {
  flex: auto;
  position: relative;
  min-width: 0;
  padding: 8px 0 0;
  transition: width @transition-normal;
}
.lyric {
  position: relative;
  height: 100%;
  overflow: hidden;
  font-size: var(--playDetail-lrc-font-size, 16px);
  -webkit-mask-image: linear-gradient(transparent 0%, #fff 20%,  #fff 80%, transparent 100%);
  mask-image: linear-gradient(transparent 0%, #fff 20%, #fff 80%, transparent 100%);
  cursor: grab;
  &.draging {
    cursor: grabbing;
  }
  :global {
    .font-lrc {
      color: var(--color-350);
    }
    .line-content {
      line-height: 1.2;
      padding: calc(var(--playDetail-lrc-font-size, 16px) / 1.85) 8% calc(var(--playDetail-lrc-font-size, 16px) / 1.85) 1px;
      overflow-wrap: break-word;
      color: var(--color-350);
      text-shadow:
        0 0 2px var(--color-primary-light-100-alpha-900),
        0 0 3px var(--color-primary-light-100-alpha-900),
        0 0 4px var(--color-primary-dark-700-alpha-900);
      transition: @transition-slow !important;
      transition-property: padding, transform !important;

      &.active {
        padding-left: var(--playDetail-lrc-active-padding-left, calc(8% + 14px));
        padding-right: var(--playDetail-lrc-active-padding-right, calc(8% + 14px));
        padding-top: calc(var(--playDetail-lrc-font-size, 16px) * 1.15);
        padding-bottom: calc(var(--playDetail-lrc-font-size, 16px) * 1.15);
      }

      .extended {
        font-size: 0.8em;
        margin-top: 5px;
      }
      &.line-mode {
        .font-lrc {
          transition: @transition-normal;
          transition-property: color;
        }
      }
      &.line-mode.active .font-lrc, &.font-mode.played .font-lrc {
        color: var(--color-primary-dark-200);
      }
      &.font-mode .extended .font-lrc {
        transition: @transition-slow;
        transition-property: font-size, color;
      }

      &.font-mode > .line > .font-lrc {
        > span {
          transition: @transition-normal;
          transition-property: font-size;
          font-size: 1em;
          background-image: -webkit-linear-gradient(top, var(--color-primary-dark-200), var(--color-primary-dark-200));
          background-repeat: no-repeat;
          background-color: var(--color-300);
          -webkit-text-fill-color: transparent;
          -webkit-background-clip: text;
          background-size: 0 100%;
        }
      }
    }
  }
  // p {
  //   padding: 8px 0;
  //   line-height: 1.2;
  //   overflow-wrap: break-word;
  //   transition: @transition-normal !important;
  //   transition-property: color, font-size;
  // }
  // .lrc-active {
  //   color: var(--color-primary);
  //   font-size: 1.2em;
  // }
}
.lrcActiveZoom {
  :global {
    .line-content {
      &.active {
        transform-origin: var(--playDetail-lrc-transform-origin, center center);
        transform: scale(1.08);
      }
    }
  }
}

.fontWeight {
  :global {
    .line-content {
      font-weight: 700;
    }
    .line-content .extended {
      font-weight: 700;
    }
  }

  .lyricSelectline,
  .lyricSelectlineExtended {
    font-weight: 700;
  }
}

.skip {
  position: absolute;
  top: calc(38% + var(--playDetail-lrc-font-size, 16px) + 4px);
  left: 0;
  // height: 6px;
  width: 100%;
  pointer-events: none;
  // opacity: .5;
  .line {
    border-top: 2px dotted var(--color-primary-dark-100);
    opacity: .15;
    margin-right: 8%;
    -webkit-mask-image: linear-gradient(90deg, transparent 0%, transparent 15%, #fff 100%);
    mask-image: linear-gradient(90deg, transparent 0%, transparent 15%, #fff 100%);
  }
  .label {
    position: absolute;
    right: 8%;
    top: -16px;
    line-height: 1.2;
    font-size: 13px;
    color: var(--color-primary-dark-100);
    opacity: .7;
  }
  .skipBtn {
    position: absolute;
    right: -2%;
    top: 0;
    transform: translateY(-50%);
    width: 12%;
    aspect-ratio: 1 / 1;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: none !important;
    pointer-events: initial;
    transition: @transition-normal;
    transition-property: opacity;
    opacity: .8;
    &:hover {
      opacity: .6;
    }
  }
}
.lyricSelectContent {
  position: absolute;
  left: 0;
  top: 0;
  // text-align: center;
  height: 100%;
  width: 100%;
  font-size: var(--playDetail-lrc-font-size, 16px);
  z-index: 10;
  color: var(--color-350);

  .lyricSelectline {
    padding: calc(var(--playDetail-lrc-font-size, 16px) / 1.85) 6% calc(var(--playDetail-lrc-font-size, 16px) / 1.85) 1px;
    overflow-wrap: break-word;
    transition: @transition-normal !important;
    transition-property: color, font-size;
    line-height: 1.3;
  }
  .lyricSelectlineExtended {
    font-size: 14px;
  }
  .lrcActive {
    color: var(--color-primary);
  }
}

.lyricSpace {
  height: 60%;
}

</style>
