<template>
  <div class="app-container">
    <div class="filter-container">
      <div>
        <el-input v-model="listQuery.name" placeholder="名称" style="width: 200px;margin-right: 8px" class="filter-item"
                  @keyup.enter.native="handleFilter"
        />
        <el-select v-model="listQuery.prio" placeholder="优先级" clearable style="width: 90px;margin-right: 8px"
                   class="filter-item"
        >
          <el-option v-for="item in importanceOptions" :key="item" :label="item" :value="item">
            <el-rate :value="handleRate(item)" :max="3" disabled></el-rate>
          </el-option>
        </el-select>
        <el-select v-model="listQuery.type" placeholder="状态" clearable class="filter-item"
                   style="width: 130px;margin-right: 8px"
        >
          <el-option v-for="item in statusOptions" :key="item" :label="item"
                     :value="item"
          >handleRate
            <el-tag :type="handleStatusColor(row.status)">{{ item }}</el-tag>
          </el-option>
        </el-select>
      </div>
      <!--      <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item" @change="handleFilter">-->
      <!--        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />-->
      <!--      </el-select>-->
      <div>
        <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
          Search
        </el-button>
        <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit"
                   @click="handleCreate"
        >
          Add
        </el-button>
      </div>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <el-table-column label="名称"  width="150px" align="left">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="ID" prop="id" align="left" width="120px">
        <template slot-scope="{row}">
          <span :style="{overflow: 'hidden',textOverflow: 'ellipsis', whiteSpace: 'nowrap'}">{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="任务描述" min-width="200px">
        <template slot-scope="{row}">
          <span :style="{overflow: 'hidden',textOverflow: 'ellipsis', whiteSpace: 'nowrap'}">{{
              row.description
            }}</span>
        </template>
      </el-table-column>
      <el-table-column label="负责人" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.personInCharge }}</span>
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
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit"
                @pagination="getList"
    />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px"
               style="width: 400px; margin-left:50px;"
      >
        <el-form-item label="Type" prop="type">
          <el-select v-model="temp.type" class="filter-item" placeholder="Please select">
            <el-option v-for="item in statusOptions" :key="item.key" :label="item.display_name"
                       :value="item.key"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="Date" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date"/>
        </el-form-item>
        <el-form-item label="Title" prop="title">
          <el-input v-model="temp.title"/>
        </el-form-item>
        <el-form-item label="Status">
          <el-select v-model="temp.status" class="filter-item" placeholder="Please select">
            <el-option v-for="item in statusOptions" :key="item" :label="item" :value="item"/>
          </el-select>
        </el-form-item>
        <el-form-item label="Imp">
          <el-rate v-model="temp.importance" :colors="['#99A9BF', '#F7BA2A', '#FF9900']" :max="3"
                   style="margin-top:8px;"
          />
        </el-form-item>
        <el-form-item label="Remark">
          <el-input v-model="temp.remark" :autosize="{ minRows: 2, maxRows: 4}" type="textarea"
                    placeholder="Please input"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          Confirm
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
import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import Mock from 'mockjs'

