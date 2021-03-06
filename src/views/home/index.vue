<template>
  <div class="home-container">
    <!-- 导航栏 -->
    <!-- #1 -->
    <van-nav-bar class="page-nav-bar" fixed>
      <van-button
        class="search-btn"
        slot="title"
        type="info"
        size="small"
        round
        icon="search"
        to="/search"
        >搜索</van-button
      >
    </van-nav-bar>
    <!-- 频道列表 -->
    <van-tabs class="channel-tabs" v-model="active" animated swipeable>
      <van-tab
        v-for="channel in channels"
        :key="channel.id"
        :title="channel.name"
        ><article-list :channel="channel"></article-list
      ></van-tab>
      <div slot="nav-right" class="placeholder"></div>
      <div
        slot="nav-right"
        class="hamburger-btn"
        @click="isChannelEditShow = true"
      >
        <i class="iconfont icongengduo"></i>
      </div>
    </van-tabs>
    <!-- 频道编辑弹出层 -->
    <van-popup
      v-model="isChannelEditShow"
      closeable
      position="bottom"
      close-icon-position="top-left"
      :style="{ height: '100%' }"
    >
      <channel-edit
        :my-channels="channels"
        :active="active"
        @update-active="onUpdateActive"
      />
    </van-popup>
  </div>
</template>

<script>
import { getUserChannels } from "@/api/user";
import ArticleList from "./components/article-list";
import ChannelEdit from "./components/channel-edit";
import { mapState } from "vuex";
import { getItem } from "@/utils/storage";

export default {
  name: "HomeIndex",
  components: {
    ArticleList,
    ChannelEdit,
  },
  data() {
    return {
      active: 0,
      channels: [],
      isChannelEditShow: false, // 控制频道编辑弹出层的显示
    };
  },
  computed: {
    ...mapState(["user"]),
  },
  created() {
    this.loadChannels();
  },
  methods: {
    async loadChannels() {
      try {
        let channels = [];
        // 如果登录了
        if (this.user) {
          const { data } = await getUserChannels();
          channels = data.data.channels;
        } else {
          // 未登录
          const localChannels = getItem("TOUTIAO_CHANNELS");
          if (localChannels) {
            // 本地有，直接用
            channels = localChannels;
          } else {
            // 本地没有，未登录则会获取默认频道列表
            const { data } = await getUserChannels();
            channels = data.data.channels;
          }
        }
        this.channels = channels;
      } catch (err) {
        this.$toast("获取频道数据失败");
      }
    },
    onUpdateActive(index, isChannelEditShow = true) {
      this.active = index;
      this.isChannelEditShow = isChannelEditShow; // 关闭弹层
    },
  },
};
</script>

<style lang="less" scoped>
// 当前组件中加了 scoped 对内部样式的修改需要加 /deep/，或者去掉 scoped
.home-container {
  // #3
  padding-top: 174px;
  padding-bottom: 100px;
  /deep/ .van-nav-bar__title {
    max-width: unset;
  }
  .search-btn {
    width: 555px;
    height: 64px;
    background-color: #5babfb;
    border: none;
    font-size: 28px;
    .van-icon {
      font-size: 32px;
    }
  }
  /deep/ .channel-tabs {
    // #2
    .van-tabs__wrap {
      position: fixed;
      top: 92px;
      left: 0;
      right: 0;
      z-index: 1;
      height: 82px;
    }
    // Tab 标签页
    .van-tab {
      border-right: 1px solid #edeff3;
      min-width: 200px;
      font-size: 30px;
      color: #777777;
    }
    .van-tab--active {
      color: #333333;
    }
    .van-tabs__nav {
      padding-bottom: 0;
    }
    .van-tabs__line {
      bottom: 8px;
      width: 31px !important;
      height: 6px;
      background-color: #3296fa;
    }
    // 汉堡
    .placeholder {
      flex-shrink: 0;
      width: 66px;
      height: 82px;
    }
    .hamburger-btn {
      position: fixed;
      right: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 66px;
      height: 82px;
      background-color: #fff;
      background-color: rgba(255, 255, 255, 0.902);
      i.iconfont {
        font-size: 33px;
      }
      &:before {
        content: "";
        position: absolute;
        left: 0;
        width: 1px;
        height: 58px;
        background-image: url(~@/assets/gradient-gray-line.png);
        background-size: contain;
      }
    }
  }
}
</style>
