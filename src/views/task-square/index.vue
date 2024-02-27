<template>
  <div class="app-container">
    <div class="filter-container">
      <div class="filter-container-left">
        <el-input v-model="listQuery.name" placeholder="名称" style="width: 200px;margin-right: 8px" class="filter-item"
                  @keyup.enter.native="handleFilter"
        />
        <el-select v-model="listQuery.priority" placeholder="优先级" clearable style="width: 90px;margin-right: 8px"
                   class="filter-item"
        >
          <el-option v-for="item in importanceOptions" :key="item.label" :label="item.label" :value="item.label">
            <el-rate :value="handleRate(item)" :max="3" disabled/>
          </el-option>
        </el-select>
        <el-select v-model="listQuery.status" placeholder="状态" clearable class="filter-item"
                   style="width: 130px;margin-right: 8px"
        >
          <el-option v-for="item in statusOptions" :key="item" :label="item"
                     :value="item"
          >
            <el-tag :type="handleStatusColor(item)">{{ item }}</el-tag>
          </el-option>
        </el-select>
      </div>
      <div>
        <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
          搜索
        </el-button>
      </div>
    </div>
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="filterList"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <el-table-column label="序号" width="80px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.index }}</span>
        </template>
      </el-table-column>
      <el-table-column label="名称" width="150px" align="left">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="任务描述" min-width="200px">
        <template slot-scope="{row}">
          <span :style="{overflow: 'hidden',textOverflow: 'ellipsis', whiteSpace: 'nowrap'}">{{
              row.description
            }}</span>
        </template>
      </el-table-column>
      <el-table-column label="任务发布者" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.personInCharge }}</span>
        </template>
      </el-table-column>
      <el-table-column label="负责人" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.picker || '-' }}</span>
        </template>
      </el-table-column>
      <el-table-column label="优先级" width="110px" align="center">
        <template slot-scope="{row}">
          <el-rate :value="handleRate(row.priority)" disabled></el-rate>
        </template>
      </el-table-column>
      <el-table-column label="状态" width="110px" align="center">
        <template slot-scope="{row}">
          <el-tag :type="handleStatusColor(row.status)">{{ row.status }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="截止日期" width="160px">
        <template slot-scope="{row}">
          <span>{{ row.lastDateTime }}</span>
        </template>
      </el-table-column>
      <!--      <el-table-column label="任务最后更新时间" width="160px">-->
      <!--        <template slot-scope="{row}">-->
      <!--          <span>{{ row.updateTime }}</span>-->
      <!--        </template>-->
      <!--      </el-table-column>-->
      <el-table-column label="操作" fixed="right" align="center" width="160" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button v-if="row.status === '待分配'" type="primary" size="mini" @click="handleUpdate(row)">
            领取
          </el-button>
          <el-button v-if="row.status === '进行中'" type="success" size="mini" @click="handleFinish(row,'已完成')">
            完成
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit"
                @pagination="handlePaginationChange"
    />

    <el-dialog v-loading="dialogLoading" title="领取任务" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width=" 90px"
               style="width: 400px; margin-left:50px;"
      >
        <el-form-item label="任务名称" prop="name" >
          <el-input v-model="temp.name" disabled/>
        </el-form-item>
        <el-form-item label="任务描述" prop="remark">
<!--          <el-input v-model="temp.remark" :autosize="{ minRows: 2, maxRows: 4}" type="textarea"-->
<!--                    placeholder="请填入任务描述"-->
<!--          />-->
          <p style="margin: 0">{{temp.remark}}</p>
        </el-form-item>
        <el-form-item label="优先级" prop="priority">
          <el-rate v-model="temp.priority" :max="3" style="margin-top:8px;" disabled/>
        </el-form-item>
        <el-form-item label="截止时间" prop="lastDateTime"  >
          <el-date-picker v-model="temp.lastDateTime" type="datetime" placeholder="请选择截止时间"
                          format="yyyy-MM-DD HH:mm:ss" disabled
          />
        </el-form-item>
        <el-form-item label="任务发布者" prop="personInCharge"  >
          <el-input v-model="temp.personInCharge" disabled/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData('进行中')">
          领取
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel"/>
        <el-table-column prop="pv" label="Pv"/>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchPv, createArticle, updateArticle } from '@/api/article'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import Mock from 'mockjs'
