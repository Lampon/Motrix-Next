<template>
  <div :key="task.gid" class="task-item" v-on:dblclick="onDbClick">
    <el-checkbox
      class="task-checkbox"
      v-model="isChecked"
      @change="handleCheckboxChange"
    />
    <div class="task-name" :title="taskFullName">
      <span>{{ taskFullName }}</span>
    </div>
    <mo-task-item-actions mode="LIST" :task="task" />
    <div class="task-progress">
      <mo-task-progress
        :completed="Number(task.completedLength)"
        :total="Number(task.totalLength)"
        :status="taskStatus"
      />
      <mo-task-progress-info :task="task" />
    </div>
  </div>
</template>

<script>
  import { checkTaskIsSeeder, getTaskName } from '@shared/utils'
  import { TASK_STATUS } from '@shared/constants'
  import { openItem, getTaskFullPath } from '@/utils/native'
  import TaskItemActions from './TaskItemActions'
  import TaskProgress from './TaskProgress'
  import TaskProgressInfo from './TaskProgressInfo'

  export default {
    name: 'mo-task-item',
    components: {
      [TaskItemActions.name]: TaskItemActions,
      [TaskProgress.name]: TaskProgress,
      [TaskProgressInfo.name]: TaskProgressInfo
    },
    props: {
      task: {
        type: Object
      },
      isSelected: {
        type: Boolean,
        default: false
      }
    },
    data () {
      return {
        isChecked: this.isSelected
      }
    },
    computed: {
      taskFullName () {
        return getTaskName(this.task, {
          defaultName: this.$t('task.get-task-name'),
          maxLen: -1
        })
      },
      taskName () {
        return getTaskName(this.task, {
          defaultName: this.$t('task.get-task-name')
        })
      },
      isSeeder () {
        return checkTaskIsSeeder(this.task)
      },
      taskStatus () {
        const { task, isSeeder } = this
        if (isSeeder) {
          return TASK_STATUS.SEEDING
        } else {
          return task.status
        }
      }
    },
    watch: {
      isSelected (newVal) {
        this.isChecked = newVal
      }
    },
    methods: {
      handleCheckboxChange (val) {
        this.$emit('select', this.task.gid, val)
      },
      onDbClick () {
        const { status } = this.task
        const { COMPLETE, WAITING, PAUSED } = TASK_STATUS
        if (status === COMPLETE) {
          this.openTask()
        } else if ([WAITING, PAUSED].includes(status) !== -1) {
          this.toggleTask()
        }
      },
      async openTask () {
        const { taskName } = this
        this.$msg.info(this.$t('task.opening-task-message', { taskName }))
        const fullPath = getTaskFullPath(this.task)
        const result = await openItem(fullPath)
        if (result) {
          this.$msg.error(this.$t('task.file-not-exist'))
        }
      },
      toggleTask () {
        this.$store.dispatch('task/toggleTask', this.task)
      }
    }
  }
</script>

<style lang="scss">
.task-item {
  position: relative;
  min-height: 78px;
  padding: 16px;
  background-color: $--task-item-background;
  border: 1px solid $--task-item-border-color;
  border-radius: 6px;
  margin-bottom: 12px;
  transition: all 0.3s ease;
  display: flex;
  align-items: flex-start;
  &:hover {
    border-color: $--task-item-hover-border-color;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  }
  .task-checkbox {
    margin-right: 16px;
    margin-top: 4px;
  }
  .task-item-actions {
    position: absolute;
    top: 16px;
    right: 16px;
  }
}

.selected .task-item {
  border-color: $--color-primary;
  background-color: rgba($--color-primary, 0.05);
}

.task-name {
  color: $--color-text-primary;
  margin-bottom: 1rem;
  margin-right: 200px;
  word-break: break-all;
  min-height: 26px;
  flex: 1;
  margin-left: 8px;
  &> span {
    font-size: 14px;
    line-height: 26px;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
  }
}

.task-progress {
  position: absolute;
  bottom: 16px;
  left: 16px;
  right: 16px;
}
</style>
