<template>
  <el-container>
    <el-main>
      <el-row :gutter="20">
        <el-col :span="24">
          <el-card class="art-card">
            <div slot="header">
              <span>撰写新文章</span>
            </div>
            <el-form ref="artform"
                     :model="artform"
                     :rules="formRules"
                     label-width="100px"
                     class="el-formtable">
              <el-row :gutter="20">
                <el-col :span="14">
                  <el-form-item label="文章标题"
                                prop="title">
                    <el-input v-model="artform.title"
                              placeholder="请输入文章标题" />
                  </el-form-item>
                </el-col>
                <el-col :span="6">
                  <el-form-item label="分类目录"
                                prop="catg">
                    <el-select v-model="artform.catg"
                               placeholder="请选择">
                      <el-option v-for="item in catgList"
                                 :key="item._id"
                                 :label="item.categoryname"
                                 :value="item.categoryname">
                      </el-option>
                    </el-select>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <el-col :span="14">
                  <el-form-item label="文章描述"
                                prop="desc">
                    <el-input v-model="artform.desc"
                              placeholder="请输入文章描述" />
                  </el-form-item>
                </el-col>
                <el-col :span="6">
                  <el-form-item label="文章转载"
                                prop="origin">
                    <el-select v-model="artform.origin"
                               placeholder="请选择">
                      <el-option v-for="item in origin"
                                 :key="item._id"
                                 :label="item.originname"
                                 :value="item._id">
                      </el-option>
                    </el-select>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <el-col :span="24">
                  <el-form-item label="缩略图"
                                prop="banner">
                    <el-input v-model="artform.banner"
                              type="hidden"
                              style="display: none" />
                    <el-upload ref="upload"
                               :show-file-list="false"
                               :action="regionUrl"
                               :http-request="uploadImg"
                               :before-upload="beforeUpload"
                               class="el-formLoader"
                               list-type="picture-card">
                      <img v-if="artform.banner"
                           :src="artform.banner"
                           class="el-formbanner" />
                      <i v-else
                         class="el-icon-plus" />
                    </el-upload>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <el-col :span="24">
                  <el-form-item label="文章标签"
                                prop="tag">
                    <el-select v-model="artform.tag"
                               multiple
                               filterable
                               default-first-option
                               style="width: 500px;"
                               placeholder="请选择文章标签">
                      <el-option v-for="item in tagList"
                                 :key="item._id"
                                 :label="item.tagname"
                                 :value="item.tagname" />
                    </el-select>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row :gutter="20">
                <el-col :span="24">
                  <el-form-item label="文章内容"
                                prop="content">
                    <!-- 引入自定义markdown组件这里 -->
                    <v-mark-editor ref="markEditor"
                                   :regions="regionUrl"
                                   v-model="artform.content"></v-mark-editor>
                  </el-form-item>
                </el-col>
              </el-row>
              <!-- <el-form-item label="发布日期">
                <el-date-picker @change="changeDate"
                                v-model="form.curdate"
                                type="date"
                                placeholder="选择日期"
                                format="yyyy 年 MM 月 dd 日"
                                value-format="yyyy-MM-dd">
                </el-date-picker>
              </el-form-item> -->
              <el-row :gutter="20">
                <el-col :span="24">
                  <el-form-item>
                    <el-button type="primary"
                               @click="submitArticle('artform')">
                      发布文章
                    </el-button>
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
          </el-card>
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
import vMarkEditor from '@/components/mark-editor';
import { getQNToken } from '@/api/qiniu';
import { insertArticle, getArtDetl, editArticle } from '@/api/article';
import { getAllCatgs } from '@/api/category';
import { getAllTags } from '@/api/tag';
import * as qiniu from 'qiniu-js';
import './index.scss';

