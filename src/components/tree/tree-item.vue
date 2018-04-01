<template>
  <div class="t-tree-item">
    <div
      @click="checkout"
      class="t-tree-item__label"
      :style="{paddingLeft: indent}">
      <i
        class="fa fa-caret-right t-tree-item__drop-icon"
        :style="{visibility: (hasChildren || (!hasChildren && lazy && !loaded)) ? 'visible' : 'hidden'}"
        :class="[expand ? 't-tree-item__drop-icon--open' : '']">
      </i>
      <t-checkbox :indeterminate="indeterminate" v-model="isChecked" v-if="showCheckbox" :disabled="checkDisabled" @change="checkChangeHandler"/>
      <i v-if="loading"
         class="fa fa-circle-notch fa-spin t-tree-item__loading-icon">
      </i>
      {{ label }}
    </div>
    <transition name="t-fly-in-center">
      <div
        class="t-tree-item__children"
        v-show="expand">
        <slot></slot>
      </div>
    </transition>
  </div>
</template>

<script>
import Emitter from '../../mixins/emitter'

export default {
  name: 't-tree-item',

  mixins: [Emitter],

  provide: {
    TTreeItem: this
  },

  inject: {
    TTreeItem: {
      default: ''
    }
  },

  data () {
    return {
      expand: false,
      currentIndex: '',
      isChecked: false,
      childrenCheckCount: 0,
      indeterminateChildren: 0,
      reechoCount: 0,
      indeterminate: false,
      loaded: false,
      loading: false
    }
  },

  props: {
    label: String,
    indent: String,
    hasChildren: Boolean,
    showCheckbox: Boolean,
    lazy: Boolean,
    nodeIndex: String,
    initChecked: Boolean,
    initExpand: Boolean,
    checkDisabled: Boolean
  },

  created () {
    this.$on('check-change', this.checkChangeHandler)
    this.$on('node-item-echo', this.echoHandler)
    this.$on('node-item-reecho', this.reechoHandler)
  },

  mounted () {
    this.initChecked && this.checkChangeHandler(this.initChecked, true)
    this.initExpand && (this.expand = true)
  },

  methods: {
    echoHandler (val) {
      if (this.hasChildren) {
        this.broadcast('t-tree-item', 'node-item-echo', val)
      } else {
        !this.checkDisabled && (this.isChecked = val)
        this.TTreeItem && this.dispatch('t-tree-item', 'node-item-reecho', val)
      }
    },
    reechoHandler () {
      this.isChecked = this.childCheckAll()[0]
      this.indeterminate = this.childCheckAll()[1]
      this.TTreeItem && this.dispatch('t-tree-item', 'node-item-reecho', this.isChecked)
    },
    checkChangeHandler (val, init = false) {
      if (!init && this.checkDisabled) return

      if (this.hasChildren) {
        this.broadcast('t-tree-item', 'node-item-echo', val)
      } else {
        this.isChecked = val
        this.TTreeItem && this.dispatch('t-tree-item', 'node-item-reecho', val)
      }

      this.dispatch('t-tree', 'check-change', {
        label: this.label,
        checked: val,
        indeterminate: this.indeterminate
      })
    },
    checkout () {
      this.expand = !this.expand
    },
    childCheckAll () {
      let checkCount = 0
      let indeterminateCount = 0
      this.$children.forEach(function (el, idx) {
        if (idx > 0) {
          el.isChecked && checkCount++
          el.indeterminate && indeterminateCount++
        }
      })

      return [checkCount === this.childLen, (checkCount > 0 && checkCount < this.childLen) || indeterminateCount > 0]
    }
  },

  watch: {
    expand (val) {
      this.dispatch('t-tree', 'node-click', {
        label: this.label
      })
      if (this.lazy && !this.loaded && !this.loading) {
        this.loading = true
        this.dispatch('t-tree', 'lazy-load', this)
      }
    }
  },
  computed: {
    childLen () {
      return this.$children.length - 1
    }
  }
}
</script>

<style scoped>

</style>