<template>
  <div class="app-container">
    <div style="margin: 0 0 20px 0;">
      <el-button type="primary" plain @click="OpenDialog('新增用户')">新增</el-button>
      <el-button type="primary" plain @click="OpenDialog('删除用户')">删除</el-button>
      <el-button type="primary" plain @click="OpenDialog('修改用户')">修改</el-button>
      <el-button type="primary" plain @click="OpenDialog('查询用户')">查询</el-button>
    </div>

    <el-table
      v-loading="IsTableLoading"
      :data="tableData"
      border fit highlight-current-row
      @selection-change="handleSelectionChange"
      style="width:100%;">

      <el-table-column
        type="selection"
        align="center"
        width="40">
      </el-table-column>

      <el-table-column
          v-for="(column, index) in this.columns"
          :key="index"
          :prop="column.prop"
          :label="column.label"
          :width="column.width"
          :align="column.align"
          >
          <template slot-scope="scope" >
            <div v-if="column.label==='创建时间'">{{ tableData[scope.$index][column.prop].replace(/T/g, ' ') }}</div>
            <div v-else-if="column.label==='密码'">哈希值</div>
            <div v-else-if="column.label==='角色'"><el-tag>{{ tableData[scope.$index][column.prop]}}</el-tag></div>
            <div v-else-if="column.label==='地址'" v-for="(column2, index2) in tableData[scope.$index][column.prop]" :key="index2">
              {{index2+1}}:
              <span v-for="(column3, index3) in column2.info" :key="index3">
                {{ column3}}
              </span>
            </div>
            <div v-else>
              {{ tableData[scope.$index][column.prop]}}
            </div>
          </template>
      </el-table-column>

    </el-table>

    <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[5, 10, 15, 20]"
          :page-size=PageSize
          layout="total, sizes, prev, pager, next, jumper"
          :total="TotalPage"
          style="margin-top: 20px;">
    </el-pagination>

    <!-- 弹出框1 -->
    <el-dialog :title="this.dialog_title2" :visible.sync="dialogVisible2" width="60%" >
      
      <el-form :rules="rules" label-position="right" ref="form" :model="dialogdata2" label-width="80px">
        <!-- 使用el-row和el-col创建栅格布局 -->
        <el-row :gutter="20">
          <el-col :span="12" v-for="(column, index) in columns" :key="index">
            <span v-if="column.label==='ID'||column.label==='创建时间'||column.label==='版本'||column.label==='地址'" />
            
            <el-form-item v-else :label="column.label" :prop="column.prop">
              <el-input v-model="dialogdata2[column.prop]"></el-input>
            </el-form-item>
          
          </el-col>
        </el-row>

        <el-form-item style="max-height: 200px; overflow-y: auto;" label="设定地址">
          <div style="margin:10px 20px 20px 10px;display: block;" v-for="(data2,index2) in addresses" :key="index2">
            <div class="radio-group-content" :label="index2">
              {{ index2+1 }}. | 所在地区：
              <span v-for="(column3, index3) in data2.info" :key="index3">
                {{ column3}}
              </span>
              | 详细地址：{{ data2.detail }} | 收件人：{{ data2.name }} | 电话：{{ data2.phone }} <div v-if="data2.is_default" style="margin: 5px 0 0 30px;display: inline-block;background-color: rgba(127, 255, 212, 0.648);border: 1px rgba(0, 0, 0, 0.171) solid;border-radius: 5px;padding: 0 5px;color: rgb(14, 198, 249);">默认</div><el-button 
              @click="$refs.address.dialog_title='修改地址',change_address(index2,data2)" style="margin-left: 20px;" size="mini" type="primary" icon="el-icon-edit" circle></el-button><el-button size="mini" 
              @click="delete_address(index2)" type="danger" icon="el-icon-delete" circle></el-button>
            </div>
          </div>

          <!-- 触发弹窗2的开关 -->
          <el-button @click="new_address" style="margin: 10px 0 0 20px;display: block;" size="mini" type="primary" icon="el-icon-edit">
            新增地址
          </el-button>

        </el-form-item>
        
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible2 = false,gettable()">取消</el-button>
        <el-button type="primary" @click="confirm_change">确定</el-button>
      </span>
    </el-dialog>

    <!-- 弹出框1-2(地址选择) -->
    <El2 ref="address" style="text-align:left;margin-left:20px;margin-bottom:20px;" @address="address_finish" />


  </div>
