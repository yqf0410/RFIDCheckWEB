<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.produLotCode"
        class="filter-item"
        placeholder="零件批次号"
        style="width: 150px"
        @keyup.enter.native="handleFilter"
      />
      <el-input
        v-model="listQuery.equipCode"
        class="filter-item"
        placeholder="机床编号"
        style="width: 150px"
        @keyup.enter.native="handleFilter"
      />
      <el-input
        v-model="listQuery.rfidUid"
        class="filter-item"
        placeholder="RFID Uid"
        style="width: 150px"
        @keyup.enter.native="handleFilter"
      />
      <el-date-picker
        v-model="listQuery.createDateS"
        placeholder="创建开始时间"
        style="width: 200px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
      <a> - </a>
      <el-date-picker
        v-model="listQuery.createDateE"
        placeholder="创建结束时间"
        style="width: 200px; margin-right: 40px"
        type="datetime"
        format="yyyy-MM-dd HH:mm:ss"
        @keyup.enter.native="handleFilter"
      />
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
        <template slot-scope="scope" width="200" align="center">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="机床编号" width="150" align="center">
        <template slot-scope="scope">
          {{ scope.row.equipCode }}
        </template>
      </el-table-column>
      <el-table-column label="零件批次号" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.produLotCode }}
        </template>
      </el-table-column>
      <el-table-column label="总加工数量" width="150" align="center">
        <template slot-scope="scope">
          {{ scope.row.qty }}
        </template>
      </el-table-column>
      <el-table-column label="RFID UID" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.rfidUid }}
        </template>
      </el-table-column>
      <el-table-column label="RFID Data" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.rfidData }}
        </template>
      </el-table-column>
      <el-table-column label="过点状态" width="150" align="center">
        <template slot-scope="scope">
          {{ checkStateOperations[scope.row.checkState].value }}
        </template>
      </el-table-column>
      <el-table-column label="过点时间" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.checkDate }}
        </template>
      </el-table-column>
      <el-table-column label="导入/创建时间" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.createDate }}
        </template>
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button
            v-if="row.status != 'deleted'"
            size="mini"
            type="danger"
            @click="handleDelete(row, $index)"
          >
            删除
          </el-button>
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
import UploadExcelComponent from "@/components/UploadExcel/index.vue";
import axios from "axios";

const checkStateOperations = [
  { key: "0", value: "未过点" },
  { key: "1", value: "成功" },
  { key: "2", value: "失败" },
];

const checkStateKeyValue = checkStateOperations.reduce((acc, cur) => {
  acc[cur.key] = cur.value;
  return acc;
}, {});

export default {
  components: { Pagination, UploadExcelComponent },
  filters: {
    statusFilter(status) {
      const statusMap = {
        draft: "info",
        deleted: 'danger',
      };
      return statusMap[status];
    },
    typeFilter(type) {
      return checkStateKeyValue[type];
    },
  },
  data() {
    return {
      list: null,
      listLoading: true,
      total: 0,
      listQuery: {
        page: 1,
        limit: 10,
        equipCode: "",
        produLotCode: "",
        rfidUid: "",
        createDateS: "",
        createDateE: "",
        sort: "-create_date",
      },
      checkStateOperations,
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      axios
        .post("http://localhost/t/selectTaskBind", this.listQuery)
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
    handleDelete(row) {
      this.temp = Object.assign({}, row); // copy obj
      const tempData = Object.assign({}, this.temp);
      tempData.timestamp = +new Date(tempData.timestamp);
      axios
        .post("http://localhost/t/daleteTask", tempData)
        .then((resData) => {
          this.$notify({
            title: "成功",
            message: "删除成功",
            type: "success",
            duration: 2000,
          });
          this.dialogFormVisible = false;
          this.getList();
        })
        .catch((err) => {
          this.$notify({
            title: "失败",
            message: err.data,
            type: "error",
            duration: 2000,
          });
        });
    },
  },
};
</script>
