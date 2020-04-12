<template>
  <div class="box">
    <!-- 头部 -->
    <div class="nav">
      <span class="iconfont iconjiantou2" @click="$router.back()"></span>
      <input type="text" v-model="value" @keyup.enter="handleSearch" />
      <span class="searchBtn" @click="handleSearch">搜索</span>
      <span class="iconfont iconsearch"></span>
    </div>
    <div class="history">
      <div class="h-text">
        <p>历史记录</p>
        <span class="iconfont iconicon-test" @click="handleDel"></span>
      </div>
      <div class="h-item">
        <span
          class="h-item"
          v-for="(item,index) in history"
          :key="index"
          @click="handleclick(item)"
        >{{item}}</span>
      </div>
    </div>
    <div class="hot">
      <p>热门搜索</p>
      <div class="item">
        <span>办公室小野否认解散</span>
        <span>月季如何嫁接</span>
      </div>
    </div>
    <div class="cover" v-if="cover">
      <div class="c-item" v-for="(item,index) in list" :key="index">
        <PostItem1 v-if="item.type===1&&item.cover.length<3" :data="item"></PostItem1>
        <PostItem2 v-if="item.type===1&&item.cover.length>3" :data="item"></PostItem2>
        <PostItem3 v-if="item.type===2" :data="item"></PostItem3>
      </div>
    </div>
    <div class="noneCover" v-if="nonecover">
      <p>抱歉,查询不到对应文章</p>
    </div>
  </div>
</template>

<script>
import PostItem1 from "@/components/PostItem1";
import PostItem2 from "@/components/PostItem2";
import PostItem3 from "@/components/PostItem3";

export default {
  name: "search",
  components: {
    PostItem1,
    PostItem2,
    PostItem3
  },
  //   路由拦截从首页进入,清空list和value的值
  beforeRouteEnter(to, from, next) {
    next(vm => {
      // 通过 `vm` 访问组件实例
      if (from.path === "/") {
        vm.list = [];
        vm.cover = false;
        vm.nonecover = false;
        vm.value = "";
      }
    });
  },
  data() {
    return {
      cover: false,
      value: "",
      list: [],
      nonecover: false,
      history: JSON.parse(localStorage.getItem("history")) || []
    };
  },
  watch: {
    value() {
      // 清空输入框隐藏遮罩层
      if (this.value.trim() === "") {
        this.cover = false;
        this.nonecover = false;
        this.list = [];
      }
    }
  },
  methods: {
    handleDel() {
      this.history = [];
      localStorage.removeItem("history");
    },
    handleclick(item) {
      this.value = item;
      this.handleSearch();
    },
    //   搜索
    handleSearch() {
      if (this.value.trim() === "") {
        return;
      }
      this.history.unshift(this.value);
      this.history = [...new Set(this.history)];
      localStorage.setItem("history", JSON.stringify(this.history));
      this.getList();
    },
    getList() {
      this.$axios({
        url: "/post_search",
        params: {
          keyword: this.value
        }
      }).then(res => {
        const { data } = res.data;
        // 搜索成功;
        if (data.length === 0) {
          this.nonecover = true;
          return;
        }
        console.log(data);
        this.cover = true;
        this.list = data;

        console.log(data);
      });
    }
  }
};
</script>

<style scoped lang='less'>
.box {
  padding: 10/360 * 100vw;
}
.nav {
  display: flex;
  position: relative;
  justify-content: space-around;
  align-items: center;
  position: relative;

  input {
    width: 250/360 * 100vw;
    height: 38/360 * 100vw;
    border-radius: 50px;
    border: 1px solid #dddddd;
    padding-left: 40px;
    box-sizing: border-box;
  }
  .searchBtn {
    font-size: 14px;
    margin-right: 5/360 * 100vw;
  }
  .iconsearch {
    position: absolute;
    left: 55px;
    top: 13px;
  }
}
.history {
  font-size: 14px;
  padding: 10/360 * 100vw 20/360 * 100vw;
  border-bottom: 1px solid #dddddd;
  margin-bottom: 20/360 * 100vw;
  .h-text {
    display: flex;
    justify-content: space-between;

    p {
      font-weight: 700;
      margin-bottom: 10px;
    }
  }

  .h-item {
    display: flex;
    flex-wrap: wrap;
    span {
      padding: 0 10/360 * 100vw;
      border: 1px solid #ddd;
      margin: 10px 20px 0 0;
    }
  }
}
.hot {
  p {
    font-size: 14px;
    font-weight: 700;
    margin-bottom: 10/360 * 100vw;
  }
  .item {
    display: flex;
    justify-content: space-around;
    font-size: 14px;
  }
}
.cover {
  position: absolute;
  top: 57px;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #fff;

  .c-item {
    padding: 0 10/360 * 100vw;
    // border-bottom: 1px solid #dddddd;
  }
}
.noneCover {
  position: absolute;
  display: flex;
  justify-content: center;
  padding: 50px 0;
  top: 57px;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #fff;
  p {
    color: #a69b9b;
  }
}
</style>