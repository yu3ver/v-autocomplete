<template lang="html">
  <div class="v-autocomplete">
    <div class="v-autocomplete-input-group" :class="{'v-autocomplete-selected': value}">
      <input type="search" v-model="searchText" :placeholder="placeholder" :class="inputClass"
            :disabled="disabled" @blur="blur" @focus="focus" @input="inputChange"
            @keyup.enter="keyEnter" @keyup.up="keyUp" @keyup.down="keyDown">
    </div>
    <div class="v-autocomplete-list" v-if="showList && internalItems.length">
      <div class="v-autocomplete-list-item" v-for="item, i in internalItems" @click="onClickItem(item)"
           :class="{'v-autocomplete-item-active': i === cursor}">
        <div :is="componentItem" :item="item"></div>
      </div>
    </div>
  </div>
</template>

<script>
import Item from './Item.vue'
import utils from './utils.js'

export default {
  name: 'v-autocomplete',
  props: {
    componentItem: { default: () => Item },
    placeholder: String,
    minLen: { type: Number, default: utils.minLen },
    wait: { type: Number, default: utils.wait },
    value: null,
    getLabel: {
      type: Function,
      default: item => item
    },
    items: Array,
    autoSelectOneItem: { type: Boolean, default: true },
    inputClass: {type: String, default: 'v-autocomplete-input'},
    disabled: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      searchText: '',
      showList: false,
      cursor: -1,
      internalItems: this.items || []
    }
  },
  methods: {
    inputChange () {
      this.showList = true
      this.cursor = -1
      this.onSelectItem(null, 'inputChange')
      utils.callUpdateItems(this.searchText, this.updateItems)
      this.$emit('change', this.searchText)
    },

    updateItems () {
      this.$emit('update-items', this.searchText)
    },

    focus () {
      this.showList = true
    },

    blur () {
      setTimeout( () => this.showList = false, 200)
    },

    onClickItem(item) {
      this.onSelectItem(item)
      this.$emit('item-clicked', item)
    },

    onSelectItem (item) {
      if (item) {
        this.internalItems = [item]
        this.searchText = this.getLabel(item)
        this.$emit('item-selected', item)
      } else {
        this.setItems(this.items)
      }
      this.$emit('input', item)
    },

    setItems (items) {
      this.internalItems = items || []
    },

    isSelecteValue (value) {
      return 1 == this.internalItems.length && value == this.internalItems[0]
    },

    keyUp (e) {
      if (this.cursor > -1) {
        this.cursor--
        this.itemView(this.$el.getElementsByClassName('v-autocomplete-list-item')[this.cursor])
      }
    },

    keyDown (e) {
      if (this.cursor < this.internalItems.length) {
        this.cursor++
        this.itemView(this.$el.getElementsByClassName('v-autocomplete-list-item')[this.cursor])
      }
    },

    itemView (item) {
      if (item && item.scrollIntoView) {
        item.scrollIntoView(false)
      }
    },

    keyEnter (e) {
      if (this.showList && this.internalItems[this.cursor]) {
        this.onSelectItem(this.internalItems[this.cursor])
      }
    },

  },
  created () {
    utils.minLen = this.minLen
    utils.wait = this.wait
    this.onSelectItem(this.value)
  },
  watch: {
    items (newValue) {
      this.setItems(newValue)
      let item = utils.findItem(this.items, this.searchText, this.autoSelectOneItem)
      if (item) {
        this.onSelectItem(item)
        this.showList = false
      }
    },
    value (newValue) {
      if (!this.isSelecteValue(newValue) ) {
        this.onSelectItem(newValue)
      }
    }
  }
}
</script>

<style>
  .v-autocomplete {
    position: relative;
  }
  .v-autocomplete .v-autocomplete-list {
    position: absolute;
  }
  .v-autocomplete .v-autocomplete-list .v-autocomplete-list-item {
    cursor: pointer;
  }
  .v-autocomplete .v-autocomplete-list .v-autocomplete-list-item.v-autocomplete-item-active {
    background-color: #f3f6fa;
  }
</style>
