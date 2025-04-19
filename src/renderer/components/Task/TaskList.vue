<template>
  <div class="task-list-container">
    <div class="task-list" v-if="taskList.length > 0">
      <div
        v-for="item in taskList"
        :key="item.gid"
        :class="getItemClass(item)"
      >
        <mo-task-item
          :task="item"
          :is-selected="selectedList.includes(item.gid)"
          @select="handleSelect"
        />
      </div>
    </div>
    <div class="no-task" v-else>
      <div class="no-task-inner">
        {{ $t('task.no-task') }}
      </div>
    </div>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import { cloneDeep } from 'lodash'
  import TaskItem from './TaskItem'

  export default {
    name: 'mo-task-list',
    components: {
      [TaskItem.name]: TaskItem
    },
    data () {
      const selectedList = cloneDeep(this.$store.state.task.selectedList) || []
      return {
        selectedList
      }
    },
    computed: {
      ...mapState('task', {
        taskList: state => state.taskList,
        selectedGidList: state => state.selectedGidList
      })
    },
    methods: {
      handleSelect (gid, selected) {
        let newSelectedList = [...this.selectedList]
        if (selected) {
          if (!newSelectedList.includes(gid)) {
            newSelectedList.push(gid)
          }
        } else {
          newSelectedList = newSelectedList.filter(id => id !== gid)
        }
        this.selectedList = newSelectedList
        this.$store.dispatch('task/selectTasks', cloneDeep(newSelectedList))
      },
      getItemClass (item) {
        const isSelected = this.selectedList.includes(item.gid)
        return {
          selected: isSelected
        }
      }
    },
    watch: {
      selectedGidList (newVal) {
        this.selectedList = newVal
      }
    }
  }
</script>

<style lang="scss">
.task-list-container {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.task-list {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
  min-height: 100%;
  box-sizing: border-box;
}

.no-task {
  display: flex;
  height: 100%;
  text-align: center;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  color: #555;
  user-select: none;
}

.no-task-inner {
  width: 100%;
  padding-top: 360px;
  background: transparent url('~@/assets/no-task.svg') top center no-repeat;
}
</style>
