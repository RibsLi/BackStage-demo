<template>
  <el-table
    :data="songs"
    stripe
    highlight-current-row
    @row-dblclick="playMusic"
    size="mini"
    empty-text="Loading..."
    style="width: 100%"
  >
    <el-table-column
      type="index"
      label=" "
      header-align="center"
      align="center"
    />
    <el-table-column prop="" label="操作" width="80">
      <template v-slot:default="scope">
        <i class="iconfont icon-shoucang" @click="likeMusic(scope.row.id)"></i>
        <i class="iconfont icon-xiazai"></i>
      </template>
    </el-table-column>
    <el-table-column prop="name" label="标题" :show-overflow-tooltip="true">
      <template v-slot:default="scope">
        <span class="singer" @click="songClick(scope.row.id)">{{ scope.row.name}}</span>
      </template>
    </el-table-column>
    <el-table-column prop="ar" label="歌手" :show-overflow-tooltip="true">
      <template v-slot:default="scope">
        <span class="singer" v-for="item in scope.row.ar" :key="item" @click="singerClick(item.id)">{{ item.name }} &nbsp;</span>
      </template>
    </el-table-column>
    <el-table-column prop="al.name" label="专辑" :show-overflow-tooltip="true">
      <template v-slot:default="scope">
        <span class="singer">{{ scope.row.al.name}}</span>
      </template>
    </el-table-column>
    <el-table-column prop="dt" label="时间" width="80">
      <template v-slot:default="scope">
        {{ playTime(scope.row.dt) }}
      </template>
    </el-table-column>
    <el-table-column prop="pop" label="热度" width="150">
      <template v-slot:default="scope">
        <el-progress :percentage="scope.row.pop" :show-text="false" />
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import { formatDuration } from "common/utils";

export default {
  name: "TableData",
  props: {
    songs: {
      type: Array,
      default() {
        return []
      }
    }
  },
  methods: {
    // 歌曲时长处理函数
    playTime(dt) {
      return formatDuration(dt);
    },
    singerClick(id) {
      // console.log(id);
      this.$router.push({
        path: "/singerDetail",
        query: {
          id
        },
      });
    },
    playMusic(row) {
      this.$emit('songClick', row.id)
    },
    songClick(id) {
      this.$emit('songClick', id)
    },
    likeMusic(id) {
      this.$emit('likeMusic', id)
    }
  },
};
</script>

<style lang="less" scoped>
.iconfont {
  margin: 0 5px;
  cursor: pointer;
}
.singer {
  cursor: pointer;
  &:hover {
    color: #409eff;
  }
}
</style>