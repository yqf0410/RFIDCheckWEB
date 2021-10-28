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
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >
        新增
      </el-button>
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-import"
        @click="handleImport"
      >
        EXCEL导入
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
      <el-table-column label="机床编号" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.equipCode }}
        </template>
      </el-table-column>
      <el-table-column label="零件批次号" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.produLotCode }}
        </template>
      </el-table-column>
      <el-table-column label="总加工数量" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.qty }}
        </template>
      </el-table-column>
      <el-table-column label="已加工数量" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.finishQty }}
        </template>
      </el-table-column>
      <el-table-column label="待加工数量" width="200" align="center">
        <template slot-scope="scope">
          {{ scope.row.bindQty }}
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
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            修改
          </el-button>
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

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="100px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="机床编号" prop="equipCode">
          <el-input v-model="temp.equipCode" />
        </el-form-item>
        <el-form-item label="零件批次号" prop="produLotCode">
          <el-input v-model="temp.produLotCode" />
        </el-form-item>
        <el-form-item label="数量" prop="qty">
          <el-input v-model="temp.qty" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >
          确认
        </el-button>
      </div>
    </el-dialog>

    <el-dialog title="上传" :visible.sync="dialogImportVisible">
      <upload-excel-component
        :on-success="handleSuccess"
        :before-upload="beforeUpload"
      />
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 100%; margin-top: 20px"
      >
        <el-table-column
          v-for="item of tableHeader"
          :key="item"
          :prop="item"
          :label="item"
        />
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogImportVisible = false"> 取消 </el-button>
        <el-button type="primary" @click="importExcel()"> 确认 </el-button>
      </div>
    </el-dialog>
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

const bindStateOperations = [
  { key: "0", value: "未绑定" },
  { key: "1", value: "已绑定" },
];

const bindStateKeyValue = bindStateOperations.reduce((acc, cur) => {
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
      return bindStateKeyValue[type];
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
        createDateS: "",
        createDateE: "",
        sort: "-create_date",
      },
      bindStateOperations,
      //修改
      temp: {
        id: "",
        equipCode: "",
        produLotCode: "",
        qty: "",
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "修改",
        create: "新增",
      },
      rules: {
        equipCode: [{ required: true, message: "请输入机床编号", trigger: "blur" }],
        produLotCode: [
          { required: true, message: "请输入零件批次号", trigger: "blur" },
        ],
        qty: [{ required: true, message: "请输入数量", trigger: "blur" }],
      },
      dialogImportVisible: false,
      //导入
      tableData: [],
      tableHeader: [],
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      axios
        .post("http://localhost/t/selectTask", this.listQuery)
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
    resetTemp() {
      this.temp = {
        id: "",
        seq: "",
        noCode: "",
        produCode: "",
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    handleImport() {
      this.dialogImportVisible = true;
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          axios
            .post("http://localhost/t/saveTask", tempData)
            .then((resData) => {
              if (resData.data == "success") {
                this.$notify({
                  title: "成功",
                  message: "保存成功",
                  type: "success",
                  duration: 2000,
                });
                this.dialogFormVisible = false;
                this.getList();
              } else {
                this.$notify({
                  title: "失败",
                  message: resData.data,
                  type: "error",
                  duration: 2000,
                });
              }
            })
            .catch((err) => {
              this.$notify({
                title: "失败",
                message: err.data,
                type: "error",
                duration: 2000,
              });
            });
        }
      });
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          tempData.timestamp = +new Date(tempData.timestamp);
          axios
            .post("http://localhost/t/saveTask", tempData)
            .then((resData) => {
              this.$notify({
                title: "成功",
                message: "更新成功",
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
        }
      });
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
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 1;
      if (isLt1M) {
        return true;
      }
      this.$message({
        message: "Please do not upload files larger than 1m in size.",
        type: "warning",
      });
      return false;
    },
    handleSuccess({ results, header }) {
      this.tableData = results;
      this.tableHeader = header;
    },
    importExcel() {
      axios
        .post("http://localhost/t/importTask", this.tableData)
        .then((resData) => {
          this.$notify({
            title: "成功",
            message: "导入成功",
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
