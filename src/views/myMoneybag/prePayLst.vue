<template>
  <div>
    <el-form ref="ruleForm" :rules="rules" :inline="true" :model="formAllInfo" label-width="100px">
      <el-row :gutter="20">

        <el-col :span="8">
          <el-form-item label="选择代理商" prop="selAgentId">
            <el-select v-model="formAllInfo.selAgentId" filterable placeholder="请选择">
              <el-option :label="item.agentname" :value="item.id" v-for="item in agentsList" :key="item.id"></el-option>
            </el-select>
            
          </el-form-item>
        </el-col>

        <el-col :span="8">
          <el-form-item label="开始日期">
            <el-date-picker
              type="date"
              value-format="yyyy-MM-dd"
              placeholder="选择日期"
              v-model="formAllInfo.fromSearch"
            ></el-date-picker>
          </el-form-item>
        </el-col>

        <el-col :span="8">
          <el-form-item label="结束日期">
            <el-date-picker
              type="date"
              value-format="yyyy-MM-dd"
              placeholder="选择日期"
              v-model="formAllInfo.toSearch"
            ></el-date-picker>
          </el-form-item>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="4" :offset="10">
          <el-button type="primary" @click="searchData('prePayLst')">查询</el-button>
        </el-col>
      </el-row>

      <div class="clearfix">
        <el-button
          type="primary"
          size="mini"
          class="fr"
          @click="searchData('downLoadPrePaysLst')"
        >导出</el-button>
      </div>


    </el-form>

<el-divider>查询结果</el-divider>

<el-table border :stripe="true" :data="tableList.slice((currentPage-1)*pageSize,currentPage*pageSize)" style="width: 100%">
        <el-table-column label="代理商" prop="agentname" align="center"></el-table-column>
        <el-table-column label="充值金额" prop="balance" align="center"></el-table-column>
        <el-table-column label="充值时间" prop="createTime"  align="center"></el-table-column>
        <el-table-column label="充值后余额" prop="currBalance" align="center"></el-table-column>
        <el-table-column align="right" width="200">
          <template slot="header" slot-scope="scope">
            <el-input v-model="keySearch" size="mini" placeholder="输入代理商搜索"/>
          </template>
        </el-table-column>
</el-table>

    <pagination
      class="fr"
      :total="tableList.length"
      :currentPage="currentPage"
      :pageSize="pageSize"
      :pageSizes="pageSizes"
      @handleSizeChangeEmit="handleSizeChange"
      @handleCurrentChangeEmit="handleCurrentChange"
    />

  </div>
</template>

<script>
import pagination from "../../components/myMoneybag/pagination";
export default {
  name: "prePayLst",
  data() {
    return {
        formAllInfo:{},
        rules: {
            selAgentId: [
                { required: true, message: '必填', trigger: 'blur' },
            ],
        },
        agentsList:[],
        tableData: [],
        currentPage: 1, //当前第一页
        pageSize: 30, //默认每页1条数据
        pageSizes: [30, 50, 100], //设置每页显示多少条
        keySearch: "",
    };
  },
  components: { pagination},
  created() {
      this.getMyAgentFn();
  },
  computed: {
      //复制一份表格数据
    tableList() {
        return this.tableData.filter(item => {
            if (item.agentname.includes(this.keySearch)) {
                return item;
            }
            this.currentPage = 1;
        });
    },
  },
  methods:{
    handleSizeChange(data) {
        this.pageSize = data;
        this.currentPage = 1;
    },
    handleCurrentChange(data) {
        this.currentPage = data;
    },
    async getMyAgentFn(){
        try {
            const data = await this.$axios.post(
            "/fr/AllUSIM/myAgent",
            this._qs.stringify({
              agentId: localStorage.getItem("agentid"),
              userId:localStorage.getItem("userid"),
              username:JSON.parse(localStorage.getItem("userLoginInfo")).username
            }),
            );
            if(data.errcode==0){
                this.agentsList=data.data.data;
            }else{
                this.$message.error(data.errmsg);
            }
            
        } catch (err) {
            console.log(err);
            this.$message.error("服务器异常，请稍后再试！");
        }
    },
    searchData(url){

        this.$refs['ruleForm'].validate( async (valid) => {
            if (valid) {
                let paramsObj={
                    agentId:localStorage.getItem("agentid"),
                    userType:localStorage.getItem("usertype"),
                    userId :localStorage.getItem("userid"),
                    userName :JSON.parse(localStorage.getItem("userLoginInfo")).username,
                    ...this.formAllInfo
                }

                try {
                    const data = await this.$axios.post(
                    "/fr/Reconciliatio/"+url,
                    this._qs.stringify(paramsObj),
                    );
                    if(data.errcode==0){
                        if(url=="prePayLst"){
                            this.tableData=data.myagent.data;
                        }
                        if(url=="downLoadPrePaysLst"){
                            window.location.href = data.downUrl;
                        }
                    }else{
                        this.$message.error(data.errmsg);
                    }
                    
                } catch (err) {
                    console.log(err);
                    this.$message.error("服务器异常，请稍后再试！");
                }
            }else {
                return false;
            }
        });



        
    },
  }
};
</script>

<style scoped>
</style>