<template>
  <div class="sidebar-project flex-col expand">
    <div class="sidebar-list">
      <nav class="list-group" ref="wrapper">
        <div class="list-heading row">
          <div class="sub row flex-middle expand">
            <div class="expand" title="Wefunder Project Dashboard">
              Wefunder Dashboard
            </div>
            <div class="actions">
              <a @click.prevent="refresh">
                <i title="Refresh Project" class="material-icons">refresh</i>
              </a>
            </div>
          </div>
        </div>
        <error-alert v-if="error" :error="error" title="Problem loading project" />
        <sidebar-loading v-else-if="loading" />
        <div v-else class="list-body">
          <div class="with-schemas">
            <tree-list :items="this.items" :connection="connection"></tree-list>
          </div>
        </div>
      </nav>
    </div>
  </div>
</template>

<script>
  import _ from 'lodash'
  import TreeList from './project_list/TreeList'
  import ErrorAlert from '@/components/common/ErrorAlert.vue'
  import SidebarLoading from '@/components/common/SidebarLoading.vue'

  import { mapState } from 'vuex'

  const ITEMS_QUERY = `
    SELECT "type", "schema", "object", "folder", "title", "description", "command"
    FROM   bks.project_items
    WHERE  visible IS TRUE
    ORDER BY folder NULLS FIRST, title
  `

  export default {
    components: {
      ErrorAlert,
      SidebarLoading,
      TreeList,
    },
    data: function () {
      return {
        loading: true,
        items: [],
        error: null,
      }
    },
    computed: {
      ...mapState(['connection']),
    },
    mounted() {
      document.addEventListener('mousedown', this.maybeUnselect)
      this.loadItems();
    },
    beforeDestroy() {
      document.removeEventListener('mousedown', this.maybeUnselect)
    },
    methods: {
      refresh() {
        this.loading = true
        this.items = []
        this.error = null
        this.loadItems()
      },
      async loadItems() {
        const data = await this.connection.executeQuery(ITEMS_QUERY)
        const rows = data?.[0]?.rows

        if (!rows.length) {
          this.error = 'Unable to load the records'
          return
        }

        const index = { '/': this.items }
        rows.forEach(({ folder, ...rest }) => {
          folder = folder || '/'

          if (!_.has(index, folder)) {
            folder.split('/').reduce(([list, baseName], name) => {
              const fullName = baseName ? `${baseName}/${name}` : name
              if (_.has(index, fullName)) return [index[fullName], fullName]

              const items = index[fullName] = []
              list.push({ type: 'folder', name, subitems: items })
              return [items, fullName]
            }, [index['/'], ''])
          }

          index[folder].push(rest)
        })

        this.loading = false
      },
    },
  }
</script>
