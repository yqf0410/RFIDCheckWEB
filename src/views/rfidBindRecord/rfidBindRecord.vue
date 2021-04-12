<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.noCode"
        class="filter-item"
        placeholder="单号NO"
        style="width: 150px"
        @keyup.enter.native="handleFilter"
      />
      <el-input
        v-model="listQuery.produCode"
        class="filter-item"
        placeholder="零件代号"
        style="width: 150px"
        @keyup.enter.native="handleFilter"
      />
      <el-select
        v-model="listQuery.bindType"
        class="filter-item"
        placeholder="绑定类型"
        bindType
        clearable
        style="width: 150px"
        @change="handleFilter"
      >
        <el-option
          v-for="item in bindTypeOperations"
          :key="item.key"
          :label="item.value"
          :value="item"
        />
      </el-select>
      <el-date-picker
        v-model="listQuery.bindDateS"
        placeholder="绑定开始时间"
        style="width: 200px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
      <a> - </a>
      <el-date-picker
        v-model="listQuery.bindDateE"
        placeholder="绑定结束时间"
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
        <template slot-scope="scope">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="顺序号" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.seq }}
        </template>
      </el-table-column>
      <el-table-column label="绑定类型" width="150" align="center">
        <template slot-scope="scope">
          {{ bindTypeOperations[scope.row.bindType].value }}
        </template>
      </el-table-column>
      <el-table-column label="单号NO" width="200" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.noCode }}</span>
        </template>
      </el-table-column>
      <el-table-column label="零件编码" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.produCode }}
        </template>
      </el-table-column>
      <el-table-column label="工位号" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.workCellCode }}
        </template>
      </el-table-column>
      <el-table-column label="二维码值串" align="center">
        <template slot-scope="scope">
          {{ scope.row.barCode }}
        </template>
      </el-table-column>
      <el-table-column label="RFID UID" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.rfidUid }}
        </template>
      </el-table-column>
      <el-table-column label="绑定时间" width="150" align="center">
        <template slot-scope="scope">
          {{ scope.row.bindDate }}
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

const bindTypeOperations = [
  { key: "0", value: "手工" },
  { key: "1", value: "扫码" },
];

const bindTypeKeyValue = bindTypeOperations.reduce((acc, cur) => {
  acc[cur.key] = cur.value;
  return acc;
}, {});

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
        noCode: "",
        produCode: "",
        bindType: "",
        bindDateS: "",
        bindDateE: "",
        sort: "-bind_date",
      },
      bindTypeOperations,
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      axios
        .post("http://localhost/pb/selectBindRecord", this.listQuery)
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
        bindType: "",
        bindDateS: "",
        bindDateE: "",
        sort: "-create_date",
      };
      this.getList();
    },
  },
};
</script>
