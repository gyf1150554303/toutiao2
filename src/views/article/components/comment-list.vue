<template>
  <!-- 只有 List 在可视范围内，才会检查滚动位置触发 onLoad -->
  <van-list
    v-model="loading"
    :finished="finished"
    finished-text="没有更多了"
    :error.sync="error"
    error-text="加载失败，请点击重试"
    :immediate-checj="false"
    @load="onLoad"
  >
    <!-- <van-cell
      v-for="(item, index) in list"
      :key="index"
      :title="item.content"
    /> -->
    <comment-item
      v-for="(item, index) in list"
      :key="index"
      :comment="item"
      @reply-click="$emit('reply-click', $event)"
    />
  </van-list>
</template>

<script>
import { getComments } from "@/api/comment";
import CommentItem from "./comment-item";
// http://localhost:8080/#/article/137825
export default {
  name: "CommentList",
  components: {
    CommentItem
  },
  props: {
    source: {
      type: [Number, String, Object],
      required: true
    },
    list: {
      type: Array,
      default: () => [] // 对象给默认值要以函数返回值的方式
    },
    type: {
      type: String,
      validator(value) {
        return ["a", "c"].includes(value);
      },
      default: "a"
    }
  },
  data() {
    return {
      // list: [],
      loading: false,
      finished: false,
      offset: null, // 获取下一页数据的标记
      limit: 10,
      error: false
    };
  },
  created() {
    // 一上来就加载一次，为了显示评论总数
    // 当你手动初始 onLoad 的话，它不会自动开始初始的 loading，所以我们要手动的开启初始 loading
    this.loading = true;
    this.onLoad();
  },
  methods: {
    async onLoad() {
      try {
        // 1. 请求数据
        const { data } = await getComments({
          type: this.type, // 评论类型，a 代表对文章的评论，c 代表评论的回复
          source: this.source.toString(), // 源 id，文章 id 或评论 id
          offset: this.offset,
          limit: this.limit // 获取的个数
        });
        // 2. 将数据添加到列表中
        const { results } = data.data;
        this.list.push(...results);
        // 把总数量传递到父级
        this.$emit("onload-success", data.data);
        // 3. 将 loading 设置为 false
        this.loading = false;
        // 4. 判断是否还有数据
        if (results.length) {
          // 4.1 有就更新获取下一页的数据页码
          this.offset = data.data.last_id;
        } else {
          // 4.2 没有就将 finished 设置结束
          this.finished = true;
        }
      } catch (err) {
        this.error = true;
        this.loading = false;
      }
    }
  }
};
</script>

<style lang="less" scoped></style>
