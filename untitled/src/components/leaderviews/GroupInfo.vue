<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 基础表格
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-button
            class="handle-del mr10"
            icon="el-icon-delete"
            type="primary"
            @click="delAllSelection"
        >批量删除
        </el-button>
        <el-select v-model="query.class" class="handle-select mr10" placeholder="班级">
          <el-option key="1" label="软件81班" value="软件81班"></el-option>
          <el-option key="2" label="软件82班" value="软件82班"></el-option>
        </el-select>
        <el-input v-model="query.name" class="handle-input mr10" placeholder="用户名"></el-input>
        <el-button icon="el-icon-search" type="primary" @click="handleSearch">搜索</el-button>
      </div>
      <el-table
          ref="multipleTable"
          :data="tableData"
          border
          class="table"
          header-cell-class-name="table-header"
          @selection-change="handleSelectionChange"
      >
        <el-table-column align="center" type="selection" width="55"></el-table-column>
        <el-table-column align="center" label="ID" prop="id" width="55"></el-table-column>
        <el-table-column label="用户名" prop="name"></el-table-column>
        <el-table-column align="center" label="头像(查看大图)">
          <template slot-scope="scope">
            <el-image
                :preview-src-list="[scope.row.thumb]"
                :src="scope.row.thumb"
                class="table-td-thumb"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column label="班级" prop="class"></el-table-column>
        <el-table-column align="center" label="状态">
          <template slot-scope="scope">
            <el-tag
                :type="scope.row.state==='成功'?'success':(scope.row.state==='失败'?'danger':'')"
            >{{ scope.row.state }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column align="center" label="操作" width="180">
          <template slot-scope="scope">
            <el-button
                icon="el-icon-edit"
                type="text"
                @click="handleEdit(scope.$index, scope.row)"
            >编辑
            </el-button>
            <el-button
                class="red"
                icon="el-icon-delete"
                type="text"
                @click="handleDelete(scope.$index, scope.row)"
            >删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
            :current-page="query.pageIndex"
            :page-size="query.pageSize"
            :total="pageTotal"
            background
            layout="total, prev, pager, next"
            @current-change="handlePageChange"
        ></el-pagination>
      </div>
    </div>

    <!-- 编辑弹出框 -->
    <el-dialog :visible.sync="editVisible" title="编辑" width="30%">
      <el-form ref="form" :model="form" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="身份">
          <el-input v-model="form.identity"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
    </el-dialog>
    <div>
      <el-button @click="Cancel">取消</el-button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BaseTable',
  data() {
    return {
      query: {
        class: '',
        name: '',
        pageIndex: 1,
        pageSize: 10
      },
      tableData: [],
      multipleSelection: [],
      delList: [],
      editVisible: false,
      pageTotal: 0,
      form: {},
      idx: -1,
      id: -1
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    getData() {
      this.$axios({
        method: 'get',
        url: '/table.json'
      }).then((response) => {
        console.log(response.data);
        this.tableData = response.data.list;
      }, response => {
        console.log('error');
      });
    },
    // 触发搜索按钮
    handleSearch() {
      this.$set(this.query, 'pageIndex', 1);
      this.getData();
    },
    // 删除操作
    handleDelete(index, row) {
      // 二次确认删除
      this.$confirm('确定要删除吗？', '提示', {
        type: 'warning'
      })
          .then(() => {
            this.$message.success('删除成功');
            this.tableData.splice(index, 1);
          })
          .catch(() => {
          });
    },
    // 多选操作
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    delAllSelection() {
      const length = this.multipleSelection.length;
      let str = '';
      this.delList = this.delList.concat(this.multipleSelection);
      for (let i = 0; i < length; i++) {
        str += this.multipleSelection[i].name + ' ';
      }
      this.$message.error(`删除了${str}`);
      this.multipleSelection = [];
    },
    // 编辑操作
    handleEdit(index, row) {
      this.idx = index;
      this.form = row;
      this.editVisible = true;
    },
    // 保存编辑
    saveEdit() {
      this.editVisible = false;
      this.$message.success(`修改第 ${this.idx + 1} 行成功`);
      this.$set(this.tableData, this.idx, this.form);
    },
    // 分页导航
    handlePageChange(val) {
      this.$set(this.query, 'pageIndex', val);
      this.getData();
    },
    Cancel() {
      this.$cookie.delete('groupsetenable');
      this.$router.push('/groupsetup');
      this.$router.go(0);
    }
  }
};
</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}

.table {
  width: 100%;
  font-size: 14px;
}

.red {
  color: #ff0000;
}

.mr10 {
  margin-right: 10px;
}

.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>