<template>
  <div class="page" v-if="prefsLoaded">

    <b-table striped small hover show-empty :items="items" :fields="fields">
      <template slot="name" scope="item">
        <div class="limited-width">{{ item.value }}</div>
      </template>
      <template slot="keyword" scope="item">
        <div class="limited-width">{{ item.value }}</div>
      </template>
      <template slot="icon" scope="item">
        <img class="icon" :src="item.value ? expandUserPath(item.value) : defaultIcon" />
      </template>
      <template slot="cmd" scope="item">
        <div class="cmd">
          <div class="text-wrapper">
            <div class="text">{{ item.value }}</div>
          </div>
          <div class="actions">
            <b-btn size="sm" @click="edit(item.item)"><i class="fa fa-pencil"></i></b-btn>
            <b-btn size="sm" @click="remove(item.item)"><i class="fa fa-trash"></i></b-btn>
          </div>
        </div>
      </template>
      <template slot="empty">
        <td class="empty-text" colspan="4">There are not shortcuts</td>
      </template>
    </b-table>

    <a class="add-link" href="" @click.prevent="add">
      <i class="fa fa-plus"></i> Add shortcut
    </a>
  </div>
</template>

<script>
import jsonp from '@/api'
import bus from '@/event-bus'
import { mapState, mapGetters } from 'vuex'
import defaultIcon from '../../assets/executable-icon.png'

export default {
  name: 'shortcuts',
  created () {
    this.fetchData()
  },
  data () {
    return {
      items: [],
      defaultIcon: defaultIcon,
      fields: {
        icon: {label: ''},
        name: {label: 'Name'},
        keyword: {label: 'Keyword'},
        cmd: {label: 'Query or Script'}
      }
    }
  },
  computed: {
    ...mapState(['prefs']),
    ...mapGetters(['prefsLoaded'])
  },
  methods: {
    fetchData () {
      jsonp('prefs://shortcut/get-all').then((data) => {
        this.items = data
      })
    },
    expandUserPath (path) {
      return path.indexOf('~') === 0 ? path.replace('~', this.prefs.env.user_home, 1) : path
    },
    edit (item) {
      this.$router.push({path: 'edit-shortcut', query: item})
    },
    remove (item) {
      jsonp('prefs://shortcut/remove', {id: item.id}).then(() => {
        this.items = this.items.filter((i) => item.id === i.id ? null : i)
      }, (err) => bus.$emit('error', err))
    },
    add () {
      this.$router.push({name: 'edit-shortcut'})
    }
  }
}
</script>

<style lang="scss" scoped>
.page { padding: 15px; }
button { cursor: pointer }
.empty-text { text-align: center }
.icon { width: 20px; height: 20px }
.add-link {
  color: #555;
  font-style: italic;
  display: inline-block;
  padding-left: 8px;

  i {
    margin-right: 5px;
    display: inline-block;
  }

  &:hover {
    color: #015aa7;
    text-decoration: underline;
  }
}
.limited-width {
  white-space: nowrap;
  max-width: 150px;
  min-width: 100px;
  text-overflow: ellipsis;
  overflow: hidden;
}
.cmd {
  position: relative;

  .text-wrapper {
    display: table;
    table-layout: fixed;
    width:100%;

    .text {
      display: table-cell;
      text-overflow: ellipsis;
      overflow: hidden;
      white-space: nowrap;
    }
  }

  .actions {
    position: absolute;
    display: none;
    top: 0;
    right: 0;
  }
}
tr:hover .actions {
  display: block;
}
</style>
