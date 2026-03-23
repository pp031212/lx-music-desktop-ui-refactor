<template lang="pug">
transition(enter-active-class="animated slideInRight" leave-active-class="animated slideOutDown" @after-enter="handleAfterEnter" @after-leave="handleAfterLeave")
  div(v-if="isShowPlayerDetail" :class="[$style.container, { fullscreen: isFullscreen }]" @contextmenu="handleContextMenu")
    div(:class="$style.bg")
    ControlBtnsLeftHeader(v-if="appSetting['common.controlBtnPosition'] == 'left'")
    ControlBtnsRightHeader(v-else)
    div(:class="[$style.main, {[$style.showComment]: isShowPlayComment}]")
      section.left(:class="$style.leftPanel")
          div(:class="$style.mediaCard")
          div(:class="[$style.coverWrap, { [$style.coverPlaying]: isPlay }]")
            span(:class="$style.coverDotTop")
            span(:class="$style.coverDotBottom")
            svg(:class="$style.cd" viewBox="0 0 100 100" aria-hidden="true")
              defs
                mask(id="play-detail-cd-hole")
                  rect(width="100" height="100" fill="white")
                  circle(cx="50" cy="50" r="11.6" fill="black")
              circle(cx="50" cy="50" r="50" fill="var(--color-primary-light-400)" mask="url(#play-detail-cd-hole)")
              foreignObject(x="2" y="2" width="96" height="96" mask="url(#play-detail-cd-hole)")
                img(v-if="musicInfo.pic" :class="$style.img" :src="musicInfo.pic")
                div(v-else :class="$style.coverPlaceholder")
                  svg(version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24")
                    use(xlink:href="#icon-musicFile")
              circle(
                cx="50"
                cy="50"
                r="20"
                fill="var(--color-primary-light-300-alpha-600)"
                mask="url(#play-detail-cd-hole)"
                style="mix-blend-mode: multiply;"
              )
              circle(
                cx="50"
                cy="50"
                r="11"
                fill="none"
                stroke="var(--color-primary-light-300)"
                stroke-width="1.2"
                style="mix-blend-mode: exclusion;"
              )
              circle(cx="50" cy="50" r="11.6" fill="none" stroke="var(--color-primary-light-300)" stroke-width="0.4")
          div(:class="$style.description")
            p.metaTitle(:class="$style.metaTitle")
              span(:ref="setMetaViewportRef('title')" :class="$style.metaViewport")
                span(
                  :class="[$style.metaTrack, { [$style.metaTrackAuto]: metaScrollState.title.enabled, [$style.metaTextMasked]: metaScrollState.title.enabled }]"
                  :style="metaScrollState.title.style"
                )
                  span(
                    :ref="setMetaTextRef('title')"
                    :class="$style.metaText"
                  ) {{ musicInfo.name }}
                  span(v-if="metaScrollState.title.enabled" :class="[$style.metaText, $style.metaTextDuplicate]" aria-hidden="true") {{ musicInfo.name }}
            p(:class="$style.metaLine")
              span(:class="$style.metaLabel") {{ $t('player__music_singer') }}
              span(:ref="setMetaViewportRef('singer')" :class="[$style.metaViewport, $style.metaValue]")
                span(
                  :class="[$style.metaTrack, { [$style.metaTrackAuto]: metaScrollState.singer.enabled, [$style.metaTextMasked]: metaScrollState.singer.enabled }]"
                  :style="metaScrollState.singer.style"
                )
                  span(
                    :ref="setMetaTextRef('singer')"
                    :class="$style.metaText"
                  ) {{ musicInfo.singer }}
                  span(v-if="metaScrollState.singer.enabled" :class="[$style.metaText, $style.metaTextDuplicate]" aria-hidden="true") {{ musicInfo.singer }}
            p(v-if="musicInfo.album" :class="$style.metaLine")
              span(:class="$style.metaLabel") {{ $t('player__music_album') }}
              span(:ref="setMetaViewportRef('album')" :class="[$style.metaViewport, $style.metaValue]")
                span(
                  :class="[$style.metaTrack, { [$style.metaTrackAuto]: metaScrollState.album.enabled, [$style.metaTextMasked]: metaScrollState.album.enabled }]"
                  :style="metaScrollState.album.style"
                )
                  span(
                    :ref="setMetaTextRef('album')"
                    :class="$style.metaText"
                  ) {{ musicInfo.album }}
                  span(v-if="metaScrollState.album.enabled" :class="[$style.metaText, $style.metaTextDuplicate]" aria-hidden="true") {{ musicInfo.album }}

      transition(enter-active-class="animated fadeIn" leave-active-class="animated fadeOut")
        LyricPlayer(v-if="visibled")
      music-comment(v-if="visibled" :class="$style.comment" :show="isShowPlayComment" :music-info="playMusicInfo.musicInfo" @close="hideComment")
    transition(enter-active-class="animated fadeIn" leave-active-class="animated fadeOut")
      play-bar(v-if="visibled")
    transition(enter-active-class="animated-slow fadeIn" leave-active-class="animated-slow fadeOut")
      common-audio-visualizer(v-if="appSetting['player.audioVisualization'] && visibled")
</template>


<script>
import { ref, watch, reactive, nextTick, onMounted, onBeforeUnmount } from '@common/utils/vueTools'
import { isFullscreen } from '@renderer/store'
import {
  isPlay,
  isShowPlayerDetail,
  isShowPlayComment,
  musicInfo,
  playMusicInfo,
} from '@renderer/store/player/state'
import {
  setShowPlayerDetail,
  setShowPlayComment,
  setShowPlayLrcSelectContentLrc,
} from '@renderer/store/player/action'
import LyricPlayer from './LyricPlayer.vue'
import PlayBar from './PlayBar.vue'
import MusicComment from './components/MusicComment/index.vue'
import ControlBtnsLeftHeader from './ControlBtnsLeftHeader.vue'
import ControlBtnsRightHeader from './ControlBtnsRightHeader.vue'
import { registerAutoHideMounse, unregisterAutoHideMounse } from './autoHideMounse'
import { appSetting } from '@renderer/store/setting'
import { closeWindow, maxWindow, minWindow, setFullScreen } from '@renderer/utils/ipc'

export default {
  name: 'CorePlayDetail',
  components: {
    ControlBtnsLeftHeader,
    ControlBtnsRightHeader,
    LyricPlayer,
    PlayBar,
    MusicComment,
  },
  setup() {
    const visibled = ref(false)
    const metaViewportRefs = reactive({
      title: null,
      singer: null,
      album: null,
    })
    const metaTextRefs = reactive({
      title: null,
      singer: null,
      album: null,
    })
    const metaScrollState = reactive({
      title: { enabled: false, style: {} },
      singer: { enabled: false, style: {} },
      album: { enabled: false, style: {} },
    })

    let clickTime = 0

    const setMetaViewportRef = key => el => {
      metaViewportRefs[key] = el
    }
    const setMetaTextRef = key => el => {
      metaTextRefs[key] = el
    }
    const updateMetaScrollState = () => {
      for (const key of Object.keys(metaScrollState)) {
        const viewport = metaViewportRefs[key]
        const text = metaTextRefs[key]
        if (!viewport || !text) continue
        const distance = Math.max(0, Math.ceil(text.scrollWidth - viewport.clientWidth))
        if (!distance) {
          metaScrollState[key].enabled = false
          metaScrollState[key].style = {}
          continue
        }
        const gap = Math.max(36, Math.round(viewport.clientWidth * 0.12))
        metaScrollState[key].enabled = true
        metaScrollState[key].style = {
          '--meta-scroll-gap': `${gap}px`,
          '--meta-scroll-distance': `${Math.ceil(text.scrollWidth + gap)}px`,
          '--meta-scroll-duration': `${Math.max(10, Math.round((text.scrollWidth + gap) / 18))}s`,
        }
      }
    }
    const syncMetaScrollState = () => {
      void nextTick(updateMetaScrollState)
    }

    const hide = () => {
      setShowPlayerDetail(false)
    }
    const handleContextMenu = () => {
      if (window.performance.now() - clickTime > 400) {
        clickTime = window.performance.now()
        return
      }
      clickTime = 0
      hide()
    }

    const hideComment = () => {
      setShowPlayComment(false)
    }

    const handleAfterEnter = () => {
      if (isFullscreen.value) registerAutoHideMounse()

      visibled.value = true
    }

    const handleAfterLeave = () => {
      setShowPlayLrcSelectContentLrc(false)
      hideComment(false)
      visibled.value = false

      unregisterAutoHideMounse()
    }

    watch(isFullscreen, isFullscreen => {
      (isFullscreen ? registerAutoHideMounse : unregisterAutoHideMounse)()
    })
    watch(
      [
        isShowPlayerDetail,
        isShowPlayComment,
        () => musicInfo.name,
        () => musicInfo.singer,
        () => musicInfo.album,
      ],
      syncMetaScrollState,
      { flush: 'post' },
    )

    onMounted(() => {
      window.addEventListener('resize', syncMetaScrollState)
      syncMetaScrollState()
    })
    onBeforeUnmount(() => {
      window.removeEventListener('resize', syncMetaScrollState)
    })


    return {
      appSetting,
      isPlay,
      playMusicInfo,
      isShowPlayerDetail,
      isShowPlayComment,
      musicInfo,
      hide,
      handleContextMenu,
      hideComment,
      handleAfterEnter,
      handleAfterLeave,
      visibled,
      isFullscreen,
      metaScrollState,
      setMetaViewportRef,
      setMetaTextRef,
      fullscreenExit() {
        void setFullScreen(false).then((fullscreen) => {
          isFullscreen.value = fullscreen
        })
      },
      min() {
        minWindow()
      },
      max() {
        maxWindow()
      },
      close() {
        closeWindow()
      },
    }
  },
}
</script>


<style lang="less" module>
@import '@renderer/assets/styles/layout.less';

@control-btn-width: @height-toolbar * .26;

.container {
  position: absolute;
  display: flex;
  flex-flow: column nowrap;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background-color: var(--color-content-background);
  z-index: 10;
  // -webkit-app-region: drag;
  overflow: hidden;
  border-radius: @radius-border;
  color: var(--color-font);
  -webkit-app-region: no-drag;
  contain: strict;

  box-sizing: border-box;

  * {
    box-sizing: border-box;
  }
}
.bg {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background: var(--background-image) var(--background-image-position) no-repeat;
  background-size: var(--background-image-size);
  // filter: blur(60px);
  opacity: .7;
  z-index: -1;
  &:before {
    content: '';
    display: block;
    width: 100%;
    height: 100%;
    background-color: var(--color-app-background);
  }
  &:after {
    position: absolute;
    left: 0;
    top: 0;
    content: '';
    display: block;
    width: 100%;
    height: 100%;
    background:
      radial-gradient(circle at top left, var(--color-primary-light-100-alpha-500), transparent 34%),
      radial-gradient(circle at 82% 18%, var(--color-primary-alpha-500), transparent 26%),
      linear-gradient(135deg, var(--color-main-background), var(--color-content-background));
  }
}

.main {
  flex: auto;
  min-height: 0;
  overflow: hidden;
  display: flex;
  gap: 26px;
  margin: 0 30px;
  padding: 8px 0 0;
  position: relative;
  transition: padding-right @transition-normal;

  &.showComment {
    padding-right: calc(46% + 12px);

    .leftPanel {
      width: 26%;
      min-width: 240px;
    }
    .metaTitle {
      font-size: 23px;
    }
    :global(.right) {
      flex: auto;
      width: auto;
      min-width: 0;
      .lyricSelectContent {
        font-size: 14px;
      }
    }
    .comment {
      opacity: 1;
      transform: scaleX(1);
    }
  }
}
.leftPanel {
  flex: none;
  width: 42%;
  min-width: 300px;
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  justify-content: center;
  padding: 18px 12px 18px 0;
  overflow: hidden;
  transition: width @transition-normal;
}

.mediaCard {
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: center;
  width: min(100%, 460px);
  min-height: 0;
  padding: 12px 18px 12px;
  border-radius: 0;
  background: transparent;
  box-shadow: none;
}

.coverWrap {
  --cover-size: clamp(248px, 30vw, 360px);
  position: relative;
  width: min(100%, var(--cover-size));
  aspect-ratio: 1 / 1;
  flex: none;
  padding: 5%;
  contain: strict;
  background-color: var(--color-primary-light-300-alpha-800);
  border-radius: 8px;
  box-shadow: 0 0 6px var(--color-primary-alpha-500);
  opacity: .85;
  backdrop-filter: blur(4px);
}

.coverDotTop,
.coverDotBottom {
  &::before,
  &::after {
    content: '';
    position: absolute;
    width: 7%;
    aspect-ratio: 1 / 1;
    background-color: var(--color-primary-light-300-alpha-800);
    border-radius: 50%;
    box-shadow: inset 0 0 4px var(--color-primary-dark-300-alpha-800);
  }
}

.coverDotTop {
  &::before {
    top: 5%;
    left: 5%;
  }
  &::after {
    top: 5%;
    right: 5%;
  }
}

.coverDotBottom {
  &::before {
    left: 5%;
    bottom: 5%;
  }
  &::after {
    right: 5%;
    bottom: 5%;
  }
}

.cd {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  box-shadow: 0 0 4px var(--color-primary-alpha-400);
  animation: coverSpin 120s linear infinite;
  animation-play-state: paused;
}

.img,
.coverPlaceholder {
  display: block;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
  box-shadow: none;
}

.coverPlaceholder {
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-font-label);
  background:
    radial-gradient(circle at 30% 30%, var(--color-primary-light-100-alpha-500), transparent 35%),
    linear-gradient(135deg, var(--color-primary-dark-300-alpha-500), var(--color-primary-light-300-alpha-300));

  svg {
    width: 30%;
    height: 30%;
    opacity: .7;
    fill: currentColor;
  }
}

