<template>
    <div class="idctab">
        <el-dialog :title="operateType === 'add' ? '添加机房' : '修改机房'" :visible.sync="visible"
                   close-on-press-escape center fullscreen>
            <el-form ref="assetsForm" :model="idc">
                <el-divider content-position="center">基础信息</el-divider>
                <el-row type="fiex" justify="space-between">
                    <el-col :span="6" :offset="3">
                        <el-form-item label="机房" prop="name">
                            <el-input v-model="idc.name"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="电话" prop="linkphone">
                            <el-input v-model="idc.linkphone"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="地址" prop="address">
                            <el-input v-model="idc.address"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row type="fiex">
                    <el-col :span="6" :offset="3">
                        <el-form-item label="运营商" prop="operator">
                            <el-select v-model="idc.operator" placeholder="请选择运营商">
                                <el-option label="电信" value="0"></el-option>
                                <el-option label="联通" value="1"></el-option>
                                <el-option label="移动" value="2"></el-option>
                                <el-option label="铁通" value="3"></el-option>
                                <el-option label="小宽带" value="4"></el-option>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="带宽" prop="bandwidth">
                            <el-input v-model="idc.bandwidth"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col :span="6">
                        <el-form-item label="网段" prop="network_segment">
                            <el-input v-model="idc.network_segment"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <span slot="footer">
                <el-button type="primary" @click="idcSubmit('idcForm')">提交</el-button>
                <el-button @click="visible=false">取消</el-button>
            </span>
        </el-dialog>
        <div class="mytab-header">
            <el-form :inline="true" :model="config">
                <el-form-item>
                    <el-button type="primary" @click="addIdc">添加</el-button>
                    <el-button type="danger" @click="deleteIdc">删除</el-button>
                    <el-tooltip effect="dark" content="刷新列表" placement="top">
                        <el-button type="warning" plain @click="getIdcTableData"
                                   icon="el-icon-refresh"></el-button>
                    </el-tooltip>
                </el-form-item>
            </el-form>
        </div>
        <el-table border class="commonTable"
                  ref="idcTable"
                  :data="tableData"
                  tooltip-effect="dark"
                  style="width: 100%"
                  @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"></el-table-column>
            <el-table-column prop="name" label="机房名"></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="warning" plain size="mini" @click="editIdc(scope.row)">Edit</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination class="pager"
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :current-page="config.page"
                       :page-sizes="[10, 15,20]"
                       :page-size="config.page_size"
                       layout="total, sizes, prev, pager, next, jumper"
                       background
                       :total="config.total">
        </el-pagination>
    </div>
</template>

<script>

  export default {
    data() {
      return {
        // 添加操作,或者 edit 修改
        operateType: 'add',
        // 是否显示dialog
        visible: false,
        idc: {
          name: "",
          linkphone: "",
          address: "",
          bandwidth: "",
          network_segment: "",
          operator: "",
        },

        // 数据
        // 表单数据
        tableData: [],
        // 左侧选择数据
        Selection:
          [],
        // 页面配置
        config:
          {
            page: 1,
            page_size: 10,
            total: undefined
          },
      }
    },
    methods: {
      getIdcTableData(event, my_url) {
        if (!my_url) {
          my_url = `api/v1/idc?page=${this.config.page}&limit=${this.config.page_size}`;
        }
        this.$http.get(my_url)
          .then(resp => {
            // 需要将 tag id 转换 text/value tag name 转化 value/label
            console.log(resp.data)
            this.tableData = resp.data.data;
            this.config.total = resp.data.count;
          }).catch(err => {
          console.log(err)
        })
      },
      addIdc() {
        this.idc = {
          name: "",
          linkphone: "",
          address: "",
          bandwidth: "",
          network_segment: "",
          operator: "",
        };
        this.operateType = "add";
        this.visible = true
      },
      editIdc(row) {
        this.$http.get(`api/v1/idc/${row.id}/`)
          .then(resp => {
            // 需要将 tag id 转换 text/value tag name 转化 value/label
            this.idc = resp.data;
            this.operateType = 'edit';
            this.visible = true
          }).catch(err => {
          this.$alert('未找到该用户,可能由于后台已删除,请刷新！', '错误', {
            confirmButtonText: '确定',
          });
          console.log(err)
        })
      },
      deleteIdc() {
        let id_list = [];
        this.Selection.forEach(obj => {
          id_list.push(obj.id)
        });
        // 发送delete 请求
        let param = {ids: id_list};
        this.$http.delete("api/v1/idc/", {data: param})
          .then(resp => {
            // 需要将 tag id 转换 text/value tag name 转化 value/label
            this.$message({
              message: '删除成功！',
              type: 'success'
            });
            // 刷新表格
            this.getIdcTableData()
          }).catch(err => {
          this.$notify.error({
            title: '错误',
            message: '出错了',
            duration: 0
          });
          console.log(err)
        })
      },
      idcSubmit() {
        this.$confirm('确定要提交吗, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            if (this.operateType === 'add') {
              // 执行添加
              this.$http.post('api/v1/idc/', this.idc, {
                transformRequest: [function (data) {
                  let params = "";
                  for (let index in data) {
                    params += index + "=" + data[index] + "&"
                  }
                  return params
                }]
              }).then(resp => {
                // 需要将 tag id 转换 text/value tag name 转化 value/label
                this.$message({
                  message: '恭喜，添加成功！',
                  type: 'success'
                });
                this.visible = false;
                // 刷新表格
                this.getIdcTableData()
              }).catch(err => {
                this.$notify.error({
                  title: '错误',
                  message: '出错了:' + err.data.message,
                  duration: 0
                });
                console.log(err)
              })
            } else {
              // 执行修改
              this.$http.put(`api/v1/idc/${this.idc.id}/`, this.idc, {
                transformRequest: [function (data) {
                  let params = "";
                  for (let index in data) {
                    params += index + "=" + data[index] + "&"
                  }
                  return params
                }]
              }).then(resp => {
                // 需要将 tag id 转换 text/value tag name 转化 value/label
                this.$message({
                  message: '恭喜，修改成功！',
                  type: 'success'
                });
                this.visible = false;
                // 刷新表格
                this.getIdcTableData()
              }).catch(err => {
                this.$notify.error({
                  title: '错误',
                  message: '出错了:' + err.data.message,
                  duration: 0
                });
                console.log(err)
              })
            }
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消提交'
            });
          });
      },
      // 多选 点击后获得选中数据
      handleSelectionChange(val) {
        this.Selection = val;
      },
      handleSizeChange(val) {
        // console.log(`每页 ${val} 条`);
        this.config.page_size = val;
        this.getIdcTableData()
      },
      handleCurrentChange(val) {
        // console.log(`当前页: ${val}`);
        this.config.page = val;
        this.getIdcTableData()
      }
    },
    created() {
      this.getIdcTableData();
    }
  }
</script>

<style scoped lang="scss">
    .idctab {
        height: 85%;
        padding-bottom: 15px;
        /*overflow: hidden;*/

        .el-button {
            margin-right: 5px;
        }

        &-header {
            display: inline-flex;
            justify-content: space-between;
            align-items: flex-start;

            .el-select {
                width: 130px;
            }
        }

        .commonTable {
            height: calc(100% - 65px);
            position: relative;

        }

        .pager {
            border: 0;
            position: absolute;
            right: 100px;
            margin-top: 20px;
        }
    }
</style>