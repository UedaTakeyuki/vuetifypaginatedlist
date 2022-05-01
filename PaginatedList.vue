<template>
  <div>
    <v-list
      shaped
      dense
      style="max-height: 250px"
      class="overflow-y-auto"
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
        <v-list-item
          v-for="item in items"
          :key="item.ID"
          @click="itemSelected(item)"
        >
          <v-list-item-content>
            <v-list-item-title class="text-sm" v-text="itemName(item)" ></v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        </v-list-item-group>
    </v-list>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      // items
      items: "",
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
      if (! (item.ID in this.$devices)){
        this.$devices[item.ID] ={response: "", lastConnectTime: 0}
      }
      // request last connect time of this device
      this.requestLastConnectTime(item.ID)

      console.log("item", item)
      this.selectedItem = item
      this.$refs.device.open(item)
//      this.$refs.edit.refreshBind(bind) // not working here!
    },
    /**
     * Request last connect time of the device
     */
    async requestLastConnectTime(id){
      console.log("requestLastConnectTime called")
      const idToken = await this.getIdToken()

      // message
      let message = {
        type: "requestLastConnectTime", 
        param: null, 
        needResponseFlag: false, 
        deviceID: id,
        idToken: idToken
      }
      let messageJson = JSON.stringify(message)
      console.log("messageJson", messageJson)

      // send
      this.$ws.send(messageJson)
    },

    itemName: function(item){
      if (item.Attr.name){
        return item.Attr.name
      } else {
        return item.ID
      }
    },
    updateItems: async function() {
      this.progressLinear = true
      const idToken = await this.getIdToken()
      {
        const url=sprintf("https://connect-srv.uedasoft.com/gettotalnumberofdevices/%s",idToken)
        const res = await fetch(url)
        if (res.status == 200) {
          const data = await res.json()
          console.log("getTotalNumberOfItems", data)
          console.log("totalnumber", data.totalnumber)
          this.totalNumber = data.totalnumber
        }
      }
      {
        const url=sprintf("https://connect-srv.uedasoft.com/getdevices/%d/%d/%s", 
                          this.itemsPerPage, 
                          (this.page -1)*this.itemsPerPage,
                          idToken)
        const res = await fetch(url)
        if (res.status == 200) {
          const data = await res.json()
          console.log("result", data)
          this.items = data.devices
        }
      }
      this.progressLinear = false
    },
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
