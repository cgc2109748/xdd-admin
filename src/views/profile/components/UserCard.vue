<template>
  <el-card style="margin-bottom:20px;">
    <div slot="header" class="clearfix">
      <span>关于我</span>
    </div>

    <div class="user-profile">
      <div class="box-center">
        <pan-thumb :image="user.avatar" :height="'100px'" :width="'100px'" :hoverable="false">
          <div>Hello</div>
          {{ user.name }}
        </pan-thumb>
      </div>
      <div class="box-center">
        <div class="user-name text-center">{{ user.name }}</div>
        <div class="user-role text-center text-muted">{{ rolesHandler(user.role) | uppercaseFirst }}</div>
      </div>
    </div>

    <div class="user-bio">
      <div class="user-education user-bio-section">
        <div class="user-bio-section-header">
          <svg-icon icon-class="education"/>
          <span>个人简介</span></div>
        <div class="user-bio-section-body">
          <div class="text-muted">
            <p>2024 届 xxx 专业应届毕业生</p>
            <p>当下前端开发实习生</p>
            <p>未来可期前端开发工程师</p>
            <p>爱三国、爱生活、爱探索、对未知充满好奇</p>
          </div>
        </div>
      </div>

      <div class="user-skills user-bio-section">
        <div class="user-bio-section-header">
          <svg-icon icon-class="skill"/>
          <span>技能列表</span></div>
        <div class="user-bio-section-body">
          <div class="progress-item">
            <span>Html</span>
            <el-progress :percentage="100" status="success"/>
          </div>
          <div class="progress-item">
            <span>JavaScript</span>
            <el-progress :percentage="60"/>
          </div>
          <div class="progress-item">
            <span>Css</span>
            <el-progress :percentage="80"/>
          </div>
          <div class="progress-item">
            <span>Vue2.0</span>
            <el-progress :percentage="90"/>
          </div>
          <div class="progress-item">
            <span>Vue3.0</span>
            <el-progress :percentage="65"/>
          </div>
          <div class="progress-item">
            <span>React</span>
            <el-progress :percentage="60"/>
          </div>
          <div class="progress-item">
            <span>Node</span>
            <el-progress :percentage="60"/>
          </div>
          <div class="progress-item">
            <span>Express</span>
            <el-progress :percentage="50"/>
          </div>
          <div class="progress-item">
            <span>ESLint</span>
            <el-progress :percentage="50"/>
          </div>
        </div>
      </div>
    </div>
  </el-card>
</template>

<script>
import PanThumb from '@/components/PanThumb'

export default {
  components: { PanThumb },
  props: {
    user: {
      type: Object,
      default: () => {
        return {
          name: '',
          email: '',
          avatar: '',
          role: ''
        }
      }
    }
  },
  methods: {
    rolesHandler(role) {
      if (role === 'admin') {
        return '任务发布者（超级管理员）'
      } else {
        return '任务猎手（普通用户）'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.box-center {
  margin: 0 auto;
  display: table;
}

.text-muted {
  color: #777;
}

.user-profile {
  .user-name {
    font-weight: bold;
  }

  .box-center {
    padding-top: 10px;
  }

  .user-role {
    padding-top: 10px;
    font-weight: 400;
    font-size: 14px;
  }

  .box-social {
    padding-top: 30px;

    .el-table {
      border-top: 1px solid #dfe6ec;
    }
  }

  .user-follow {
    padding-top: 20px;
  }
}

.user-bio {
  margin-top: 20px;
  color: #606266;

  span {
    padding-left: 4px;
  }

  .user-bio-section {
    font-size: 14px;
    padding: 15px 0;

    .user-bio-section-header {
      border-bottom: 1px solid #dfe6ec;
      padding-bottom: 10px;
      margin-bottom: 10px;
      font-weight: bold;
    }
  }
}
</style>
