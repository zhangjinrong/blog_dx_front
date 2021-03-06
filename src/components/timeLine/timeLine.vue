<template>
  <div v-show="articleListData.length" class="timeLine">
    <viewTitle :title-text="timeLineName" />
    <div
      v-for="(item, index) of yearsData"
      :key="index"
      class="year-part clearfix"
    >
      <span class="year-num">{{ item.year }}</span>
      <div class="year-list">
        <div v-for="(list, i) of item.list" :key="i" class="list-item">
          <span class="list-date">{{ list.date | formatDate }}</span>
          <span class="list-title">
            <router-link
              :to="{ name: 'article', params: { articleId: list._id } }"
            >
              {{ list.title }}
            </router-link>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import viewTitle from "@/components/viewTitle/viewTitle";
import { GetListByType } from "@/api/article";
import { formatDate } from "@/utils";

export default {
  name: "TimeLine",
  filters: {
    formatDate(time) {
      const date = new Date(time);
      return formatDate(date, "MM/dd");
    }
  },
  components: {
    viewTitle
  },
  props: {
    timeLineName: {
      type: String,
      required: true
    },
    apiName: {
      type: String,
      required: true
    },
    para: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      yearsData: [],
      articleListData: []
    };
  },
  created() {
    this.getListData();
  },
  activated() {
    this.getListData();
  },
  methods: {
    getListData() {
      GetListByType(this.apiName, this.para).then(res => {
        this.articleListData = res.data.resData.list;
        this.$nextTick(function() {
          this.yearsData = [];
          this.getYearsData();
        });
      });
    },
    getYearsData() {
      let count = 0;
      const years = {};
      this.articleListData.forEach(current => {
        const year = new Date(current.date).getFullYear();
        if (years[year] !== 0 && !year[year]) {
          years[year] = count;
          this.yearsData.push({
            year: year,
            list: [
              {
                title: current.title,
                date: current.date,
                _id: current._id
              }
            ]
          });
          count++;
        } else {
          this.yearsData[years[year]].list.push({
            title: current.title,
            date: current.date,
            _id: current._id
          });
        }
      });
    }
  }
};
</script>

<style lang="scss">
.timeLine {
  .year-part {
    text-align: left;
    .year-num {
      float: left;
      margin: 5px 0 0 2px;
      border-bottom: 2px solid $color-base; /*no*/
      font-size: $font-size-sm;
    }
    .year-list {
      padding: 10px 0;
      margin-left: 80px;
      border-left: 1px dashed $color-border; /*no*/
    }
    .list-item {
      display: flex;
      position: relative;
      padding: 0 0 30px 40px;
      &::before {
        content: "";
        position: absolute;
        top: 2px;
        left: -6.4px;
        width: 12px;
        height: 12px;
        border-radius: 100%;
        background: rgba(79, 142, 84, 0.4);
      }
    }
    .list-date {
      flex: 0 0 60px;
    }
    .list-title {
      flex: 1;
    }
  }
}
</style>
