<template>
  <div class="xtx-pagination">
    <a v-if="myCurrentPage <= 1" href="javascript:;" class="disabled">上一页</a>
    <a @click="changePage(myCurrentPage - 1)" v-else href="javascript:;"
      >上一页</a
    >
    <span v-if="pager.start > 1">...</span>
    <a
      @click="changePage(i)"
      href="javascript:;"
      :class="{ active: i === myCurrentPage }"
      v-for="i in pager.btnArr"
      :key="i"
      >{{ i }}</a
    >
    <span v-if="pager.end < pager.pageCount">...</span>
    <a
      v-if="myCurrentPage >= pager.pageCount"
      href="javascript:;"
      class="disabled"
      >下一页</a
    >
    <a @click="changePage(myCurrentPage + 1)" v-else href="javascript:;"
      >下一页</a
    >
  </div>
</template>
<script>
import { ref, computed, watch } from 'vue'
export default {
  name: 'Pagination',
  // 接收外部数据，总条数，默认显示第几页，每页条数，和按钮个数
  props: {
    total: {
      type: Number,
      default: 100,
    },
    currentPage: {
      type: Number,
      default: 1,
    },
    pageSize: {
      type: Number,
      default: 10,
    },
    btnCount: {
      type: Number,
      default: 5,
    },
  },
  setup(props, { emit }) {
    // 总条数
    const myTotal = ref(100)
    // 每页条数
    const myPageSize = ref(10)
    // 当前第几页
    const myCurrentPage = ref(5)
    // 按钮个数(就是显示多少也前后变为...)
    const btnCount = ref(7)
    // 重点：根据上述数据得到（总页数，起始页码，结束页码，按钮数组）
    const pager = computed(() => {
      // 计算总页数  [总页数也就是总条数/每页显示多少条] 向上取整
      const pageCount = Math.ceil(myTotal.value / myPageSize.value)
      // 计算起始页码和结束页码
      // 1. 理想情况根据当前页码是5，和按钮个数数组可得到
      //   start = 当前页- 总按钮/2
      let start = myCurrentPage.value - Math.floor(btnCount.value / 2) //是5的话除2像下取整再让当前页数减去他就可以计算到开始值
      // ent = 起始页+ 操作按钮 -1 -1是因为多少个按钮中间就需要个多少个
      let end = start + btnCount.value - 1 //结束值就相当于
      // 2.1 如果起始页码小于1了，需要重新计算 //不理想情况下,也就是页码不是五的话例如是1||2
      // 那么 start= 1- 5/2 = -1  end = -1 +5 -1=3  //起始页不可未0
      // 那么 start= 2- 5/2 = 0  end = 0 +5 -1=4 //这样的话就会有问题,因为需要有五个按钮
      if (start < 1) {
        // 所以起始页小于1的时候起始页变为1
        start = 1
        // 结束页就是起始页+按钮个数判断是否大于总页数,如果大于了总页数则直接返回总页数,否则就返回当前页数+按钮个数-1
        //比如当前再第一页,但是一共就4页数据
        end =
          start + btnCount.value - 1 > pageCount
            ? pageCount
            : start + btnCount.value - 1
      }
      // 2.2 如果结束页码大于总页数，需要重新计算
      if (end > pageCount) {
        // 这里判断是如果结束页大于了总页数的话,比如12页,现在已经到了11页,
        // 那么结束也就等于11页
        end = pageCount
        // 起始页页需要变化,例如当前是11页,但是一共12页,那么就是需要距离五个按钮,12-5+1
        start = end - btnCount.value + 1 < 1 ? 1 : end - btnCount.value + 1
      }
      // 处理完毕start和end得到按钮数组
      const btnArr = []
      for (let i = start; i <= end; i++) {
        btnArr.push(i)
      }
      return { pageCount, start, end, btnArr }
    })
    // 改变页码
    const changePage = (newPage) => {
      if (myCurrentPage.value !== newPage) {
        myCurrentPage.value = newPage
        // 通知父组件最新页码
        emit('current-change', newPage)
      }
    }
    // 监听传人的值改变
    watch(
      props,
      () => {
        myTotal.value = props.total
        myPageSize.value = props.pageSize
        myCurrentPage.value = props.currentPage
        btnCount.value = props.btnCount
      },
      { immediate: true }
    )
    return { pager, myCurrentPage, changePage }
  },
}
</script>
<style scoped lang="less">
.xtx-pagination {
  display: flex;
  justify-content: center;
  padding: 30px;
  > a {
    display: inline-block;
    padding: 5px 10px;
    border: 1px solid #e4e4e4;
    border-radius: 4px;
    margin-right: 10px;
    &:hover {
      color: #27ba9b;
    }
    &.active {
      background: #27ba9b;
      color: #fff;
      border-color: #27ba9b;
    }
    &.disabled {
      cursor: not-allowed;
      opacity: 0.4;
      &:hover {
        color: #333;
      }
    }
  }
  > span {
    margin-right: 10px;
  }
}
</style>
