<template>
  <div class="app-container">
    <div class="filter-container">
      <el-select
        v-model="listQuery.taskName"
        class="filter-item"
        placeholder="接口名称"
        taskName
        clearable
        style="width: 200px"
        @change="handleFilter"
      >
        <el-option
          v-for="item in taskNameOperations"
          :key="item.key"
          :label="item.value"
          :value="item"
        />
      </el-select>
      <el-date-picker
        v-model="listQuery.requestDateS"
        placeholder="请求开始时间"
        style="width: 200px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
      <a> - </a>
      <el-date-picker
        v-model="listQuery.requestDateE"
        placeholder="请求结束时间"
        style="width: 200px; margin-right: 40px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />

      <el-date-picker
        v-model="listQuery.responseDateS"
        placeholder="返回开始时间"
        style="width: 200px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
      <a> - </a>
      <el-date-picker
        v-model="listQuery.responseDateE"
        placeholder="返回结束时间"
        style="width: 200px; margin-right: 40px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleRefresh"
      >
        刷新
      </el-button>
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >
        查询
      </el-button>
    </div>

    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="序号" width="95">
        <template slot-scope="scope">
          {{ scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column label="ID" v-if="false">
        <template slot-scope="scope" width="250" align="center">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="接口名称" width="250" align="center">
        <template slot-scope="scope">
          {{ scope.row.name }}
        </template>
      </el-table-column>
      <el-table-column label="状态" width="250" align="center">
        <template slot-scope="scope">
          {{ flagOperations[scope.row.flag].value }}
        </template>
      </el-table-column>
      <el-table-column label="请求时间" width="250" align="center">
        <template slot-scope="scope">
          {{ scope.row.requestDate }}
        </template>
      </el-table-column>
      <el-table-column label="返回时间" width="250" align="center">
        <template slot-scope="scope">
          {{ scope.row.responseDate }}
        </template>
      </el-table-column>
      <el-table-column label="结果" width="250" align="center">
        <template slot-scope="scope">
          {{ scope.row.message }}
        </template>
      </el-table-column>
      <el-table-column label="数据" width="300" align="center">
        <template slot-scope="scope">
          {{ scope.row.data }}
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
  </div>
</template>
<style scoped>
.filter-item {
  margin: 0 10px 15px 0;
}
</style>
<script>
import Pagination from "@/components/Pagination";
import axios from "axios";

const taskNameOperations = [{ key: "校验RFID信息", value: "校验RFID信息" }];

const taskNameKeyValue = taskNameOperations.reduce((acc, cur) => {
  acc[cur.key] = cur.value;
  return acc;
}, {});

const flagOperations = [
  { key: "0", value: "失败" },
  { key: "1", value: "成功" },
];

export default {
  components: { Pagination },
  data() {
    return {
      list: null,
      listLoading: true,
      total: 0,
      listQuery: {
        page: 1,
        limit: 10,
        taskName: "",
        requestDateS: "",
        requestDateE: "",
        responseDateS: "",
        responseDateE: "",
        sort: "-request_date",
      },
      taskNameOperations,
      flagOperations,
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      axios
        .post("http://localhost/tl/select", this.listQuery)
        .then((resData) => {
          this.list = resData.data.items;
          this.total = resData.data.total;
          this.listLoading = false;
        })
        .catch((err) => {
          this.$notify({
            title: "失败",
            message: err.data,
            type: "faild",
            duration: 2000,
          });
          this.listLoading = false;
        });
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },
    handleRefresh() {
      this.listQuery = {
        page: 1,
        limit: 10,
        noCode: "",
        produCode: "",
        taskName: "",
        requestDateS: "",
        requestDateE: "",
        sort: "-create_date",
      };
      this.getList();
    },
  },
};
</script>
