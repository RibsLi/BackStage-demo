<template>
  <el-table
    :data="$store.state.serachData"
    stripe
    highlight-current-row
    @row-dblclick="playMusic"
    size="mini"
    style="width: 100%"
  >
    <el-table-column
      type="index"
      label=" "
      header-align="center"
      align="center"
    />
    <el-table-column prop="" label="操作" width="80">
      <i class="iconfont icon-shoucang"></i>
      <i class="iconfont icon-xiazai"></i>
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
        <span class="singer" @click="albumClick(scope.row.al.id)">{{ scope.row.al.name}}</span>
      </template>
    </el-table-column>
    <el-table-column prop="dt" label="时间" width="100">
      <template v-slot:default="scope">
        {{ playTime(scope.row.dt) }}
      </template>
    </el-table-column>
    <el-table-column prop="pop" label="热度" width="150" sortable>
      <template v-slot:default="scope">
        <el-progress :percentage="scope.row.pop" :show-text="false" />
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import { formatDuration } from "common/utils";

export default {
  name: "SongData",
  props: {
    songTableData: {
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
    // 表格双击传入改行id
    playMusic(row) {
      // console.log(row);
      // console.log(row.id);
      // this.$emit('songsClick')
      this.$emit('songClick', row.id)
    },
    songClick(id) {
      this.$emit('songClick', id)
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
    albumClick(id) {
      // console.log('---');
      this.$router.push({
        path: "/albumDetail",
        query: {
          id
        },
      })
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