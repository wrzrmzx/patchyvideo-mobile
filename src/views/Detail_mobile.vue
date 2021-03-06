<!--    vue页面：Detail_mobile.vue     -->
<!--
    组件：网站的视频详情页面
    功能：显示视频的详细信息
    更新日志：
      2/14/2020:
        release
-->

<template>
  <div style="height:100%">
    <!-- 页头 -->
    <PageHead what="视频"></PageHead>

    <!-- 页面正文 -->
    <div class="content" v-loading="loading">
      <!-- 视频封面 -->
      <el-image
        @click="openOriginalWeb(myVideoData.video.item.url)"
        :src="'/images/covers/' + myVideoData.video.item.cover_image"
      >
        <div slot="placeholder" class="image-slot">
          加载中
          <span class="dot">...</span>
        </div>
      </el-image>

      <!-- 视频原链接 -->
      <el-button
        class="openOriginalWeb"
        round
        size="mini"
        type="primary"
        @click="openOriginalWeb(myVideoData.video.item.url)"
      >
        <img
          style="transform: translate(1px, 1px);"
          :src="imgURL"
          height="12px"
        />
        打开原网站
      </el-button>

      <!-- 视频内容 -->
      <div class="videoDesc">
        <!-- 没展开的样式 -->
        <div v-if="!descOpen">
          <!-- 视频标题 -->
          <h3 class="title_not_open">{{ myVideoData.video.item.title }}</h3>
          <!-- 视频上传日期 -->
          <p style="font-weight: 600;">{{ videodate }}</p>
          <!-- 视频详情 -->
          <p class="desc_not_open desc">{{ myVideoData.video.item.desc }}</p>
          <p class="descOpen" @click="descOpen = true">展开全部</p>
        </div>
        <!-- 展开的样式 -->
        <div v-else>
          <!-- 视频标题 -->
          <h3>{{ myVideoData.video.item.title }}</h3>
          <!-- 视频上传日期 -->
          <p style="font-weight: 600;">{{ videodate }}</p>
          <!-- 视频详情 -->
          <p class="desc">{{ myVideoData.video.item.desc }}</p>
          <p class="descOpen" @click="descOpen = false">收起</p>
        </div>
      </div>

      <!-- 视频标签 -->
      <div class="videoTags">
        <h3 class="tagTitle">视频标签</h3>
        <p class="noTag" v-if="myVideoData.tags.length == 0">此视频暂无标签</p>
        <el-tag
          class="tagItems"
          size="small"
          v-for="item in myVideoData.tags"
          :key="item"
          @click="gotoHome(item)"
          >{{ item }}</el-tag
        >
      </div>

      <!-- 视频副本 -->
      <div class="copies">
        <h3 class="tagTitle">副本</h3>
        <p class="copyItem" v-if="myVideoData.copies.length == 0">
          此视频不存在副本
        </p>
        <p v-else>此视频有{{ myVideoData.copies.length }}个副本</p>
        <div
          v-for="(value, key, index) in myVideoData.copies_by_repost_type"
          :key="index"
        >
          <h3 v-if="key == 'official'">原始发布</h3>
          <h3 v-if="key == 'official_repost'">官方再发布</h3>
          <h3 v-if="key == 'authorized_translation'">授权翻译</h3>
          <h3 v-if="key == 'authorized_repost'">授权转载</h3>
          <h3 v-if="key == 'translation'">自发翻译</h3>
          <h3 v-if="key == 'repost'">自发搬运</h3>
          <h3 v-if="key == 'unknown'">其他</h3>
          <ul class="copyItem" v-for="item in value" :key="item._id.$oid">
            <img
              :src="require('../static/img/' + item.item.site + '.png')"
              width="13px"
              style="margin-right:2px"
            />
            <router-link
              :to="{ path: '/video', query: { id: item._id.$oid } }"
              tag="a"
              >{{ item.item.title }}</router-link
            >
          </ul>
        </div>
      </div>

      <!-- 播放列表 -->
      <div class="lists">
        <h3 class="tagTitle">播放列表</h3>
        <p class="listItem" v-if="myVideoData.playlists.length == 0">
          本视频不包含于任何播放列表中
        </p>
        <ul
          class="listItem listItemDetail"
          v-for="item in myVideoData.playlists"
          :key="item._id.$oid"
        >
          <router-link
            v-if="item.prev != ''"
            :to="{ path: '/video', query: { id: item.prev } }"
            tag="a"
            >【前一篇】</router-link
          >
          <span v-else>【没有啦】</span>
          <router-link
            :to="{ path: '/listdetail', query: { id: item._id.$oid } }"
            tag="a"
            class="listItemTitle"
            >{{ item.item.title }}</router-link
          >
          <router-link
            v-if="item.next != ''"
            :to="{ path: '/video', query: { id: item.next } }"
            tag="a"
            style="float:right"
            >【后一篇】</router-link
          >
          <span v-else style="float:right">【没有啦】</span>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import PageHead from "../components/PageHead";
