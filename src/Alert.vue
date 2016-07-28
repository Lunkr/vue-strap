<template>
  <div
    v-show="show"
    v-bind:class="{
      'alert':    true,
      'alert-success':(type == 'success'),
      'alert-warning':(type == 'warning'),
      'alert-info': (type == 'info'),
      'alert-danger': (type == 'danger'),
      'top':      (placement === 'top'),
      'top-right':  (placement === 'top-right')
    }"
    transition="fade"
    v-bind:style="{width:width}"
    role="alert">
    <button v-show="dismissable" type="button" class="close"
      @click="show = false">
      <span>&times;</span>
    </button>
    {{{message}}}
  </div>
</template>

<script>
  import Vue from 'vue'
  import coerceBoolean from './utils/coerceBoolean.js'

  export default {
    props: {
      message: {
        type: String
      },
      type: {
        type: String
      },
      dismissable: {
        type: Boolean,
        coerce: coerceBoolean,
        default: false,
      },
      show: {
        type: Boolean,
        coerce: coerceBoolean,
        default: true,
        twoWay: true
      },
      duration: {
        type: Number,
        default: 0
      },
      width: {
        type: String
      },
      placement: {
        type: String
      },
      container: {
        type: String,
        default: '.vue-alert'
      },
    },
    data() {
      return {
        $_parent_: null,
      }
    },
    created () {
      let $parent = this.$parent
      if (!$parent) {
        let parent = document.querySelector(this.container)
        if (!parent) {
          parent = document.createElement('div')
          parent.classList.add(this.container.replace('.', ''))
          const Alert = Vue.extend()
          $parent = new Alert({
            el: parent
          }).$appendTo(document.body)
        }
        // Hacked.
        this.$_parent_ = parent.__vue__
      }
    },
    compiled () {
      if (this.$_parent_) {
        this.$appendTo(this.$_parent_.$el)
        delete this.$_parent_
      }
    },
    ready () {
      if (this.duration > 0) {
        this._timeout = setTimeout(() => this.close(), this.duration)
      }
    },
    methods: {
      close() {
        clearTimeout(this._timeout)
        this.$destroy(true)
      }
    }
  }
</script>

<style>
.fade-transition {
  transition: opacity .3s ease;
}
.fade-enter,
.fade-leave {
  height: 0;
  opacity: 0;
}
.alert.top {
  position: fixed;
  top: 30px;
  margin: 0 auto;
  left: 0;
  right: 0;
  z-index: 2;
}
.alert.top-right {
  position: fixed;
  top: 30px;
  right: 50px;
  z-index: 2;
}
</style>
