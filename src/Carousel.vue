<template>
  <div class="carousel slide" data-ride="carousel">
    <!-- Indicators -->
    <ol class="carousel-indicators" v-show="indicators && !(hideIndicatorsAtLast && lastSlider)">
      <indicator :indicator.sync="indicator" :active-index.sync="activeIndex" :is-animating.sync="isAnimating"></indicator>
    </ol>
    <!-- Wrapper for slides -->
    <div class="carousel-inner" role="listbox">
      <slot></slot>
    </div>
    <!-- Controls -->
    <div v-show="controls" class="carousel-controls hidden-xs">
      <a class="left carousel-control" role="button" @click="prevClick" v-show="activeIndex">
        <span class="lunkrfont lunkr-lastscene3x" aria-hidden="true"></span>
      </a>
      <a class="right carousel-control" role="button" @click="nextClick" v-show="!lastSlider">
        <span class="lunkrfont lunkr-nextscene3x" aria-hidden="true"></span>
      </a>
    </div>
  </div>
</template>

<script>
  import EventListener from './utils/EventListener.js'
  import coerceBoolean from './utils/coerceBoolean.js'

  export default {
    props: {
      indicators: {
        type: Boolean,
        coerce: coerceBoolean,
        default: true
      },
      controls: {
        type: Boolean,
        coerce: coerceBoolean,
        default: true
      },
      interval: {
        type: Number,
        default: 5000
      },
      hideIndicatorsAtLast: {
        type: Boolean,
        coerce: coerceBoolean,
        default: false
      }
    },
    components: {
      'indicator': {
        //inherit: true,
        props: ['indicator', 'activeIndex', 'isAnimating'],
        template: '<li v-for="i in indicator" @click="handleIndicatorClick($index)" v-bind:class="{\'active\':$index === activeIndex}"><span></span></li>',
        methods: {
          handleIndicatorClick(index) {
            if (this.isAnimating || this.activeIndex === index) return false
            this.isAnimating = true
            this.activeIndex = index
          }
        },
      }
    },
    data() {
      return {
        indicator: [],
        activeIndex: 0,
        isAnimating: false
      }
    },
    computed: {
      slider() {
        return this.$el.querySelectorAll('.item')
      },
      lastSlider() {
        return this.activeIndex === this.slider.length - 1
      }
    },
    watch: {
      activeIndex(newVal, oldVal) {
        newVal > oldVal ? this.slide('left', newVal, oldVal) : this.slide('right', newVal, oldVal)
      }
    },
    methods: {
      slide(direction, selected, prev) {
        if (this._prevSelectedEvent) this._prevSelectedEvent.remove()
        if (this._selectedEvent) this._selectedEvent.remove()

        const prevSelectedEl = this.slider[prev]
        const selectedEl = this.slider[selected]
        const transitionendFn = ()=> {
          [...this.slider].forEach((el)=> el.className = 'item')
          selectedEl.classList.add('active')
          this.isAnimating = false
        }

        direction === 'left' ? selectedEl.classList.add('next') : selectedEl.classList.add('prev')
        // request property that requires layout to force a layout
        var x = selectedEl.clientHeight
        this._prevSelectedEvent = EventListener.listen(prevSelectedEl, 'transitionend', transitionendFn)
        this._selectedEvent = EventListener.listen(selectedEl, 'transitionend', transitionendFn)
        prevSelectedEl.classList.add(direction)
        selectedEl.classList.add(direction)
      },
      nextClick() {
        if (this.isAnimating) return false
        this.isAnimating = true
        this.activeIndex + 1 < this.slider.length ? this.activeIndex += 1 : this.activeIndex = 0
      },
      prevClick() {
        if (this.isAnimating) return false
        this.isAnimating = true
        this.activeIndex === 0 ? this.activeIndex = this.slider.length - 1 : this.activeIndex -= 1
      }
    },
    ready() {
      let intervalID = null
      const el = this.$el
      function intervalManager(flag, func, time) {
        flag ? intervalID =  setInterval(func, time) : clearInterval(intervalID)
      }
      if (!!this.interval) {
        intervalManager(true, this.nextClick, this.interval)
        el.addEventListener('mouseenter', ()=> intervalManager(false))
        el.addEventListener('mouseleave', ()=> intervalManager(true, this.nextClick, this.interval))
      }
    },

  }
</script>

<style scoped>
  .carousel-control {
    cursor: pointer;
    background-image: none;
    width: 48px;
    height: 48px;
    top: 50%;
    margin-top: -24px;
    background-color: #999;
    border-radius: 48px;
    line-height: 48px;
    text-align: center;
    text-shadow: none;
  }
  .carousel-control.left{
    margin-left: 20px;
  }
  .carousel-control.right{
    margin-right: 20px;
  }
  .carousel-control >span{
    font-size: 48px;
    color: #fff;
  }
</style>