export default {
  data() {
    return {
      // 视频的详细信息
      myVideoData: {
        // 视频的副本列表
        copies: [],
        // 视频的播放列表
        playlists: [],
        // 视频的标签列表(已分类)
        tag_by_category: {},
        // 视频的标签列表(未分类)
        tags: [],
        video: {
          item: {
            // 视频的标题
            title: "",
            // 视频介绍
            desc: "",
            // 视频上传时间(时间戳对象)
            upload_time: {
              $date: "",
            },
            // 视频的链接
            url: "",
            //视频封面
            cover_image: "",
            // 视频的来源网站
            site: "",
          },
        },
      },
      // 视频列表是否属于加载状态的判断
      loading: false,
      // 视频详情是否打开的判断
      descOpen: false,
    };
  },
  computed: {
    videoType() {
      switch (this.myVideoData.video.item.repost_type) {
        case "official":
          return "原始发布";
          break;
        case "official_repost":
          return "官方再发布";
          break;
        case "authorized_translation":
          return "授权翻译";
          break;
        case "authorized_repost":
          return "授权转载";
          break;
        case "translation":
          return "自发翻译";
          break;
        case "repost":
          return "自发搬运";
          break;
        case "unknown":
          return "其他";
          break;
      }
    },
    // 视频的上传日期
    videodate() {
      var upload_time = new Date(this.myVideoData.video.item.upload_time.$date);
      var y = upload_time.getFullYear(); //getFullYear方法以四位数字返回年份
      var M = upload_time.getMonth() + 1; // getMonth方法从 Date 对象返回月份 (0 ~ 11)，返回结果需要手动加一
      var d = upload_time.getDate(); // getDate方法从 Date 对象返回一个月中的某一天 (1 ~ 31)
      var h = upload_time.getHours(); // getHours方法返回 Date 对象的小时 (0 ~ 23)
      var m = upload_time.getMinutes(); // getMinutes方法返回 Date 对象的分钟 (0 ~ 59)
      var s = upload_time.getSeconds(); // getSeconds方法返回 Date 对象的秒数 (0 ~ 59)
      return (
        "发布于 " +
        y +
        "-" +
        // 数字不足两位自动补零，下同
        (Array(2).join(0) + M).slice(-2) +
        "-" +
        (Array(2).join(0) + d).slice(-2) +
        " " +
        (Array(2).join(0) + h).slice(-2) +
        ":" +
        (Array(2).join(0) + m).slice(-2) +
        ":" +
        (Array(2).join(0) + s).slice(-2) +
        " GMT+8"
      );
    },
    imgURL() {
      if (
        this.myVideoData.video.item.site == "" ||
        this.myVideoData.video.item.site == "ipfs"
      ) {
        return require("../static/img/defaultAvatar.jpg");
      }
      var imgName = this.myVideoData.video.item.site + "_letter";
      return require("../static/img/" + imgName + ".png");
    },
    // 判断是否登录的标志
    isLogin() {
      if (
        JSON.stringify(this.$store.state.username) != "null" &&
        this.$store.state.username != ""
      ) {
        return true;
      } else {
        return false;
      }
    },
  },
  created() {
    this.searchVideo();
  },
  mounted() {
    // 防止B站侦测ferrer导致视频链接跳转出现404
    $("head").append('<meta name="referrer" content="never">');
  },
  methods: {
    // 打开原网站
    openOriginalWeb(url) {
      window.open(url, "_self");
    },
    // 点击标签跳转到home页面显示搜索结果
    gotoHome(query) {
      this.$router
        .push({ path: "/home", query: { keyword: query } })
        .catch((err) => {
          return err;
        });
    },
    // 查询视频详细信息
    searchVideo: function() {
      this.loading = true;

      // 直接向后端请求视频数据
      this.axios({
        method: "post",
        url: "be/getvideo.do",
        data: {
          vid: this.$route.query.id,
          lang: "CHS",
        },
      })
        .then((result) => {
          this.myVideoData = result.data.data;

          // 修改网站标题
          document.title = this.myVideoData.video.item.title;
          this.pid = this.myVideoData.video._id.$oid;
          // 视频pid储存到vuex中
          this.$store.commit("setVideoPid", this.myVideoData.video._id.$oid);
          // 加载结束,加载动画消失
          this.loading = false;
        })
        .catch((error) => {
          this.$router.push({ path: "/404" });
        });
    },
  },
  watch: {
    // 监控视频ID变化情况
    $route(newV, oldV) {
      if (newV.query.id != oldV.query.id) {
        this.searchVideo();
      }
    },
  },
  components: { PageHead },
};
</script>