export default {
  name: 'TaskList',
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        name: '',
        priority: '',
        status: ''
      },
      importanceOptions: ['低', '中', '高'],
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
      statusOptions: ['待分配', '进行中', '已完成'],
      showReviewer: false,
      temp: {
        id: undefined,
        name: '',
        priority: 1,
        status: '待分配',
        remark: ''
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
    }
  },
  created() {
    this.getList()
  },
  methods: {
    async initTask() {
      try {
        const dbName = 'xdd-server'
        let db
        // 打开数据库获取当前版本
        let openRequest = indexedDB.open(dbName)

        // openRequest.onupgradeneeded = (event) => {
        //   // 如果数据库首次创建，此处可以添加初始对象存储和索引
        //   db = event.target.result
        //   if (!db.objectStoreNames.contains('task')) {
        //     let objectStore = db.createObjectStore('task', { keyPath: 'id' })
        //     objectStore.createIndex('name', 'name', { unique: false })
        //     // 添加其他索引...}
        // }
        openRequest.onsuccess = (event) => {
          db = event.target.result
          // 确认是否需要升级数据库结构，如果是，则进行升级
          // 以下是示例逻辑，实际应用中需要根据实际情况调整
          if (!db.objectStoreNames.contains('task')) {
            // 需要升级，关闭当前数据库，重新打开指定新版本进行升级
            db.close()
            let newVersion = db.version + 1
            let upgradeRequest = indexedDB.open(dbName, newVersion)

            upgradeRequest.onupgradeneeded = (event) => {
              // 执行升级操作
              let db = event.target.result
              let objectStore = db.createObjectStore('task', { keyPath: 'id' })
              // 添加索引...
              objectStore.createIndex('name', 'name', { unique: false })
              // objectStore.createIndex('type', 'type', { unique: false });
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

            upgradeRequest.onsuccess = (event) => {
              db = event.target.result
              this.populateData(db) // 填充数据
            }
          }
        }
      } catch (error) {
        console.error('初始化任务失败:', error)
        return false
      }
    },
    async populateData(db) {
      const transaction = db.transaction(['task'], 'readwrite')
      const objectStore = transaction.objectStore('task')
      // 添加数据...

      for (let i = 0; i < 10; i++) {
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

        const taskData = Mock.mock({
          id: '@guid',
          name: '@ctitle(5, 10)',
          // type: '@cword(3, 8)',
          createTime,
          updateTime,
          description: '@csentence(10, 20)',
          lastDateTime,
          personInCharge: '@cname',
          priority: '@pick(["低", "中", "高"])',
          status: '@pick(["待分配", "进行中", "已完成"])'
        })
        objectStore.add(taskData)
      }

      transaction.oncomplete = () => {
        console.log('任务表初始化完成，已添加10条数据')
      }
      transaction.onerror = (event) => {
        console.error('数据添加失败:', event)
      }
    },
    async getList() {
      this.listLoading = true
      this.initTask().then(async() => {

        try {

          // 打开数据库，不指定版本号以获取当前版本
          const db = await this.openDatabase('xdd-server')
          const transaction = db.transaction(['task'], 'readonly')
          const objectStore = transaction.objectStore('task')

          // 使用异步方式获取所有任务数据
          const request = objectStore.getAll()

          // 等待请求成功
          const result = await new Promise((resolve, reject) => {
            request.onsuccess = (event) => resolve(event.target.result)
            request.onerror = (event) => reject('查询数据失败')
          })

          // 将查询结果集直接赋值给list，并更新总数
          console.log('-> result', result)
          this.list = result
          this.total = result.length
          this.listLoading = false
        } catch (error) {
          console.error(error)
          this.listLoading = false
          this.$message.error('获取列表失败')
        }
      })
    },
    async openDatabase(dbName) {
      return new Promise((resolve, reject) => {
        const request = window.indexedDB.open(dbName)
        request.onerror = () => reject('数据库打开失败')
        request.onsuccess = (event) => resolve(event.target.result)
      })
    },
    handleRate(type) {
      switch (type) {
        case '低':
          return 1
        case '中':
          return 2
        case '高':
          return 3
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
      this.list = this.list.filter((item) => {
        return item.name.indexOf(this.listQuery.name) !== -1 || item.priority.indexOf(this.listQuery.priority) !== -1 || item.status.indexOf(this.listQuery.status) !== -1
      })
      // this.getList()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作Success',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createArticle(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateArticle(tempData).then(() => {
            const index = this.list.findIndex(v => v.id === this.temp.id)
            this.list.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleDelete(row, index) {
      this.$notify({
        title: 'Success',
        message: 'Delete Successfully',
        type: 'success',
        duration: 2000
      })
      this.list.splice(index, 1)
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    formatJson(filterVal) {
      return this.list.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
  }
}
</script>
