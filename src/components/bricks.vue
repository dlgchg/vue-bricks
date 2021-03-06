<template>
  <div class="vue-bricks">
    <div
      class="vue-bricks--item"
      v-for="(item, index) in data"
      :key="index"
    >
      <slot :item="item"></slot>
    </div>
  </div>
</template>

<script>
  import Bricks from 'bricks.js'
  import ScrollEmitter from '../lib/scroll-emitter'
  export default {
    name: 'bricks',
    props: {
      data: {
        type: Array,
        required: true
      },
      sizes: {
        type: Array,
        required: true
      },
      packed: {
        type: String,
        default: 'data-packed'
      },
      position: {
        type: Boolean,
        default: true
      },
      // scroller wrapper
      container: {
        type: [HTMLElement, window]
      },
      // the offset is near page's bottom
      offset: {
        type: Number,
        default: () => 0
      },
      // imageLazy
      lazy: {
        type: Boolean,
        default: () => false
      }
    },
    data () {
      return {
        len: 0
      }
    },
    watch: {
      data (items) {
        this.len >= items.length || this.len === 0
          ? this.packLayout()
          : this.updateLayout()
        this.len = items.length
        // lazy Done
        this.scrollEmitter.lazyDone()
      },
      sizes () {
        this.updateLayout()
      },
      packed () {
        this.updateLayout()
      },
      position () {
        this.updateLayout()
      }
    },
    methods: {
      updateLayout () {
        this.$nextTick(() => this.bricks.update())
      },
      packLayout () {
        this.$nextTick(() => this.bricks.pack())
      },
      pack () {
        this.bricks.pack()
        return this
      },
      resize (flag = true) {
        this.bricks.resize(flag)
        return this
      },
      update () {
        this.bricks.update()
      }
    },
    mounted () {
      // instance
      this.bricks = Bricks({
        container: this.$el,
        sizes: this.sizes,
        packed: this.packed,
        position: this.position
      })

      // Event Emitter
      // -------------
      // pack
      this.bricks.on('pack', (...args) => {
        this.$emit('pack', args)
      })
      // update
      this.bricks.on('update', (...args) => {
        this.$emit('update', args)
      })
      // resize
      this.bricks.on('resize', (...args) => {
        this.$emit('resize', args)
      })

      // Initialize
      !this.lazy &&
      this.bricks
            .resize(true)
            .pack()

      // scroller
      // -------------
      this.scrollEmitter = new ScrollEmitter(this.container, {
        offset: this.offset
      })
      this.scrollEmitter.on('reach', () => this.$emit('reach'))
    },
    destroyed () {
      this.scrollEmitter.destroy()
    }
  }  
</script>

<style>
  .vue-bricks{
    position: relative;
    margin: 0 auto
  }
  .vue-bricks--item {
    float: left;
    position: absolute;
    top: 100%;
    /* left: 50%; */
    z-index: 1;
    /*position: fixed;
    bottom: 0;
    left: 50%;*/
    visibility: hidden;
    opacity: 0;
    /* transform: translate3d(-50%, 0, 0); */
    transition: transform .5s cubic-bezier(.55,0,.1,1), top .5s cubic-bezier(.55,0,.1,1), left .5s cubic-bezier(.55,0,.1,1), visibility .1s, opacity .8s cubic-bezier(.55,0,.1,1) .3s, top .6s;
  }
  .vue-bricks--item[data-packed],
  .vue-bricks--item[packed] {
    /* top: 0; */
    /* left: 0; */
    opacity: 1;
    visibility: visible;
  }
</style>
