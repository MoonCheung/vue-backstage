<template>
  <el-container>
    <el-main>
      <el-row>
        <el-col :span="24">
          <el-card class="art-card">
            <div slot="header"
                 class="catg-header">
              <span>分类管理</span>
            </div>
            <div class="clearfix">
              <el-button style="float: right;"
                         type="primary"
                         @click="addCatg">
                创建
              </el-button>
            </div>
            <el-table v-loading="listLoading"
                      :data="catgData"
                      style="width: 100%">
              <el-table-column type="index"
                               width="50" />
              <el-table-column prop="categoryname"
                               label="分类名称" />
              <el-table-column prop="categorydesc"
                               label="分类描述" />
              <el-table-column prop="cdate"
                               label="创建日期" />
              <el-table-column label="操作"
                               width="160">
                <template v-slot="scope">
                  <el-button type="primary"
                             size="small"
                             @click.native.prevent="editCatg(scope.row)">
                    编辑
                  </el-button>
                  <el-button type="danger"
                             size="small"
                             @click.native.prevent="delCatg(scope.row)">
                    删除
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-card>
        </el-col>
      </el-row>
      <!-- 分页 -->
      <el-row>
        <el-col :span="24">
          <v-pagination v-show="total > 0"
                        :total="total"
                        :page.sync="currentPage"
                        :limit.sync="limit"
                        style="padding-top:10px;"
                        @pagination="getCatgList" />
        </el-col>
      </el-row>
      <!-- 对话框 -->
      <el-dialog :visible.sync="dialogFormVisible"
                 title="创建分类"
                 width="40%">
        <el-form ref="catgForm"
                 :model="catgForm"
                 :rules="catgformRules"
                 label-position="right"
                 label-width="80px">
          <el-form-item label="分类名称"
                        prop="categoryname">
            <el-input v-model="catgForm.categoryname"
                      placeholder="请输入分类名称" />
          </el-form-item>
          <el-form-item label="分类描述"
                        prop="categorydesc">
            <el-input v-model="catgForm.categorydesc"
                      type="textarea"
                      placeholder="请输入分类描述" />
          </el-form-item>
        </el-form>
        <span slot="footer"
              class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取&nbsp;消</el-button>
          <el-button type="primary"
                     @click="submitData('catgForm')">确&nbsp;定</el-button>
        </span>
      </el-dialog>
    </el-main>
  </el-container>
</template>

<script>
import { addCategory, getCategory, editCategory, delCategory } from '@/api/category';
import vPagination from '@/components/Pagination';
import './index.scss';

export default {
  name: 'CatgList',
  components: {
    vPagination
  },
  data () {
    return {
      // 分页数据
      currentPage: 1,
      limit: 10,
      total: 1,
      // 表单数据
      catgData: [],
      dialogFormVisible: false,
      // 对话框
      catgForm: {
        id: '',
        categoryname: '',
        categorydesc: ''
      },
      catgformRules: {
        categoryname: [
          { required: true, message: '名称不能为空', trigger: 'blur' },
          {
            min: 2,
            max: 5,
            message: '建议输入长度在 2 到 5个字符',
            trigger: 'blur'
          }
        ],
        categorydesc: [
          { required: true, message: '描述不能为空', trigger: 'blur' }
        ]
      },
      todo: '',
      listLoading: true
    };
  },
  created () {
    this.getCatgList();
  },
  mounted () { },
  methods: {
    // 创建新表单
    initForm () {
      this.catgForm.categoryname = '';
      this.catgForm.categorydesc = '';
    },
    // 添加分类
    addCatg () {
      const _self = this;
      _self.todo = 'add';
      _self.initForm();
      _self.dialogFormVisible = true;
    },
    // 编辑分类
    editCatg (param) {
      const _self = this;
      _self.todo = 'edit';
      _self.dialogFormVisible = true;
      _self.catgForm = param;
    },
    // 删除分类
    delCatg (param) {
      this.$confirm('确认要删除该分类?', '提示', {
        distinguishCancelAndClose: true,
        confirmButtonText: '确认',
        cancelButtonText: '返回'
      }).then(() => {
        delCategory(param).then(res => {
          if (res.data.code === 1) {
            this.$message({
              type: 'success',
              message: res.data.msg
            });
            // 再次获取分类列表
            this.getCatgList();
          }
        });
      }).catch(action => {
        this.$message({
          type: 'info',
          message:
            action === 'cancel' ? '放弃删除并离开页面' : '停留在当前页面'
        });
      });
    },
    // 向服务器提交数据
    submitData (catgForm) {
      this.$refs.catgForm.validate(valid => {
        if (valid) {
          if (this.todo === 'add') {
            addCategory(this.catgForm).then(res => {
              if (res.data.code === 1) {
                this.dialogFormVisible = false;
                this.getCatgList();
                this.$message({
                  message: res.data.msg,
                  type: 'success'
                });
              } else {
                this.$message.error(res.data.msg);
              }
            }).catch(err => {
              console.error(err);
            });
          } else {
            editCategory(this.catgForm).then(res => {
              if (res.data.code === 1) {
                this.dialogFormVisible = false;
                this.getCatgList();
                this.$message({
                  message: res.data.msg,
                  type: 'success'
                });
              } else {
                this.$message.error(res.data.msg);
              }
            }).catch(err => {
              console.error(err);
            });
          }
        }
      });
    },
    // 获取分类列表
    getCatgList () {
      this.listLoading = true;
      const param = {
        curPage: this.currentPage,
        limit: this.limit
      };
      getCategory(param).then(res => {
        this.catgData = res.data.catgData;
        if (res.data.catgData.length > 0) {
          this.total = res.data.total;
        }
        this.listLoading = false;
      });
    }
  }
};
</script>

<style>
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}
</style>
