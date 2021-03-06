<template>
  <div class="t-dropdown" @click="onClick" @mouseleave="onMouseLeave" @mouseenter="onMouseOver">
    <slot></slot>
    <t-dropmenu
      :parent="self"
      :isOpen="isOpen"
      :initial="initial"
      :width="width"
      :minWidth="minWidth"
      :textCenter="textCenter"
      :side="side"
      @mouseleave="onMouseLeave"
      @mouseenter="onMouseOver"
      @subopen="onSubOpen"
      @subclose="onSubClose"
      @command="handleCommand"
      @subcommand="handleSubCommand"
    >
      <slot name="dropdown"/>
    </t-dropmenu>
  </div>
</template>
<script>
//  TODO: open/close main menu /sub menu with focusIndex
import TDropmenu from './dropmenu.vue'
import Emitter from '../../mixins/emitter'

export default {
  components: {
    TDropmenu
  },

  name: 't-dropdown',

  mixins: [Emitter],

  data () {
    return {
      self: this,
      initial: false,
      isOpen: false,
      closeTimer: null,
      forceCheck: false,
      isHover: false
    }
  },

  props: {
    width: Number,
    minWidth: Number,
    maxHeight: Number,
    hideOnClick: Boolean,
    textCenter: {
      type: Boolean,
      default: true
    },
    trigger: {
      type: String,
      default: 'hover'
    },
    value: {},
    side: Boolean,
    hasParentMenu: Boolean,
    sub: Boolean
  },

  created () {
    this.$on('drop-focus', this.onMouseEnter)
    this.$on('drop-blur', this.onMouseLeave)
    this.$on('on-sub-command', this.handleSubCommand)
  },

  methods: {
    onSubClose () {
      !this.isHover && this.close()
    },
    onSubOpen () {
      this.open()
    },
    onMouseOver () {
      this.isHover = true
      if (!this.forceCheck && this.trigger === 'hover') {
        this.open()
        this.sub && this.dispatch('t-dropdown-menu', 'on-sub-open')
      }
    },
    onMouseLeave () {
      this.isHover = false
      if (!this.forceCheck && this.trigger === 'hover') {
        this.close()
      }
    },
    onClick () {
      if (!this.forceCheck && this.trigger === 'click') {
        this.checkout()
      }
    },
    open () {
      clearTimeout(this.closeTimer)
      !this.initial && (this.initial = true)
      this.isOpen = true
    },
    close () {
      this.closeTimer = setTimeout(() => {
        this.isOpen = false
        this.sub && this.dispatch('t-dropdown-menu', 'on-sub-close')
      }, 100)
    },
    checkout () {
      this.isOpen ? this.close() : this.open()
    },
    handleCommand (cmd) {
      if (this.hasParentMenu) {
        this.dispatch('t-dropdown-menu', 'sub-close', {cmd: cmd, isClose: this.hideOnClick})
      } else {
        this.$emit('command', cmd)
      }

      if (this.hideOnClick) {
        this.forceCheck = true
        this.isOpen = false
        setTimeout(() => {
          this.forceCheck = false
        })
      }
    },
    handleSubCommand ({cmd, isClose}) {
      if (this.hasParentMenu) {
        this.dispatch('t-dropdown-menu', 'sub-close', {cmd: cmd, isClose: isClose})
      } else {
        this.$emit('command', cmd)
      }

      if (isClose) {
        this.forceCheck = true
        this.isOpen = false
        setTimeout(() => {
          this.forceCheck = false
        })
      }
    }
  }
}
</script>