<style scoped>
.content {
  z-index: 10;
  margin-top: 37px;
  width: 100%;
  min-height: 100%;
}
.content .el-image {
  width: 100%;
  display: block;
  margin: 0 auto;
}

.openOriginalWeb {
  width: 85%;
  margin: 5px;
}

.videoDesc {
  font-size: 12px;
  word-wrap: break-word;
  white-space: pre-wrap;
  text-align: left;
  width: 90%;
  margin: 0 auto;
}
.videoDesc_shadow {
  box-shadow: inset 0px -3px 2px rgba(0, 0, 0, 0.13);
}
.videoDesc h3 {
  width: 100%;
  margin: 0 auto;
  font-size: 18px;
  text-align: left;
  margin-bottom: 2px;
}
.title_not_open {
  /* 使文字变为最多显示2行，多余的使用省略号代替 */
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
.desc_not_open {
  /* 使文字变为最多显示2行，多余的使用省略号代替 */
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
.descOpen {
  text-align: right;
  color: #409eff;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #a3a3a327;
}

.videoTags {
  width: 90%;
  margin: 0 auto;
  text-align: left;
}
.tagTitle {
  width: 100%;
  font-size: 18px;
  margin-bottom: 2px;
  border-bottom-style: solid;
  border-bottom-width: 1px;
  border-bottom-color: #21c6ef;
}
.noTag {
  font-size: 13px;
  margin-bottom: 5px;
}
.tagItems {
  margin-right: 5px;
  margin-bottom: 5px;
}

.copies {
  width: 90%;
  margin: 0 auto;
  margin-bottom: 5px;
  text-align: left;
}
.copyItem {
  font-size: 13px;
}

.lists {
  width: 90%;
  margin: 0 auto;
  margin-bottom: 5px;
  text-align: left;
}
.listItem {
  font-size: 13px;
}
.listItemDetail {
  display: flex;
  display: -webkit-flex;
  justify-content: space-between;
  white-space: nowrap;
}
.listItemTitle {
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