</template>

<script>
import axios from '@/utils/axios';
import El2 from '/src/components/AddressAdd/ElAddress2'


  export default {
    components:{
      El2
    },
    data() {
      return {
        tableData: [],
        currentPage:1,
        PageSize:10,
        TotalPage:null,
        IsTableLoading:true,
        columns : [
        { prop: 'id', label: 'ID', width: '180' },
        { prop: 'name', label: '姓名', width: '80' },
        { prop: 'age', label: '年龄', width: '80' },
        { prop: 'sex', label: '性别', width: '80' },
        { prop: 'phone', label: '电话', width: '80' },
        { prop: 'addresses', label: '地址' },
        { prop: 'role', label: '角色', width: '100',align:'center' },
        { prop: 'password', label: '密码' , width: '120' },
        { prop: 'money', label: '钱包', width: '100' },
        { prop: 'create_time', label: '创建时间', width: '180' },
        { prop: 'version', label: '版本', width: '100' },
        ],
        dialogVisible2:false,
        dialog_title2:"",
        dialogdata2:
        {
          "id": null,
          "name":null,
          "age": null,
          "sex": null,
          "phone": null,
          "addresses":null,
          "role":null,
          "password":null,
          "money":null,
          "create_time":null,
          "version":null,
        },
        radio:null,
        addresses:[],
        rules: {
          name: [
            { required: true, message: '请输入姓名', trigger: 'blur' },
            //{ min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            //{ min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
        },
        rows_selection:[]
      }
    },
    methods:{
      // 地址组件选完 触发的事件
      address_finish(index,val){
        console.log("父组件")
        console.log(index)
        console.log(val)

        if(this.$refs.address.dialog_title==='新增地址'){
          const address = JSON.parse(JSON.stringify(val))

          // 默认
          if(address?.is_default){
            this.addresses.forEach(item=>{
              item.is_default = false
            })
          }
          this.addresses.push(address)
        }

        else if(this.$refs.address.dialog_title==='更改地址'){
          const address = JSON.parse(JSON.stringify(val))
          console.log(address)
          // 默认
          if(address?.is_default){
            this.addresses.forEach(item=>{
              item.is_default = false
            })
          }

          //this.addresses[index] = JSON.parse(JSON.stringify(val)) 直接改索引 vue不会渲染
          this.$set(this.addresses, index, address);
        }
        
        
        else{
          this.$message.error("走到这一步，说明你写错了")
        }
        console.log(this.addresses)
        

      },
      // 删除地址（数组里某个）
      delete_address(index2){
        console.log(index2)
        this.addresses.splice(index2,1)
      },
      // 新加地址
      new_address(){
        this.$refs.address.dialogdata =
        {
          "info": [],
          "info_code":[],
          "name": "",
          "phone": "",
          "detail": "",
          "is_default":false
        },
        this.$refs.address.dialog_title='新增地址'
        this.$refs.address.dialogVisible=true;
      },
      // 更改地址（数组里某个）// 这里深拷贝
      change_address(index2,data2){
        this.$refs.address.dialogdata = JSON.parse(JSON.stringify(data2))
        this.$refs.address.dialog_title = '更改地址'
        this.$refs.address.dialogindex = index2
        this.$refs.address.dialogVisible=true
      },
      gettable(){
        this.IsTableLoading = true
        axios.get('/user/page',{
          params: {
            currentPage: this.currentPage,
            PageSize: this.PageSize
          }
        }).then(response=>{
          if(response.data.code===0)this.$message.error(response.data.msg)
          else {
            this.tableData = response.data.data.records
            this.TotalPage = response.data.data.total
            this.IsTableLoading = false
            //this.$message.success("获取成功")
          }
        }).catch(error=>{
          this.$message.error(error.data.msg)
          console.log(error)
        })
      },
      // 页容量变化
      handleSizeChange(val) {
        this.PageSize = val
        this.gettable()
      },
      // 切页
      handleCurrentChange(val) {
        this.currentPage = val
        this.gettable()
      },
      // 表格列选择
      handleSelectionChange(val){
        console.log(val)
        this.rows_selection = val
      },
      // 点增删改查
      OpenDialog(val){

        if(val==="新增用户"){
          // 初始化数据
          this.dialogdata2 = 
          {
            "id": null,
            "name":null,
            "age": null,
            "sex": null,
            "phone": null,
            "addresses":null,
            "role":null,
            "password":null,
            "money":null,
            "create_time":null,
            "version":null,
          },
          this.addresses = []
          // 初始化验证 // ?.防止this.$refs.form为空报错 为空不运行
          this.$refs.form?.resetFields();

          this.dialog_title2 = val
          this.dialogVisible2 = true
        }

        else if(val==="删除用户"){
          if(this.rows_selection.length===0){
            this.$message.error("请选择要删除的用户")
            return
          }

          this.$confirm('确定删除这'+(this.rows_selection.length)+'个用户吗?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.delete_byadmin() // 删除用户
            }).catch(() => {
              this.$message({
                type: 'info',
                message: '已取消删除'
              });          
            });
        }

        else if(val==="修改用户"){
          if(this.rows_selection.length===0){
            this.$message.error("请选择要修改的用户")
            return
          }

          if(this.rows_selection.length!==1){
            this.$message.error("一次只能修改一个用户")
            return
          }


          // 初始化数据
          this.dialogdata2 = this.rows_selection?.[0]
          this.addresses = this.rows_selection?.[0]?.addresses
          // 初始化验证 // ?.防止this.$refs.form为空报错 为空不运行
          this.$refs.form?.resetFields();

          this.dialog_title2 = val
          this.dialogVisible2 = true
        }

        else{
          //
        }

        
      },
      // 展开框1的确定
      confirm_change(){
        this.dialogdata2.addresses = this.addresses
        console.log(this.dialogdata2)
        this.$refs.form.validate(result => {
                if (result) {
                  console.log("验证通过");
                  if(this.dialog_title2==="新增用户")
                    this.axios_add()
                  else if(this.dialog_title2==="修改用户")
                    this.axios_update()
                  else
                    console.log()
                } else{
                  console.log("验证失败");
                }
            })

      },
      // 添加一个
      axios_add(){
        axios.post('/user/addonebyadmin',this.dialogdata2)
        .then(response=>{
          if(response.data.code===0)this.$message.error(response.data.msg)
          else {
            this.dialogVisible2 = false
            this.$message.success("添加成功")
            this.gettable()
          }
        }).catch(error=>{
          this.$message.error(error.data.msg)
          console.log(error)
        })
      },
      // 更新一个
      // 新范例 防404报错
      axios_update(){
        console.log(this.dialogdata2)
        axios.put('/user/updateonebyadmin',this.dialogdata2)
        .then(response=>{
          if(response.data.code===0)this.$message.error(response.data.msg)
          else {
            this.dialogVisible2 = false
            this.$message.success("更新成功")
            this.gettable()
          }
        }).catch(error=>{
          if(error.data?.msg)this.$message.error(error.data.msg)
          else this.$message.error("error")
          console.log(error)
        })
      },
      // 删一堆
      delete_byadmin(){
        axios.post('/user/deletelistbyadmin',this.rows_selection)
        .then(response=>{
          if(response.data.code===0)this.$message.error(response.data.msg)
          else {
            this.dialogVisible2 = false
            this.$message.success("删除成功")
            this.gettable()
          }
        }).catch(error=>{
          this.$message.error(error.data.msg)
          console.log(error)
        })
      }
    },
    mounted(){
      this.gettable()
    }
  }
</script>

<style scoped>
.radio-group-content {
  white-space: normal; /* 允许内容换行 */
  word-break: break-word; /* 在长单词或URL地址内部进行换行 */
}
</style>
