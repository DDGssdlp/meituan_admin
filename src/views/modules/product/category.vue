<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button  
          type="primary"
          @click="addOrUpdateHandle()"
        >新增</el-button>
        <el-button
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-row>
      <el-col :span="6">
        <el-tree 
        :data="dataListTree" 
        accordion 
        :expand-on-click-node="true"
        :auto-expand-parent="false"
        node-key="catId"
        @node-click="handleNodeClick"
        :props="defaultProps"></el-tree>
      </el-col>
      <el-col :span="18">
        <el-table
          :data="dataList"
          border
          v-loading="dataListLoading"
          @selection-change="selectionChangeHandle"
          style="width: 100%;"
        >
          <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>

          <el-table-column prop="name" header-align="center" align="center" label="分类名称"></el-table-column>

          <el-table-column prop="catLevel" header-align="center" align="center" label="层级"></el-table-column>
          <el-table-column
            prop="showStatus"
            header-align="center"
            align="center"
            label="是否显示"
          >
           <template slot-scope="scope">
              <el-switch 
              v-model="scope.row.showStatus" 
               active-color="#13ce66"
               inactive-color="#ff4949"
               @change="handleChange(scope.row)"
              :active-value="1"
              :inactive-value="0"></el-switch>
          </template>
           
          </el-table-column>

          <el-table-column prop="icon" header-align="center" align="center" label="图标">
            
          </el-table-column>

          <el-table-column
            fixed="right"
            header-align="center"
            align="center"
            width="150"
            label="操作"
          >
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.catId)">修改</el-button>
              <el-button type="text" size="small" @click="deleteHandle(scope.row.catId, scope.row.name)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>

    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "./category-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      cartId: "",
      dataList: [],
      dataListTree: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList();
    this.getDataListWithTree();
  },
  created() {
    
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/getChildren"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          cartId: this.cartId
        })
      }).then(({ data }) => {
        if (data && data.code === '00000') {
          this.dataList = data.data.list;
          this.totalPage = data.data.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 过滤第三级节点：
    filterNode(value, data, node){
      return !data.children.length === 0;
    },
    //获取树状结构菜单：
    getDataListWithTree() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/getParent"),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === '00000') {
          this.dataListTree = data.data; 
          this.dataListLoading = false;
        }
      });
    },
    // 修改状态显示：
    handleChange(val){
      
      this.$http({
        url: this.$http.adornUrl("/product/category/changeStatus"),
        method: "post",
        data: this.$http.adornData({
                'catId': val.catId || undefined,
                'name': val.name,
                'parentCid': val.parentCid,
                'catLevel': val.catLevel,
                'showStatus': val.showStatus,
                'sort': val.sort,
                'icon': val.icon,
                'productUnit': val.productUnit,
                'productCount': val.productCount
              })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          console.log(data);
          this.$message({
            message: "显示状态修改成功！",
            type: "success"
          });
        }
        this.getDataListWithTree;
      });
      
    }, 
    // 点击节点获取节点下的所有子节点：
    handleNodeClick(data, node){
      console.log("node",node)
      this.cartId = data.catId;
      this.$http({
      url: this.$http.adornUrl('/product/category/getChildren'),
      method: 'get',
      params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          cartId: data.catId
        })
      }).then(({ data }) => {
         if (data && data.code === '00000') {
          this.dataList = data.data.list;
          this.totalPage = data.data.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
      })
    }, 
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id, name) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.catId;
          });
       var names = name
      console.log("dis", ids)
      this.$confirm(
        `确定对[${names}]进行 [${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              }
            });
          } else {
            this.$message.error(data.msg);
          }
        }).catch((err) => {});
      }).catch((err) => {});
    }
  }
};
</script>
