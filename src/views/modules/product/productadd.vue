
<template>
  <div class>
    <el-row>
      <el-col :span="24" >
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-form ref="spuBaseForm" :model="product" label-width="120px" :rules="ProductInfoRules">
            <el-form-item label="商品名称" prop="productName">
              <el-input v-model="product.productName"></el-input>
            </el-form-item>
            <el-form-item label="商品描述" prop="description">
              <el-input v-model="product.productDesc"></el-input>
            </el-form-item>
            <el-form-item label="选择分类" prop="catalogId">
              <el-cascader
                filterable
                clearable
                placeholder="试试搜索：手机"
                v-model="categoryIds"
                :options="categorys"
                :props="setting"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
            <el-form-item label="选择品牌" prop="brandId">
              <el-select v-model="product.brandId" placeholder="请选择">
                <el-option
                  v-for="item in brands"
                  :key="item.brandId"
                  :label="item.brandName"
                  :value="item.brandId"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="设置价格" prop="bounds">
              <label>价格</label>
              <el-input-number
                style="width:130px"
                placeholder="价格"
                :precision="2" :step="0.1"
                v-model="product.price"
                :min="0"
                controls-position="right"
              ></el-input-number>
              <label style="margin-left:15px">成长值</label>
              <el-input-number
                style="width:130px"
                placeholder="成长值"
                v-model="product.growBounds"
                :min="0"
                controls-position="right"
              >
                <template slot="prepend">成长值</template>
              </el-input-number>
            </el-form-item>
            <el-form-item label="商品图集" prop="decript">
              <el-upload
                class="upload-demo"
                drag
                multiple
                action="proxyApi/oss/upload"
                :file-list="fileList"
                list-type="text"
                :on-success="handleSuccess"
                :before-upload="beforeAvatarUpload"
              >
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">
                  将文件拖到此处，或
                  <em>点击上传</em>
                </div>
                <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过2MB 最多上传五张</div>
              </el-upload>
            </el-form-item>
            <el-form-item>
              <el-button type="success" @click="handleClick">点击：添加</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用

  data() {
    //这里存放数据
    return {
      setting: {
        value: "catId",
        label: "name",
        children: "children"
      },
      fileList: [],
      categorys: [],
      categoryIds: [],
     
      decript: "", 
      product: {
        productName: "",
        productDesc: "",
        catalogId: 0,
        brandId: "",
        price: 0,
        growBounds: 0,
        showStatus: 1,
       //商品图集，
        images: "", 
       
      },
      brands: [],
      ProductInfoRules: {
        spuName: [
          { required: true, message: "请输入商品名字", trigger: "blur" }
        ],
        spuDescription: [
          { required: true, message: "请编写一个简单描述", trigger: "blur" }
        ],
        catalogId: [
          { required: true, message: "请选择一个分类", trigger: "blur" }
        ],
        brandId: [
          { required: true, message: "请选择一个品牌", trigger: "blur" }
        ],
        decript: [
          { required: true, message: "请上传商品详情图集", trigger: "blur" }
        ],
        images: [
          { required: true, message: "请上传商品图片集", trigger: "blur" }
        ],
        weight: [
          {
            type: "number",
            required: true,
            message: "请填写正确的重量值",
            trigger: "blur"
          }
        ]
      }
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    handleClick() {
      
       console.log("~~~~~", (this.product));
      this.$confirm("将要提交商品数据，需要一小段时间，是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/productinfo/save"),
            method: "post",
            data: this.$http.adornData(this.product, false)
          }).then(({ data }) => {
            if (data.code == 0) {
              this.$message({
                type: "success",
                message: "新增商品成功!"
              });
              this.step = 4;
            } else {
              this.$message({
                type: "error",
                message: "保存失败，原因【" + data.msg + "】"
              });
            }
          });
        })
        .catch(e => {
          // console.log("已取消");
          this.$message({
            type: "info",
            message: "已取消"
          });
        });
    },
    handleChange() {
      console.log(this.categoryIds)
      this.getCatBrands();
    },
    handleSuccess(response, file) {
     
      this.product.images += response.data + ",";
     
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    getCategorys() {
      this.$http({
        url: this.$http.adornUrl("/product/category/listWithTree"),
        method: "get"
      }).then(({ data }) => {
        this.categorys = data.data;
        
      });
    },
    getCatBrands() {
      this.product.catalogId = this.categoryIds[
        this.categoryIds.length - 1
      ];
      console.log("catelogid",this.product.catalogId);
      this.$http({
        url: this.$http.adornUrl("/product/categorybrandrelation/brands/list"),
        method: "get",
        params: this.$http.adornParams({
          catId: this.product.catalogId
        })
      }).then(({ data }) => {
        console.log(data.data);
        this.brands = data.data;
      });
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.getCategorys();
  },
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {} //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
