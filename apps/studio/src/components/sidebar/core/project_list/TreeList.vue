<template>
  <div>
    <sidebar-folder
      v-for="folder in folders"
      :title="folder.name"
      :key="folder.name"
    >
      <tree-list :items="folder.subitems" :connection="connection"></tree-list>
    </sidebar-folder>

    <div v-for="item in elements" class="list-item" :key="item.title">
      <a class="list-item-btn" role="button" @dblclick.prevent="open(item)">
        <i class="material-icons item-icon" :class="iconFor(item)">{{iconName(item)}}</i>
        <div class="list-title flex-col">
          <span class="item-text title truncate expand" :title="item.title">{{item.title}}</span>
          <span class="database subtitle truncate"><span>{{item.description || '&nbsp;'}}</span></span>
        </div>
      </a>
    </div>
  </div>
</template>

<script>
  import SidebarFolder from '@/components/common/SidebarFolder.vue'
  import { mapState } from 'vuex'

  const ITEM_ICONS = {
    'table': 'table-icon',
    'query': 'query',
    'view': 'view-icon',
    'materialized_view': 'materialized-view-icon',
    'function': 'routine-icon routine-function-icon',
    'procedure': 'routine-icon routine-procedure-icon',
  }

  const ITEM_ICON_NAME = {
    'table': 'grid_on',
    'query': 'code',
    'view': 'grid_on',
    'materialized_view': 'grid_on',
    'function': 'functions',
    'procedure': 'functions',
  }

  export default {
    name: 'tree-list',
    components: { SidebarFolder },
    props: [ 'items', 'connection' ],
    data() {
      return {
        clickState: {
          timer: null,
          openClicks: 0,
          delay: 500
        },
      }
    },
    computed: {
      ...mapState(['tables', 'routines']),
      folders() {
        return this.items.filter(o => o.type == 'folder')
      },
      elements() {
        return this.items.filter(o => o.type != 'folder')
      },
    },
    methods: {
      iconFor(item) {
        return ITEM_ICONS[item.type]
      },
      iconName(item) {
        return ITEM_ICON_NAME[item.type]
      },
      open(item) {
        if (this.clickState.openClicks > 0) return
        console.dir(this)
        console.dir(item)

        if (item.type == 'query') {
          this.$root.$emit("loadQuery", {
            id: -item.id,
            title: item.title,
            description: item.description,
            query: item.command,
          })
        } else if(item.type == 'function' || item.type == 'procedure') {
          const routine = this.routines.find(o => o.schema == item.schema && o.name == item.object)
          console.dir(routine)
          console.log(item.schema, item.object)
          this.$root.$emit("loadRoutine", { routine })
        } else {
          const table = this.tables.find(o => o.schema == item.schema && o.name == item.object)
          console.dir(table)
          console.log(item.schema, item.object)
          this.$root.$emit("loadTable", { table })
        }

        this.clickState.openClicks ++;
        this.clickState.timer = setTimeout(() => {
          this.clickState.openClicks = 0
        }, this.clickState.delay);
      },
    }
  }
</script>
