<template>
  <div>
    <div style="margin-bottom:15px;">
<!--      当前角色: {{ roles }}-->
      当前角色: {{ rolesHandler(roles) }}
    </div>
    切换角色:
    <el-radio-group v-model="switchRoles">
      <el-radio-button label="editor" />
      <el-radio-button label="admin" />
<!--      <el-radio-button label="editor">任务猎手（普通用户）</el-radio-button>-->
<!--      <el-radio-button label="admin" >任务发布者（超级管理员）</el-radio-button>-->
    </el-radio-group>
  </div>
</template>

<script>
export default {
  computed: {
    roles() {
      return this.$store.getters.roles
    },
    switchRoles: {
      get() {
        return this.roles[0]
      },
      set(val) {
        console.log("-> val", val);
        this.$store.dispatch('user/changeRoles', val).then(() => {
          this.$emit('change')
        })
      }
    }
  },
  methods: {
    rolesHandler(roles) {
      if (roles[0] === 'admin') {
        return '任务发布者（超级管理员）'
      } else {
        return '任务猎手（普通用户）'
      }
    }
  }
}
</script>
