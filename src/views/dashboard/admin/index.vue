<template>
  <div class="dashboard-editor-container">

    <panel-group :listData="list" @handleSetLineChartData="handleSetLineChartData" />

<!--    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">-->
<!--      <line-chart :listData="list" :chart-data="lineChartData" />-->
<!--    </el-row>-->

<!--    <el-row :gutter="32">-->
<!--      <el-col :xs="24" :sm="24" :lg="8">-->
<!--        <div class="chart-wrapper">-->
<!--          <raddar-chart />-->
<!--        </div>-->
<!--      </el-col>-->
<!--      <el-col :xs="24" :sm="24" :lg="8">-->
<!--        <div class="chart-wrapper">-->
<!--          <pie-chart />-->
<!--        </div>-->
<!--      </el-col>-->
<!--      <el-col :xs="24" :sm="24" :lg="8">-->
<!--        <div class="chart-wrapper">-->
<!--          <bar-chart />-->
<!--        </div>-->
<!--      </el-col>-->
<!--    </el-row>-->

<!--    <el-row :gutter="8">-->
<!--      <el-col :xs="{span: 24}" :sm="{span: 24}" :md="{span: 24}" :lg="{span: 12}" :xl="{span: 12}" style="padding-right:8px;margin-bottom:30px;">-->
<!--        <transaction-table />-->
<!--      </el-col>-->
<!--      <el-col :xs="{span: 24}" :sm="{span: 12}" :md="{span: 12}" :lg="{span: 6}" :xl="{span: 6}" style="margin-bottom:30px;">-->
<!--        <todo-list />-->
<!--      </el-col>-->
<!--      <el-col :xs="{span: 24}" :sm="{span: 12}" :md="{span: 12}" :lg="{span: 6}" :xl="{span: 6}" style="margin-bottom:30px;">-->
<!--        <box-card />-->
<!--      </el-col>-->
<!--    </el-row>-->
  </div>
</template>

<script>
import GithubCorner from '@/components/GithubCorner'
import PanelGroup from './components/PanelGroup'
import LineChart from './components/LineChart'
import RaddarChart from './components/RaddarChart'
import PieChart from './components/PieChart'
import BarChart from './components/BarChart'
import TransactionTable from './components/TransactionTable'
import TodoList from './components/TodoList'
import BoxCard from './components/BoxCard'
import moment from 'moment'
import Mock from 'mockjs'

const lineChartData = {
  newVisitis: {
    expectedData: [100, 120, 161, 134, 105, 160, 165],
    actualData: [120, 82, 91, 154, 162, 140, 145]
  },
  messages: {
    expectedData: [200, 192, 120, 144, 160, 130, 140],
    actualData: [180, 160, 151, 106, 145, 150, 130]
  },
  purchases: {
    expectedData: [80, 100, 121, 104, 105, 90, 100],
    actualData: [120, 90, 100, 138, 142, 130, 130]
  },
  shoppings: {
    expectedData: [130, 140, 141, 142, 145, 150, 160],
    actualData: [120, 82, 91, 154, 162, 140, 130]
  }
}

