<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 添加按钮 -->
            <el-row>
                <el-col>
                    <el-button @click="shoAddCateDialog" type="primary">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 表格区域 -->
            <tree-table class="treeTable"
             :data="cartList" :columns="columns"
             :selection-type = "false" :expand-type = "false" 
             :show-index = "true" index-text ="#" border
             :show-row-hover="false">
             <!-- 是否有效 -->
             <template slot="isok" slot-scope ="scope">
                <i style="color:lightgreen" v-if="scope.row.cat_delete === false" class="el-icon-error"></i>
                <i style="color:lightgreen" v-else class="el-icon-success"></i>
            </template>
            <!-- 排序 -->
            <template slot="order" slot-scope ="scope">
                <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                <el-tag type="success" v-else-if="scope.row.cat_level === 1" size="mini">二级</el-tag>
                <el-tag type="warning" v-else size="mini">三级</el-tag>
            </template>
            <!-- 操作 -->
            <template slot="opt" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            </template>
            </tree-table>
            <!-- 分页区域 -->
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[3, 5, 10, 15]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
          </el-pagination>
        </el-card>
        <!-- 添加分类对话框 -->
        <el-dialog
            title="添加分类"
            :visible.sync="addCateDialogVisible"
            width="50%"
            @close="addCateDialogClosed"
            >
            <!-- 添加分类表单 -->
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRulesRef" label-width="100px">
                <el-form-item label="分类名称:" prop="cat_name">
                  <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类:" >
                    <!-- options指定数据源 props是配置选项 -->
                    <el-cascader
                    expandTrigger ="hover"
                    v-model="selectedKeys"
                    :options="parentCateList"
                    :props="cascaderProps"
                    @change="parentCateChange"
                    change-on-select 
                    clearable></el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
    export default {
        data(){
            return{
                // 商品分类数据列表，默认为空
                cartList:[],
                // 查询条件
                queryInfo:{
                    type:3,
                    pagenum:1,
                    pagesize:5
                },
                // 总数据条数
                total:0,
                // 为table指定列
                columns:[{
                    label:'分类名称',
                    prop:'cat_name',
                },{
                    label:'是否有效',
                    // 表示将当前列定义为模板列
                    type:'template',
                    // 表示当前列使用的模板名称
                    template:'isok'
                },{
                    label:'排序',
                    type:'template',
                    template:'order'
                },{
                    label:'操作',
                    type:'template',
                    template:'opt'
                }],
                // 控制添加分类对话框的显示与隐藏
                addCateDialogVisible:false,
                // 添加分类的表单数据对象
                addCateForm:{
                    // 将要添加的分类的名称
                    cat_name:'',
                    // 父级分类的id
                    cat_pid:0,
                    // 默认分类等级是一级分类
                    cat_level:0
                },
                // 添加分类表单验证规则对象
                addCateFormRules:{
                    cat_name:[
                        {required:true,message:'请输入分类名称',trigger:'blur'}
                    ]
                },
                // 父级分类列表
                parentCateList:[],
                // 指定级联选择器的配置对象
                cascaderProps:{
                    value:'cat_id',
                    label:'cat_name',
                    children:'children'
                },
                // 选中的父级分类的数组
                selectedKeys:[]
                
                    
            }
        },
        created(){
            this.getCateList()
        },
        methods:{
            // 获取商品分类数据
            async getCateList(){
                //
                const{data:res} = await this.$http.get(`categories`,{params:this.queryInfo})
                if(res.meta.status !== 200) {return this.$message.error("获取数据失败") }
                // 把数据列表赋值
                this.cartList = res.data.result
                // 为总数居条数赋值
                this.total = res.data.total
            },
            // 监听pagesize改变
            handleSizeChange(newSize){
                this.queryInfo.pagesize = newSize
                this.getCateList()
            },
            // 监听pagenum改变
            handleCurrentChange(newPage){
                this.queryInfo.pagenum = newPage
                this.getCateList()
            },
            // 点击按钮展示添加对话框
            shoAddCateDialog(){
                // 先获取父级分类的数据列表然后展示对话框
                this.getParentCateList()
                this.addCateDialogVisible = true

            },
            // 获取父级分类的数据列表
            async getParentCateList(){
               const {data:res} = await this.$http.get(`categories`,{params:{type:2}})
               if(res.meta.status!==200){return this.$message.error("获取数据失败")}
               this.parentCateList = res.data

            },
            // 选择项发生变化触发函数
            parentCateChange(){
                // 如果selectedKeys数组中的length大于0证明选中了父级分类
                // 反之没选中任何父级分类
                if(this.selectedKeys.length >0){
                    // 父级分类的ID
                    this.addCateForm.cat_pid =  this.selectedKeys[this.selectedKeys.length-1]
                    this.addCateForm.level = this.selectedKeys.length
                    return
                }
                else{
                    this.addCateForm.cat_pid = 0
                    this.addCateForm.cat_level = 0
                }
            },
            // 点击按钮添加分类
             addCate(){
                this.$refs.addCateFormRulesRef.validate(async valid=>{
                    if(!valid) return
                    const {data:res} =  await this.$http.post(`categories`,this.addCateForm)
                    if(res.meta.status !== 201){return this.$message.error("添加分类失败")}
                    this.$message.success("添加分类成功")
                    this.getCateList()
                    this.addCateDialogVisible = false
                })
            },
            // 监听对话框关闭事件重置表单数据
            addCateDialogClosed(){
                this.$refs.addCateFormRulesRef.resetFields()
                this.selectedKeys = []
                this.addCateForm.cat_level = 0
                this.addCateForm.cat_pid = 0
            }
        }
    }
</script>
<style lang="less" scoped>
.treeTable{
    margin-top: 15px;
}
.el-cascader{
    width: 100%;
}
</style>