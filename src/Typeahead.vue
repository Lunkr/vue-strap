<template>
  <div style="position: relative"
       v-bind:class="{'open':showDropdown}"
  >
    <input type="text" class="form-control"
           :placeholder="placeholder"
           autocomplete="off"
           v-model="query"
           @input="update"
           @keydown.up="up"
           @keydown.down="down"
           @keydown.enter= "hit"
           @keydown.esc="reset"
    />
    <ul class="dropdown-menu" v-el:dropdown>
      <li v-for="item in items" v-bind:class="{'active': isActive($index)}">
        <a @mousedown.prevent="hit" @mousemove="setActive($index)">
          <span v-html="item[key] | highlight query"></span>
        </a>
      </li>
    </ul>
  </div>

</template>

<script type="text/babel">
  const typeahead = {
    created() {
      this.items = this.primitiveData
    },
    watch: {
      data() {
        this.update()
      }
    },
    props: {
      query: {
        type: String
      },
      data: {
        type: Array
      },
      limit: {
        type: Number,
        default: 8
      },
      key: {
        type: String
      },
      onHit: {
        type: Function,
        default(items) {
          this.reset()
          this.query = items
        }
      },
      placeholder: {
        type: String
      }
    },
    data() {
      return {
        showDropdown: false,
        noResults: true,
        current: 0,
        items: [],
      }
    },
    computed: {
      primitiveData() {
        if (this.data) {
          return this.data.slice(0, this.limit)
        }
      }
    },
    methods: {
      update() {
        if (this.data) {
          this.items = this.primitiveData
          this.showDropdown = this.items.length
        }
      },
      reset() {
        this.items = []
        this.query = ''
        this.loading = false
        this.showDropdown = false
      },
      setActive(index) {
        this.current = index
      },
      isActive(index) {
        return this.current === index
      },
      hit(e) {
        e.preventDefault()
        this.showDropdown = false
        this.onHit(this.items[this.current], this)
      },
      up() {
        if (this.current > 0) this.current--
      },
      down() {
        if (this.current < this.items.length - 1) this.current++
      }
    },
    filters: {
      highlight(value, phrase) {
        phrase = (phrase + '').replace(/([.?*+^$[\]\\(){}|-])/g, '\\$1')
        return value.replace(new RegExp('('+phrase+')', 'gi'), '<strong>$1</strong>')
      }
    }
  }
  export default typeahead
</script>

<style lang="less" rel="stylesheet/less">
  >a{
    cursor: pointer;
  }
</style>