.coverPlaying {
  .cd {
    animation-play-state: running;
  }
}

.description {
  width: 100%;
  margin-top: 22px;
  padding: 0 6px 8px;
  display: flex;
  flex-flow: column nowrap;
  gap: 10px;
  overflow: hidden;
}

.metaTitle {
  margin: 0;
  font-size: 28px;
  line-height: 1.2;
  font-weight: 600;
  color: var(--color-font);
}

.metaLine {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 0;
  line-height: 1.45;
  font-size: 16px;
  color: var(--color-font);
}

.metaLabel {
  flex: none;
  font-size: 13px;
  color: var(--color-font-label);
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.metaValue {
  flex: auto;
  min-width: 0;
}

.metaViewport {
  position: relative;
  display: block;
  width: 100%;
  overflow: hidden;
  white-space: nowrap;
}

.metaText {
  display: inline-block;
  min-width: max-content;
  white-space: nowrap;
}

.metaTextMasked {
  mask-image: linear-gradient(90deg, transparent 0, #fff 14px, #fff calc(100% - 14px), transparent 100%);
  -webkit-mask-image: linear-gradient(90deg, transparent 0, #fff 14px, #fff calc(100% - 14px), transparent 100%);
}

.metaTrack {
  display: inline-flex;
  align-items: center;
  min-width: max-content;
}

.metaTrackAuto {
  animation: metaTicker var(--meta-scroll-duration, 10s) linear infinite;
}

.metaTextDuplicate {
  padding-left: var(--meta-scroll-gap, 40px);
}


.comment {
  position: absolute;
  right: 0;
  top: 0;
  width: 46%;
  height: 100%;
  opacity: 0;
  margin-left: 12px;
  transform: scaleX(0);
  transform-origin: 100% 50%;
  transition: transform @transition-normal, opacity @transition-normal;
}

@keyframes coverSpin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes metaTicker {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(calc(var(--meta-scroll-distance, 0px) * -1));
  }
}


</style>
