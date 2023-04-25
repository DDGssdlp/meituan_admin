<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="分类名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="分类名称"></el-input>
      </el-form-item>
      <el-form-item label="父分类" prop="parentCid">
         <el-cascader
          filterable
          clearable 
          placeholder="试试搜索：手机"
          v-model="dataForm.parentCid"
          :options="categorys"
          :props="setting"
    ></el-cascader>
      </el-form-item>

      <el-form-item label="是否显示" prop="showStatus">
        <el-switch
          v-model="dataForm.showStatus"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
      <el-form-item label="图标地址" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="图标地址"></el-input>
      </el-form-item>
      <el-form-item label="计量单位" prop="productUnit">
        <el-input v-model="dataForm.productUnit" placeholder="计量单位"></el-input>
      </el-form-item>
      <el-form-item label="商品数量" prop="productCount">
        <el-input v-model="dataForm.productCount" placeholder="商品数量"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import CategoryCascader from "../common/category-cascader";
export default {
  components: { CategoryCascader },
  data() {
    return {
      setting: {
        value: "catId",
        label: "name",
        children: "children"
      },
      categorys: [],
      visible: false,
      dataForm: {
        catId: 0,
        name: "",
        parentCid: "",
        catLevel: "",
        showStatus: "",
        sort: "",
        icon: "",
        productUnit: "",
        productCount: ""
      },
      menuList: [],
      menuListTreeProps: {
        children: "children",
        label: "name"
      },
      dataRule: {
        name: [
          { required: true, message: "分类名称不能为空", trigger: "blur" }
        ],
        parentCid: [
          { required: true, message: "父分类id不能为空", trigger: "blur" }
        ],
        catLevel: [
          { required: true, message: "层级不能为空", trigger: "blur" }
        ],
        showStatus: [
          {
            required: true,
            message: "是否显示[0-不显示，1显示]不能为空",
            trigger: "blur"
          }
        ],
        sort: [{ required: true, message: "排序不能为空", trigger: "blur" }],
        icon: [
          { required: true, message: "图标地址不能为空", trigger: "blur" }
        ],
        productUnit: [
          { required: true, message: "计量单位不能为空", trigger: "blur" }
        ],
        productCount: [
          { required: true, message: "商品数量不能为空", trigger: "blur" }
        ]
      }
    };
  },
  activated() {
    this.getDataListWithTree();
    this.getCategorys();
  },
  methods: {
    init(id) {
      this.dataForm.catId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.catId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/category/info/${this.dataForm.catId}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.category.name;
              this.dataForm.parentCid = data.category.parentCid;
              this.dataForm.catLevel = data.category.catLevel;
              this.dataForm.showStatus = data.category.showStatus;
              this.dataForm.sort = data.category.sort;
              this.dataForm.icon = data.category.icon;
              this.dataForm.productUnit = data.category.productUnit;
              this.dataForm.productCount = data.category.productCount;
            }
          });
        }
      });
    },

    getDataListWithTree() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/getParent"),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.menuList = data.data;
          this.dataListLoading = false;
        }
      });
    },
    getCategorys() {
      this.$http({
        url: this.$http.adornUrl("/product/category/listWithTree"),
        method: "get"
      }).then(({ data }) => {
        this.categorys = data.data;
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/category/${!this.dataForm.catId ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              catId: this.dataForm.catId || undefined,
              name: this.dataForm.name,
              parentCid: this.dataForm.parentCid,
              catLevel: this.dataForm.catLevel,
              showStatus: this.dataForm.showStatus,
              sort: this.dataForm.sort,
              icon: this.dataForm.icon,
              productUnit: this.dataForm.productUnit,
              productCount: this.dataForm.productCount
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    }
  }
};
</script>