import moment from 'moment'
import _ from 'lodash'
import { mapGetters } from 'vuex'

export default {
  name: 'TaskList',
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      tableKey: 0,
      list: null,
      filterList: null,
      total: 0,
      listLoading: true,
      dialogLoading: false,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        name: '',
        priority: '',
        status: ''
      },
      importanceOptions: [{
        label: '低',
        value: 1
      }, {
        label: '中',
        value: 2
      }, {
        label: '高',
        value: 3
      }],
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
      statusOptions: ['待分配', '进行中', '已完成'],
      showReviewer: false,
      temp: {
        id: Mock.Random.uuid(),
        name: '',
        priority: 1,
        remark: '',
        personInCharge: '',
        lastDateTime: undefined,
        status: '待分配'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      dialogPvVisible: false,
      pvData: [],
      rules: {
        name: [{ required: true, message: '请填写任务名称', trigger: 'blur' }],
        remark: [{ required: true, message: '请填写任务描述', trigger: 'blur' }],
        priority: [{ required: true, message: '请选择任务优先级', trigger: 'change' }],
        lastDateTime: [{ type: 'date', required: true, message: '请选择截止时间', trigger: 'change' }]
      }
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'avatar',
      'roles'
    ])
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
  watch: {
    list(newData) {
      // 计算当前页的数据开始索引
      const start = (this.listQuery.page - 1) * this.listQuery.limit
      // 计算当前页的数据结束索引
      const end = start + this.listQuery.limit
      // 从list中截取当前页的数据
      const currentPageData = newData.slice(start, end)

      this.filterList = currentPageData.map((item, index) => {
        return {
          index: index + 1,
          ...item
        }
      })
      this.total = this.list.length
      this.tableKey++
    }
  },
  methods: {
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
            const index = objectStore.index('status')
            const request = index.getAll()

            request.onerror = (event) => {
              console.error('查询数据失败')
              reject(new Error('查询数据失败'))
            }

            request.onsuccess = (event) => {
              vm.list = []
              vm.list = event.target.result.filter((item) => {
                return item.status === '待分配' || item.picker === this.name
              }).sort((a, b) => {
                // 使用 moment 解析 updateTime 并比较
                // 将 b 放在前面和 a 放在后面实现降序排序
                return moment(b.updateTime, 'YYYY-MM-DD HH:mm:ss') - moment(a.updateTime, 'YYYY-MM-DD HH:mm:ss')
              }) // 假设this.list是要更新的数据列表
              console.log("-> list", vm.list);
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
            objectStore.createIndex('status', 'status', { unique: false })
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
    handleRate(type) {
      if (type.value) {
        return type.value
      } else {
        switch (type) {
          case '低':
            return 1
          case '中':
            return 2
          case '高':
            return 3
          default:
            return type
        }
      }
    },
    handleStatusColor(status) {
      let color = 'info'
      switch (status) {
        case '待分配':
          color = 'info'
          break
        case '进行中':
          color = ''
          break
        case '已完成':
          color = 'success'
          break
      }
      return color
    },
    handleFilter() {
      this.listQuery.page = 1
      console.log("-> listQuery", this.listQuery);
      this.filterList = this.list.filter((item) => {
        return item.name.toLowerCase().indexOf(this.listQuery.name.toLowerCase()) !== -1
          && (_.isEmpty(this.listQuery.priority) ||item.priority === this.listQuery.priority)
          && (_.isEmpty(this.listQuery.status) ||item.status === this.listQuery.status)
      }).map((item, index) => {
        return {
          ...item,
          index: index + 1
        }
      })
      // console.log('-> this.filterList', this.filterList)
      this.total = this.filterList.length
      // this.getList()
    },
    handlePaginationChange({ page, limit }) {
      this.listLoading = true
      this.listQuery.page = page
      this.listQuery.limit = limit
      setTimeout(() => {
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
        this.listLoading = false
      }, 1500)
    },
    resetTemp() {
      this.temp = {
        id: Mock.Random.uuid(),
        name: '',
        priority: 1,
        remark: '',
        personInCharge: '',
        lastDateTime: undefined,
        status: '待分配'
      }
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.temp.lastDateTime = moment(this.temp.lastDateTime, 'YYYY-MM-DD HH:mm:ss').toDate()
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData(status) {
      if (this.dialogLoading) return
      try {
        this.dialogLoading = true
        let db
        const openRequest = indexedDB.open('xdd-server')

        openRequest.onsuccess = (event) => {
          db = event.target.result
          const currentVersion = db.version
          db.close()

          // 重新打开数据库，这次使用新的版本号
          const updateRequest = indexedDB.open('xdd-server', currentVersion + 1)

          updateRequest.onupgradeneeded = (event) => {
            db = event.target.result
            let objectStore
            if (!db.objectStoreNames.contains('task')) {
              objectStore = db.createObjectStore('task', { keyPath: 'id', autoIncrement: true })
              // 如果需要，创建索引
            } else {
              objectStore = updateRequest.transaction.objectStore('task')
            }

            // 使用moment格式化当前时间
            const formattedTime = moment().format('YYYY-MM-DD HH:mm:ss')
            let _temp = _.cloneDeep(this.temp)
            _temp.updateTime = formattedTime // 仅更新 updateTime 字段
            _temp.picker = this.name
            _temp.status = status

            // 如果需要，更新其他字段
            _temp.lastDateTime = moment(_temp.lastDateTime).format('YYYY-MM-DD HH:mm:ss')

            // 更新数据记录
            objectStore.put(_temp) // 使用put方法更新数据
          }

          updateRequest.onsuccess = (event) => {
            this.dialogFormVisible = false
            this.$notify({
              title: '操作成功',
              message: `更新任务${this.temp.name}成功`,
              type: 'success',
              duration: 2000
            })
            this.dialogLoading = false
            // 重新加载数据
            this.initTask().then(() => {
              console.log('重新加载数据成功')
              this.listLoading = false
              db.close()
            })
          }

          updateRequest.onerror = (event) => {
            this.dialogLoading = false
            this.listLoading = false
            console.error('数据库更新失败:', event.target.error)
            db.close()
          }
        }

        openRequest.onerror = (event) => {
          console.error('数据库打开失败:', event.target.error)
          db.close()
        }
      } catch (e) {
        console.error(e)
        this.dialogLoading = false
      }
    },
    handleDelete(row, index) {
      if (this.listLoading) return
      try {
        this.listLoading = true
        let db
        const openRequest = indexedDB.open('xdd-server')

        openRequest.onsuccess = (event) => {
          db = event.target.result
          const currentVersion = db.version
          db.close()

          // 重新打开数据库，这次使用新的版本号
          const updateRequest = indexedDB.open('xdd-server', currentVersion + 1)

          updateRequest.onupgradeneeded = (event) => {
            db = event.target.result
            let objectStore
            if (db.objectStoreNames.contains('task')) {
              objectStore = updateRequest.transaction.objectStore('task')

              // 删除指定的数据记录
              objectStore.delete(row.id) // 使用delete方法删除数据
            }
          }

          updateRequest.onsuccess = (event) => {
            this.$notify({
              title: '操作成功',
              message: `删除任务成功`,
              type: 'success',
              duration: 2000
            })
            this.listLoading = false
            // 重新加载数据
            this.initTask().then(() => {
              console.log('重新加载数据成功')
              this.listLoading = false
            })
            db.close()
          }

          updateRequest.onerror = (event) => {
            this.listLoading = false
            console.error('数据库更新失败:', event.target.error)
            db.close()
          }
        }

        openRequest.onerror = (event) => {
          console.error('数据库打开失败:', event.target.error)
          db.close()
        }
      } catch (e) {
        console.error(e)
        this.listLoading = false
      }
    },
    handleFinish(row, status) {
      if (this.listLoading) return
      this.listLoading = true
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.temp.lastDateTime = moment(this.temp.lastDateTime, 'YYYY-MM-DD HH:mm:ss').toDate()
      this.updateData(status)
    }
  }
}
</script>
