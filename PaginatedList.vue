<template>
  <div>
    <v-list
      shaped
      dense
      style="max-height: 250px"
      class="overflow-y-auto pa-0"
    >
      <v-subheader> {{name}} Items
        <v-spacer/>
        <v-btn 
          x-small
          class="grey--text"
          to="/docs/users-manual/binds/bindslist"
        >
          guide
        </v-btn>
      </v-subheader>

      <v-progress-linear
        v-if="progressLinear"
        indeterminate
        color="primary"
      ></v-progress-linear>

      <div class="text-center">
        <v-pagination
          v-if="totalPages >= 2"
          v-model="page"
          :length="totalPages"
          @input = "paginationSelected"
        ></v-pagination>
      </div>
      <v-list-item-group v-model="selectedItem" mandatory color="primary">
        <!-- refer https://v2.vuejs.org/v2/guide/components-slots.html#Scoped-Slots -->
        <slot name="list" v-bind:items="items">
          <v-list-item
            v-for="item in items"
            :key="item.ID"
            @click="itemSelected(item)"
          >
            <v-list-item-content>
              <v-list-item-title class="text-sm" v-text="itemName(item)" ></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </slot>
      </v-list-item-group>
    </v-list>
  </div>
</template>

<script>
var sprintf = require('sprintf-js').sprintf,
    vsprintf = require('sprintf-js').vsprintf
export default {
    props: {
//    'url',  // for get binds
    name: String, // Display name
    getIdToken: Function,
    getTotalNumber: Function,
    getItems: Function,
    requestOne: Function,
    itemName: {
      type: Function,
      default: function(item){
        if (item){
          return item.ID
        } else {
          return ""
        }
        return ""
      }
    }
  },
  data: function () {
    return {
      // items
      items: [],
      totalNumber: 0,
      // for v-pagination, getBind()
      page:          1, // current page (start from 1, not 0)
      itemsPerPage: 10, // number of items a page
      // progress-linear on/off
      progressLinear: false,
      // for v-list-item-group
      selectedItem: "", // index of binds which is selected
    }
  },
  methods: {
    // pagenation
    paginationSelected: function(page){
      console.log("page " + page)
      console.log("offset "+ (page -1)*this.itemsPerPage)
      this.updateItems()
//      this.getTotalNumberOfItems()
//      this.getItems()
      console.log("binds " + this.binds)
    },

    // item
    itemSelected: function(item){
      this.$emit('itemSelected', item)
    },
/*
    // itemName
    itemName: function(item){
      if (item.Attr.name){
        return item.Attr.name
      } else {
        return item.ID
      }
    },
*/
    // updateItem
    updateItems: async function() {
      this.progressLinear = true

      const idToken = await this.getIdToken()
      this.totalNumber = await this.getTotalNumber(idToken)
      this.items = await this.getItems(idToken, this.itemsPerPage, this.page)

      this.progressLinear = false
    },

    /**
     * For Parent
     * 
     */
    callRequestOne: async function() {
      this.progressLinear = true

      const idToken = await this.getIdToken()
      let items = await this.requestOne(idToken)
      this.totalNumber = items.length
      this.items = items

      this.progressLinear = false
    }

  },
  computed: {
    totalPages: function(){
      return Math.ceil(this.totalNumber/this.itemsPerPage)
    }
  },
  watch: {
    $isLogin: function(newVal){
      console.log("$isLogin",newVal)
      if (newVal){
        this.updateItems()
      }
    },
  },
  mounted(){
    if (this.$isLogin){
      this.updateItems()
    }
  },
}
</script>
