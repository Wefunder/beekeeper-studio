<template>
    <div class="global-items">
      <a
        href=""
        class="nav-item selectable"
        @click.prevent="click('tables')"
        :class="{ active: activeItem === 'tables'}"
        title="Database"
      >
        <span class="bk-database"></span>
      </a>
      <a
        href=""
        class="nav-item selectable"
        @click.prevent="click('project')"
        :class="{ active: activeItem === 'project'}"
        v-if="isWefunderDB"
      >
        <span class="material-icons">dashboard</span>
      </a>
      <a
        href=""
        class="nav-item selectable"
        @click.prevent="click('queries')"
        :class="{ active: activeItem === 'queries'}"
        title="Saved Queries"
      >
        <span class="material-icons">code</span>
      </a>
      <a
        href=""
        class="nav-item selectable"
        @click.prevent="click('history')"
        :class="{ active: activeItem === 'history'}"
        title="Run History"
      >
        <span class="material-icons">history</span>
      </a>
      <span class="expand"></span>
    </div>
</template>

<script>
  export default {
    props: ['activeItem', 'isWefunderDB'],
    components: { },
    computed: {
    },
    methods: {
      click(item) {
        if( this.activeItem === item ){
          this.$emit('toggleSidebar')
        } else {
          this.$emit('selected', item)
        }
      }
    }
  }
</script>
