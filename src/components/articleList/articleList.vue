<template>
  <div class="articleList">
    <div v-if="!articleListData.length" class="msg">
      {{ msg }}
    </div>

    <div
      v-for="(item, index) in articleListData"
      :key="index"
      class="list-item"
    >
      <div class="item-title">
        <router-link :to="{ name: 'article', params: { articleId: item._id } }">
          {{ item.title }}
        </router-link>
      </div>

      <div class="item-date">
        <svg-icon icon-class="date" class-name="icon" />
        <span class="text">{{ item.date | formatDate }}</span>
      </div>
      <router-link :to="{ name: 'article', params: { articleId: item._id } }">
        <div class="item-abstract">
          {{ item.abstract }}
        </div>
      </router-link>
      <div class="item-info clearfix">
        <div v-for="(tag, i) in item.tags" :key="i" class="item-tags">
          <svg-icon icon-class="tag" class-name="icon" />
          <router-link
            class="text"
            :to="{ name: 'tagArchive', params: { tag: tag } }"
          >
            {{ tag }}
          </router-link>
        </div>
        <div class="item-readAndComment">
          <svg-icon icon-class="comment" class-name="icon" />
          <span class="text">{{ item.commentCount }}</span>

          <svg-icon icon-class="read" class-name="icon" />
          <span class="text">{{ item.readCount }}</span>
        </div>
      </div>
    </div>

    <div class="loadMsg">
      {{ isLoadOver ? "没有更多了" : "加载中..." }}
    </div>
  </div>
</template>

<script>
import { GetList } from "@/api/article";
import { formatDate } from "@/utils";

export default {
  name: "ArticleList",
  filters: {
    formatDate(time) {
      const date = new Date(time);
      return formatDate(date, "yyyy-MM-dd hh:mm");
    }
  },
  data() {
    return {
      msg: "树苗种植中...",
      articleListData: [],
      page: 1,
      pageSize: 10,
      isInLoading: true,
      isLoadOver: false
    };
  },
  created() {
    this.getArticlesList(1);
  },
  mounted() {
    window.addEventListener("scroll", this.onScroll);
  },
  activated() {
    window.addEventListener("scroll", this.onScroll);
  },
  deactivated() {
    window.removeEventListener("scroll", this.onScroll);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.onScroll);
  },
  methods: {
    getArticlesList() {
      const param = {
        page: this.page,
        pageSize: this.pageSize
      };

      GetList(param).then(res => {
        const list = res.data.resData.list;
        // 更新页面数据
        const newList = this.articleListData.slice();
        newList.push(...list);
        this.articleListData = newList;
        // 检查是否全部加载完
        if (list.length < this.pageSize) {
          this.isLoadOver = true;
        }
        // 如果加载失败
        !this.articleListData.length &&
          (this.msg = "网络出错啦！这里什么都没有！");
        this.page++;
        this.isInLoading = false;
      });
    },
    onScroll() {
      if (this.isInLoading || this.isLoadOver) {
        return;
      }
      const pageHeight =
        document.documentElement.scrollHeight || document.body.scrollHeight;
      const scrollTop =
        document.documentElement.scrollTop || document.body.scrollTop;
      const innerHeight =
        window.innerHeight || document.documentElement.clientHeight;
      if (pageHeight - (scrollTop + innerHeight) < 60) {
        this.isInLoading = true;
        this.getArticlesList();
      }
    }
  }
};
</script>

<style lang="scss" rel="stylesheet/scss">
.articleList {
  .msg {
    margin: 40px 0;
    font-size: $font-size-md;
    color: $color-link;
  }

  .ivu-icon {
    vertical-align: top;
  }

  .list-item {
    margin: 0 auto;
    padding: 40px 0;
    text-align: left;
    border-bottom: 1px solid rgba(0, 0, 0, 0.2); /*no*/

    &:nth-last-child(2) {
      border-bottom: 0;
    }
  }

  .item-title {
    margin-bottom: 16px;
    line-height: 3;
    font-size: $font-size-md;
    font-weight: bold;
  }

  .item-date {
    font-size: $font-size-xs;
    line-height: 1;
    color: $color-tint;

    .icon {
      margin-right: 6px;
    }
  }

  .item-abstract {
    margin: 20px 0;
    font-size: $font-size-sm;
    line-height: 1.8;
    color: $color-base;
  }

  .item-info {
    line-height: 1;
  }

  .item-tags {
    float: left;
    margin-right: 12px;
    cursor: pointer;

    .icon {
      color: $color-tint;
      margin-right: 2px;
    }

    .text {
      color: #c3b74e;
    }
  }

  .item-readAndComment {
    margin-right: 4px;
    float: right;
    color: $color-tint;
    .icon {
      position: relative;
      // bottom: -1px;
      margin-right: 6px;
      margin-left: 12px;
      font-size: 16px;
    }
    .icon-read {
      margin-top: -3px;
      font-size: 20px;
    }
  }

  .loadMsg {
    padding: 6px 0;
    font-size: $font-size-xs;
    color: $color-tint;
  }
}
</style>
