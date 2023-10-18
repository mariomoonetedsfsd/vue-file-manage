<script>
// import myTree from "./document-tree";
export default {
  name: Document,
  // components: {
  //   myTree,
  // },
  data() {
    return {
      getdata: false,
      folders: [],
      picked: "",
      folderPath: ".",
      s3_path: "",
    }
  },
  computed: {
    max() {
      return Math.max.apply(Math, this.$store.state.userInfo.groups_list.map(item => { return item.word_level }))
    },
  },
  methods: {
    // 选择的路径：是path，循环树一层层递归上来的
    pathChanged(path) {
      console.log('father vue', path)
      this.picked = path
    },
    // 弹窗下面的确定按键，确定路径

    locationSured() {
      console.log('确定地址', this.data.s3_path, '11' , this.picked)
      this.$nextTick(() => {
        this.data.s3_path = this.picked;
        this.picked = "."
        this.$forceUpdate(); // 可以立即更新视图，跳过vue的响应机制
      });
      this.$bvModal.hide('folderModal');
    },
    // 弹窗下面的创建文件的按键
    creatFirstFolder(){
      this.$refs.myTreeRef.creatNewFolder(1)
    },
    // 选择文件夹按键，即弹窗的显示等
    chooseFolders() {
      this.$bvModal.show('folderModal')
      this.updateTable(".")
    },
    // 删除文件
    deleteFile(file) {
      // 其中axios请求包装起来了
      this.$request.post('/folder/delete', {
          sent_path: file,
        })
        .then(() => {
          console.log('删除事件', file, '==', this.currentDir)
          this.updateTable('.');
        })
        .catch((err) => {
          console.log(err);
        });
    },
    // 创建新的文件夹
    newFolder(folderName) {
      this.$request.post('/folder/newFolder', {
          current_path: this.folderPath,
          folder_name: folderName,
        })
        .then(() => {
          this.updateTable(this.folderPath);
        })
        .catch(() => {
          // eslint-disable-next-line no-console
          console.log();
        });
      
    },
    // 得到文件以及文件下面的子文件
    updateTable(path) {
      this.$request.post('folder/getAllFilesFromSelectedFolder', {
          path_name: path,
        })
        .then((res) => {
          this.folders = res.data;
          console.log('eltree is', this.folders)
          this.getdata = true;
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },

}
</script>
<template>
  <div class="mx-3 my-1">
    <div class="row">

          <!-- <div class="d-flex align-item-center mb-3">
            <div class="col-2">S3路径</div>
            <input v-model="s3_path" class="form-control flex-grow-1 text-secondary" @click="chooseFolders">
            <b-modal id="folderModal" size="lg" hide-header-close title="文件存放位置">
              <div v-if="getdata" class="text-container ">
                <myTree ref="myTreeRef" :treeData="folders" @deleteFile="deleteFile" @pathChanged="pathChanged" @newFolder="newFolder"/>
              </div>
              <template >
                <b-button size="lg" @click="creatFirstFolder">新建文件夹</b-button>
                <b-button size="lg" variant="primary" @click="locationSured">确定</b-button>
              </template>
            </b-modal>
          </div> -->
    </div>
  </div>
</template>

<style>
.text-container {
  outline: 2px solid rgb(96, 108, 228);
  /* 蓝色的2像素宽度外部框线 */
  padding: 10px;
  /* 可选，为文本添加一些内边距 */
}
</style>