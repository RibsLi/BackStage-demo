<template>
  <div class="detail">
    <!-- 详情头部 -->
    <detail-header :playlist="playlist" :creator="creator" @songsClick="songsClick" />
    <!-- tabs标签页 -->
    <el-tabs v-model="activeName" @tab-click="tabClick">
      <!-- 歌单列表 -->
      <el-tab-pane :label="'歌曲列表' + '('+ tableData.length + ')'" name="list">
        <table-data :tableData="tableData" @songClick="songClick"/>
      </el-tab-pane>
      <!-- 评论 -->
      <!-- <el-tab-pane :label="'评论 ' + '('+ comTotal + ')'" name="comment"> -->
      <el-tab-pane label="评论" name="comment">
        <!-- 输入框 -->
        <el-input
          maxlength="140"
          show-word-limit
          resize="none"
          v-model="submitInfo.content"
          :autosize="{ minRows: 3, maxRows: 3 }"
          type="textarea"
          placeholder="请输入内容"
        />
        <el-button round size="mini" @click="submitClick">评论</el-button>
        <div class="clearfix"></div>
        <!-- 精彩评论 -->
        <comment :comments="hotComments" :title="hotComments.length ? '精彩评论 ' + '(' + hotComments.length + ')' : ''" v-show="comment.offset == 0" />
        <!-- <comment :comments="hotComments" :title="hotComments.length ? '精彩评论' : ''" v-show="comment.offset == 0" /> -->
        <!-- 最新评论 -->
        <comment :comments="comments" :title="comments.length ? '最新评论 ' + '('+ comTotal + ')' : ''" />
        <!-- <comment :comments="comments" title="最新评论" /> -->
        <el-pagination
          background
          hide-on-single-page
          :current-page="comment.pagenum"
          :page-size="comment.limit"
          layout="prev, pager, next"
          :total="comTotal"
          @current-change="comCurrentChange"
        ></el-pagination>
      </el-tab-pane>
      <!-- 收藏 -->
      <!-- <el-tab-pane :label="'收藏者 ' + '('+ subTotal + ')'" name="collector"> -->
      <el-tab-pane label="收藏者" name="collector">
        <collect :subscribers="subscribers" />
        <el-pagination
          background
          hide-on-single-page
          :current-page="comment.pagenum"
          :page-size="comment.limit"
          layout="prev, pager, next"
          :total="subTotal"
          @current-change="subCurrentChange"
        ></el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import {
  getPlaylistDetailCookie,
  getSongDetail,
  getComment,
  getHotComment,
  getSubscribers,
} from "network/songdetail";
import { submitComment } from "network/user"
import DetailHeader from "./childComps/DetailHeader";
import TableData from "./childComps/TableData";
import Comment from "./childComps/Comment";
import Collect from "./childComps/Collect";

export default {
  name: "MyLike",
  data() {
    return {
      id: "",
      cookie: window.sessionStorage.getItem('cookie'),
      playlist: {},
      creator: {},
      activeName: "list",
      trackIds: [],
      tableData: [],
      comment: {
        id: this.$route.query.id,
        type: 2,
        limit: 20,
        offset: 0,
        parentCommentId: '',
        pagenum: 1
      },
      comments: [],
      hotComments: [],
      subscribers: [],
      comTotal: 0,
      subTotal: 0,
      songsUrl: [],
      submitInfo: {
        id: this.$route.query.id,
        t: 1,
        type: 2,
        content: '',
        cookie: window.sessionStorage.getItem('cookie'),
      },
    };
  },
  components: {
    DetailHeader,
    TableData,
    Comment,
    Collect,
  },
  created() {
    this.id = this.$route.query.id;
    this.getPlaylistDetailCookie();
    this.getHotComment();
  },
  methods: {
    // 获取歌单详情
    getPlaylistDetailCookie() {
      getPlaylistDetailCookie(this.id, this.cookie).then((res) => {
        // console.log(res);
        this.playlist = res.data.playlist;
        this.creator = res.data.playlist.creator;
        // 循环遍历获取所有id请求所有歌曲
        // const trackIds = [];
        res.data.playlist.trackIds.forEach((item) => {
          this.trackIds.push(item.id);
        });
        getSongDetail(this.trackIds).then((res) => {
          // console.log(res);
          this.tableData = res.data.songs;
        });
      });
    },
    // 获取热门评论
    getHotComment() {
      getHotComment(this.comment).then((res) => {
        // console.log(res);
        this.hotComments = res.data.hotComments;
      });
    },
    // 获取最新评论
    getComment() {
      getComment(this.comment).then((res) => {
        // console.log(res);
        this.comments = res.data.comments;
        this.comTotal = res.data.total;
      });
    },
    // 获取收藏
    getSubscribers() {
      getSubscribers(this.comment).then((res) => {
        // console.log(res);
        this.subscribers = res.data.subscribers;
        this.subTotal = res.data.total;
      });
    },
    // tabs标签点击切换事件
    tabClick() {
      if (this.activeName == "comment") {
        this.comment.pagenum = 1
        this.comment.offset = 0
        this.getComment();
      } else if (this.activeName == "collector") {
        this.comment.pagenum = 1
        this.comment.offset = 0
        this.getSubscribers();
      }
    },
    // 页码改变时执行的事件
    comCurrentChange(newPage) {
      this.comment.pagenum = newPage
      this.comment.offset = (newPage - 1) * this.comment.limit;
      this.getComment();
    },
    subCurrentChange(newPage) {
      this.comment.pagenum = newPage
      this.comment.offset = (newPage - 1) * this.comment.limit;
      this.getSubscribers();
    },
    // 提交评论信息
    submitClick() {
      if (!this.submitInfo.content) return this.$message.warning('先填写一些评论吧')
      submitComment(this.submitInfo).then(res => {
        // console.log(res);
        if (res.data.code === 200) {
          this.getComment();
          this.submitInfo.content = ''
          return this.$message.success('评论成功')
        }
      })
    },
    songsClick() {
      // this.$store.commit("addSongId", this.trackIds)
      this.$store.commit("subSongDetail", this.tableData)
      // this.$store.dispatch("addSong")
    },
    //获取单首音乐
    songClick(id) {
      // this.$store.dispatch("addSong", id)
      // const ids = []
      // ids.push(id)
      // this.$store.commit("addSongId", ids)
      getSongDetail(id).then(res => {
        // console.log(res);
        this.$store.commit("addSongDetail", res.data.songs)
      })
    }
  },
};
</script>

<style lang="less" scoped>
.el-button {
  float: right;
  margin: 10px 0;
}
.el-tabs {
  margin-top: 15px;
}
</style>