export default {
  name: 'DashboardAdmin',
  components: {
    GithubCorner,
    PanelGroup,
    LineChart,
    RaddarChart,
    PieChart,
    BarChart,
    TransactionTable,
    TodoList,
    BoxCard
  },
  data() {
    return {
      total: 0,
      list: [],
      filterList: [],
      lineChartData: lineChartData.newVisitis
    }
  },
  created() {
    this.initTask().then(() => {
      console.log('任务表初始化成功')
      this.listLoading = false

      // 计算当前页的数据开始索引
      const start = (this.listQuery.page - 1) * this.listQuery.limit
      // 计算当前页的数据结束索引
      const end = start + this.listQuery.limit
      // 从list中截取当前页的数据
      const currentPageData = this.list.slice(start, end)

      this.filterList = currentPageData.map((item, index) => {
        return {
          index: index + 1,
          ...item
        }
      })
      this.total = this.list.length
    })
  },
  methods: {
    handleSetLineChartData(type) {
      this.lineChartData = lineChartData[type]
    },
    initTask() {
      const vm = this
      vm.listLoading = true
      return new Promise((resolve, reject) => {
        let db
        const request = indexedDB.open('xdd-server')

        request.onerror = (event) => {
          console.error('数据库打开失败')
          reject(new Error('数据库打开失败'))
        }

        request.onsuccess = (event) => {
          db = event.target.result
          if (!db.objectStoreNames.contains('task')) {
            // 如果不存在task表，更新版本号以重新打开数据库
            const currentVersion = db.version
            db.close()
            vm.reopenAndInitDB(currentVersion + 1).then(resolve).catch(reject)
          } else {
            // 如果存在task表，查询数据并更新this.list
            const transaction = db.transaction(['task'], 'readonly')
            const objectStore = transaction.objectStore('task')
            const request = objectStore.getAll()

            request.onerror = (event) => {
              console.error('查询数据失败')
              reject(new Error('查询数据失败'))
            }

            request.onsuccess = (event) => {
              vm.list = []
              vm.list = event.target.result.sort((a, b) => {
                // 使用 moment 解析 updateTime 并比较
                // 将 b 放在前面和 a 放在后面实现降序排序
                return moment(b.updateTime, 'YYYY-MM-DD HH:mm:ss') - moment(a.updateTime, 'YYYY-MM-DD HH:mm:ss')
              }).map((item, index) => {
                return {
                  ...item,
                  index: index + 1
                }
              }) // 假设this.list是要更新的数据列表
              resolve(vm.list)
            }
          }
        }
      })
    },
    reopenAndInitDB(newVersion) {
      return new Promise((resolve, reject) => {
        const request = indexedDB.open('xdd-server', newVersion)

        request.onerror = (event) => {
          console.error('重新打开数据库失败')
          reject(new Error('重新打开数据库失败'))
        }

        request.onupgradeneeded = (event) => {
          const db = event.target.result
          if (!db.objectStoreNames.contains('task')) {
            const objectStore = db.createObjectStore('task', { keyPath: 'id', autoIncrement: true })
            objectStore.createIndex('name', 'name', { unique: false })
            // 创建其他字段的索引
            objectStore.createIndex('createTime', 'createTime', { unique: false })
            objectStore.createIndex('updateTime', 'updateTime', { unique: false })
            objectStore.createIndex('description', 'description', { unique: false })
            objectStore.createIndex('lastDateTime', 'lastDateTime', { unique: false })
            objectStore.createIndex('personInCharge', 'personInCharge', { unique: false })
            objectStore.createIndex('picker', 'picker', { unique: false })
            objectStore.createIndex('priority', 'priority', { unique: false })
            // objectStore.createIndex('status', 'status', { unique: false })
            objectStore.createIndex('comments', 'comments', { unique: false })
          }
        }

        request.onsuccess = (event) => {
          const db = event.target.result
          const transaction = db.transaction(['task'], 'readwrite')
          const objectStore = transaction.objectStore('task')
          // 使用Mockjs生成10条模拟数据并添加到数据库
          let array = []
          for (let i = 0; i < 30; i++) {
            let createTime = Mock.Random.datetime('yyyy-MM-dd HH:mm:ss')
            let updateTime = Mock.Random.datetime('yyyy-MM-dd HH:mm:ss')
            let lastDateTime = Mock.Random.datetime('yyyy-MM-dd HH:mm:ss')

            // 确保 createTime 在 updateTime 之前，lastDateTime 在 updateTime 之后
            if (createTime > updateTime) {
              [createTime, updateTime] = [updateTime, createTime]
            }
            lastDateTime = Mock.Random.date('yyyy-MM-dd') + ' ' + Mock.Random.time('HH:mm:ss')
            while (lastDateTime <= updateTime) {
              lastDateTime = Mock.Random.date('yyyy-MM-dd') + ' ' + Mock.Random.time('HH:mm:ss')
            }
            const mockData = Mock.mock({
              // 根据字段生成模拟数据
              id: '@guid',
              name: '@ctitle(5, 10)',
              // type: '@word',
              createTime,
              updateTime,
              description: '@csentence(10, 20)',
              lastDateTime,
              personInCharge: '@cname',
              picker: '',
              priority: '@pick(["低", "中", "高"])',
              status: '@pick(["待分配", "进行中", "已完成"])',
              comments: []
            })
            array.push(mockData)
            objectStore.add(mockData)
          }

          transaction.oncomplete = () => {
            console.log('任务表初始化完成')
            this.list = array.sort((a, b) => {
              // 使用 moment 解析 updateTime 并比较
              // 将 b 放在前面和 a 放在后面实现降序排序
              return moment(b.updateTime, 'YYYY-MM-DD HH:mm:ss') - moment(a.updateTime, 'YYYY-MM-DD HH:mm:ss')
            })
            resolve()
          }

          transaction.onerror = (event) => {
            reject(new Error('任务数据填充失败'))
          }
        }
      })
    },
  }
}
</script>

<style lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .github-corner {
    position: absolute;
    top: 0px;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width:1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
