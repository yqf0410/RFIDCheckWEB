<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.workCellCode"
        class="filter-item"
        placeholder="工位号"
        style="width: 200px"
        @keyup.enter.native="handleFilter"
      />
      <el-input
        v-model="listQuery.produCode"
        class="filter-item"
        placeholder="零件代号"
        style="width: 200px"
        @keyup.enter.native="handleFilter"
      />
      <el-select
        v-model="listQuery.checkResult"
        class="filter-item"
        placeholder="校验结果"
        checkResult
        clearable
        style="width: 200px"
        @change="handleFilter"
      >
        <el-option
          v-for="item in checkResultOperations"
          :key="item.key"
          :label="item.value"
          :value="item"
        />
      </el-select>
      <el-date-picker
        v-model="listQuery.checkDateS"
        placeholder="读取开始时间"
        style="width: 200px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
      <a> - </a>
      <el-date-picker
        v-model="listQuery.checkDateE"
        placeholder="读取结束时间"
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
      <el-checkbox-button v-model="autoRefresh" @change="handleAutoRefresh">
        自动刷新（每秒）
      </el-checkbox-button>
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
        <template slot-scope="scope" width="100" align="center">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="读取时间" width="150" align="center">
        <template slot-scope="scope">
          {{ scope.row.checkDate }}
        </template>
      </el-table-column>
      <el-table-column label="读取工位" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.checkWorkCell }}
        </template>
      </el-table-column>
      <!-- <el-table-column label="读取零件编码" width="150" align="center">
        <template slot-scope="scope">
          {{ scope.row.checkProduCode }}
        </template>
      </el-table-column> -->
      <el-table-column label="校验RFID" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.checkRfidUid }}
        </template>
      </el-table-column>
      <el-table-column label="校验状态" width="100" align="center">
        <template slot-scope="scope">
          {{ checkStateOperations[scope.row.checkState].value }}
        </template>
      </el-table-column>
      <el-table-column label="校验结果" align="center">
        <template slot-scope="scope">
          {{ checkResultOperations[scope.row.checkResult].value }}
        </template>
      </el-table-column>

      <el-table-column label="顺序号" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.seq }}
        </template>
      </el-table-column>
      <!-- <el-table-column label="绑定类型" width="100" align="center">
        <template slot-scope="scope">
          {{ bindTypeOperations[scope.row.bindType].value }}
        </template>
      </el-table-column> -->
      <el-table-column label="绑定零件编码" width="150" align="center">
        <template slot-scope="scope">
          {{ scope.row.produCode }}
        </template>
      </el-table-column>
      <el-table-column label="绑定RFID" width="100" align="center">
        <template slot-scope="scope">
          {{ scope.row.rfidUid }}
        </template>
      </el-table-column>
      <el-table-column label="绑定时间" width="200" align="center">
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

const checkStateOperations = [
  { key: "0", value: "未校验" },
  { key: "1", value: "已校验" },
];

const checkResultOperations = [
  { key: "0", value: "不通过" },
  { key: "1", value: "合格" },
];

const bindTypeOperations = [
  { key: "0", value: "手工" },
  { key: "1", value: "扫码" },
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
        workCellCode: "",
        produCode: "",
        checkResult: "",
        checkDateS: "",
        checkDateE: "",
        sort: "-check_date",
      },
      checkResultOperations,
      checkStateOperations,
      bindTypeOperations,
      autoRefresh: true,
      intervalId: "",
    };
  },
  created() {
    this.getList();
    this.intervalId = setInterval(() => {
      this.getList();
    }, 1000);
  },
  methods: {
    getList() {
      this.listLoading = true;
      axios
        .post("http://localhost/pc/select", this.listQuery)
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
        checkResult: "",
        checkDateS: "",
        checkDateE: "",
        sort: "-check_date",
      };
      this.getList();
    },
    handleAutoRefresh() {
      if (this.autoRefresh) {
        this.intervalId = setInterval(() => {
          this.getList();
        }, 1000);
      } else {
        clearInterval(this.intervalId);
      }
    },
  },
};
</script>