export default {
  name: 'ArtPub',
  components: {
    vMarkEditor
  },
  data () {
    return {
      artform: {
        title: '',
        desc: '',
        banner: '',
        tag: '',
        catg: '',
        origin: '',
        content: ''
      },
      formRules: {
        title: [{ required: true, message: '标题不能为空', trigger: 'blur' }],
        desc: [{ required: true, message: '描述不能为空', trigger: 'blur' }],
        banner: [{
          required: true,
          message: '缩略图不能为空',
          trigger: 'blur'
        }],
        catg: [{
          required: true,
          message: '请至少选择一个分类',
          trigger: 'change'
        }],
        origin: [{
          required: true,
          message: '请至少选择一个转载',
          trigger: 'change'
        }],
        tag: [{
          type: 'array',
          required: true,
          message: '请至少选择一个标签',
          trigger: 'change'
        }],
        content: [{
          required: true,
          message: '内容不能为空',
          trigger: 'blur'
        }]
      },
      tagList: [
        // { _id: "1", tagname: "javascript" },
        // { _id: "2", tagname: "nodejs", },
        // { _id: "3", tagname: "前端" }
      ],
      catgList: [
        // { _id: "1", categoryname: '测试分类1' },
        // { _id: "2", categoryname: '测试分类2' },
        // { _id: "3", categoryname: '测试分类3' }
      ],
      origin: [
        { _id: 0, originname: '原创' },
        { _id: 1, originname: '转载' },
        { _id: 2, originname: '混合' }
      ],
      // 七牛云配置
      regionUrl: 'https://upload-z2.qiniup.com', // 七牛云的上传地址，我这里是华南区
      qiniulink: 'https://static.ikmoons.com/' // 这是七牛云空间的外链默认域名
    };
  },
  created () {
    console.log('处于开发状态：' + process.env.VUE_APP_BASE_API);
    console.log('处于开发状态：' + process.env.NODE_ENV);
  },
  mounted () {
    this.getAllTagsList();
    this.getAllCatgList();
    const method = this.$route.query.method;
    if (method === 'edit') {
      this.getArtDetails();
    }
  },
  // 该方法被混入实例当中
  methods: {
    submitArticle () {
      const param = {
        id: this.$route.query.id,
        title: this.artform.title,
        desc: this.artform.desc,
        banner: this.artform.banner,
        tag: this.artform.tag,
        content: this.artform.content,
        catg: this.artform.catg,
        origin: this.artform.origin
      };
      this.$refs.artform.validate(valid => {
        if (valid) {
          if (this.$route.query.method !== 'edit') {
            insertArticle(param).then(res => {
              if (res.data.code === 1) {
                this.$message({
                  message: res.data.msg,
                  type: 'success'
                });
                this.$router.push({
                  name: 'ArtList'
                });
              } else {
                this.$message({
                  message: res.data.msg,
                  type: 'error'
                });
              }
            });
          } else {
            editArticle(param).then(res => {
              if (res.data.code === 1) {
                this.$message({
                  message: res.data.msg,
                  type: 'success'
                });
                this.$router.push({
                  name: 'ArtList'
                });
              } else {
                this.$message({
                  message: res.data.msg,
                  type: 'error'
                });
              }
            });
          }
        }
      });
    },
    // 获取标签列表
    getAllTagsList () {
      getAllTags().then(res => {
        if (res.data.code === 1) {
          this.tagList = res.data.result;
        }
      });
    },
    // 获取分类列表
    getAllCatgList () {
      getAllCatgs().then(res => {
        if (res.data.code === 1) {
          this.catgList = res.data.result;
        }
      });
    },
    // 上传缩略图
    uploadImg (req) {
      const { file } = req;
      // 重命名要上传的文件
      const key =
        'blogs/image/' +
        new Date().getTime() +
        Math.floor(Math.random() * 100) +
        '.' + file.name.split('.')[1];
      /*
      * fname: string，文件原文件名.
      * params: object，用来放置自定义变量;
      * mimeType: null || array，用来限制上传文件类型，为 null 时表示不对文件类型限制；
      * 限制类型放到数组里： ["image/png", "image/jpeg", "image/gif"]
      */
      const putExtra = {
        fname: file.name,
        params: {},
        mimeType: ["image/png", "image/jpeg", "image/gif"]
      };
      const config = {
        useCdnDomain: true,
        region: qiniu.region.z2 // 华南
      };
      const options = {
        quality: 0.92,
        noCompressIfLarger: true,
        maxWidth: 740,
        maxHeight: 400
      }
      getQNToken().then(res => {
        const self = this;
        const getToken = res.data.result.token;
        qiniu.compressImage(file, options).then(data => {
          const observable = qiniu.upload(data.dist, key, getToken, putExtra, config)
          observable.subscribe({
            next (res) {
              // 提示文件进度
              let totalPercent = 0;
              totalPercent += res.total.percent;
              const h = self.$createElement;
              if (Object.is(totalPercent, 100)) {
                self.$notify({
                  title: '上传成功',
                  message: h('div', { style: 'display:flex; flex-flow:column wrap; justify-content:flex-start; align-content:center;' }, [
                    h('span', null, `当前上传进度:${totalPercent}%`),
                    h('span', null, `已上传大小，单位为字节:${res.total.loaded}`),
                    h('span', null, `本次上传的总量控制信息:${res.total.size}`),
                  ]),
                  type: 'success'
                })
              }
            },
            error (err) {
              // 提示错误
              self.$message({
                message: err,
                type: 'error'
              });
            },
            complete (res) {
              // 完成后的操作
              self.artform.banner = self.qiniulink + res.key;
            }
          })
        }).catch(err => {
          self.$message({
            message: err,
            type: 'error'
          });
        })
      }).catch(err => {
        console.error(err);
      });
    },
    // 上传图片之前验证文件合法性
    beforeUpload (file) {
      const isJPG = file.type === 'image/jpeg' || file.type === 'image/png';
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!');
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!');
      }
      return isJPG && isLt2M;
    },
    // 获取文章详情
    getArtDetails () {
      const param = {
        id: this.$route.query.id
      };
      getArtDetl(param).then(res => {
        if (res.data.code === 1) {
          this.artform = res.data.ArtDetlData;
          const markCont = res.data.ArtDetlData.content;
          // 将content数据通过$refs传递给子组件
          this.$refs.markEditor.saveEditorMark(markCont);
        }
      });
    }
  }
};
</script>
