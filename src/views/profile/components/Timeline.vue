<template>
  <div class="block">
    <el-timeline>
      <el-timeline-item v-for="(item,index) of timeline" :key="index" :timestamp="item.timestamp" placement="top">
        <el-card>
          <h4>{{ item.title }}</h4>
          <p>{{ item.content }}</p>
        </el-card>
      </el-timeline-item>
    </el-timeline>
  </div>
</template>

<script>
export default {
  data() {
    return {
      timeline: [
        {
          timestamp: '2024/6/20',
          title: '毕业啦',
          content: '毕业于 xxx 学校 xxx 专业'
        },
        {
          timestamp: '2022/9/01',
          title: '终于升本了！',
          content: '通过不懈的努力，考上了 xxx 学校 的 xxx 专业，⛽️加油！'
        },
        {
          timestamp: '2022/6/22',
          title: '新的启程',
          content: '大专生涯结束，但这也是我旅程的新开始，继续加油！'
        },
        {
          timestamp: '2019/9/01',
          title: '卧薪尝胆',
          content: '虽然高考没考好，但是我相信能通过三年的努力💪，考上理想的本科！'
        }
      ]
    }
  }
}
</script>
