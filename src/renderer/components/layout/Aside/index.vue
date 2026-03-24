<template>
  <div :class="[$style.aside, { [$style.fullscreen]: isFullscreen }]">
    <ControlBtns v-if="appSetting['common.controlBtnPosition'] == 'left'" />
    <div v-else :class="$style.logo">L X</div>
    <NavBar />
  </div>
</template>

<script setup>
import { isFullscreen } from '@renderer/store'
import { appSetting } from '@renderer/store/setting'

import ControlBtns from './ControlBtns.vue'
import NavBar from './NavBar.vue'

</script>


<style lang="less" module>
@import '@renderer/assets/styles/layout.less';

.aside {
  transition: @transition-normal;
  transition-property: background-color, transform;
  backdrop-filter: blur(@ui-glass-blur);
  background-color: @ui-glass-background;
  border-right: @ui-glass-border;
  -webkit-app-region: drag;
  -webkit-user-select: none;
  display: flex;
  flex-flow: column nowrap;

  // 紧凑化
  border-radius: @ui-radius-component; // 减小圆角
  box-shadow: @ui-shadow-soft;
  z-index: 10;

  &.fullscreen {
    margin: 0;
    border-radius: 0;
    -webkit-app-region: no-drag;
    .logo {
      display: none;
    }
  }
}

.logo {
  box-sizing: border-box;
  padding: 0 13%;
  height: 50px; // 恢复到 50px，更紧凑
  color: var(--color-nav-font);
  opacity: .9;
  flex: none;
  text-align: center;
  line-height: 50px;
  font-weight: bold;
  font-size: 1.1rem;
  letter-spacing: 0.1em;
}


</style>
