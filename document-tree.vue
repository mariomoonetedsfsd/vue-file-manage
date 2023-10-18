<script>
import appConfig from "@/app.config";
/**
 * File-manager component
 */
export default {
  name: "myTree",
  page: {
    title: "File Manager",
    meta: [{ name: "description", content: appConfig.description }],
  },
  props: {
    treeData: {
      type: Array,
    },
  },
  data() {
    return {
      folderName: "",
      newPath: "",
      files: [], 
      folderInputName: "",
      picked: "./files",
      // picked_File: "./files/A1_Main/no_file_selected.txt",
      currentDir: "./files",
      showContextMenuFlag: false,
      showInputFirst: false,
      newFolderName: '',
      contextMenuAlias: ".",
      contextMenuLeft: 0,
      contextMenuTop: 0,
      shouldBlur: true,
    };
  },
  methods: {
    creatNewFolder(folder) {
      if( folder === 1 ){
        this.showInputFirst = true;
        this.$nextTick(() => {
          console.log('$refs', this.$refs, this.$refs.folderInputFirst, this.$refs.folderInputFirst, )
          this.$refs.folderInputFirst.focus();
        });
      } else {
        this.folderInputName = folder     // 所创建的父级文件夹的名字
        this.$nextTick(() => {
          console.log('$refs', this.$refs, this.$refs.folderInput, this.$refs.folderInput[0], )
          this.$refs.folderInput[0].focus();
        });
      }
      this.picked = this.contextMenuAlias;
    },
    confirmCreateFolder() {
      const folderName = this.newFolderName.trim();
      if (folderName) {     
        if(this.showInputFirst) {
          this.$emit('newFolder', folderName);
          this.showInputFirst = false;
        } else {
          this.folderName = folderName;
          this.newFolder();
          this.shouldBlur = false;
          this.newFolderName = '';
          this.folderInputName = "";
        }
      }
    },
    handleBlur() {
      console.log('@@@blur发生了')
      if (!this.newFolderName.trim() && this.shouldBlur) {
        this.folderName = "新建文件夹"
        this.newFolder();
        this.newFolderName = '';
        this.folderInputName = "";
      }
      this.shouldBlur = true;
    },
    handleBlurFirst() {
      if (!this.newFolderName.trim() && this.shouldBlur) {
        this.folderName = "新建文件夹"
        this.$emit('newFolder', this.folderName);
        this.showInputFirst = false;
      }
      this.shouldBlur = true;
    },

    newFolder() {    
      this.$request.post('/folder/newFolder', {
          current_path: this.picked,
          folder_name: this.folderName,
        })
        .then(() => {
          this.updateTable(this.picked);
        })
        .catch(() => {
          // eslint-disable-next-line no-console
          console.log();
        });
    },

    updateTable(path) {
      console.log('updateTabel---getAllFiles 传参是', path, this.files)
      this.$request.post('/folder/getAllFilesFromSelectedFolder', {
          path_name: path,
        })
        .then((res) => {
          this.files = res.data;
          // this.$set(this, 'files', res.data);
          console.log("updateTable--this.files is", this.picked, this.files, path);
        })
        .catch((err) => {
          console.log(err);
        });
    },

    selectSwitchFiles: function (path, fileName, isDir, event) {
      event.stopPropagation();
      // 判断是否重复点击，若重复点击则不显示子文件夹
      if (this.picked === path) {
        var lastSlashIndex = this.picked.lastIndexOf("/");
        if (lastSlashIndex !== -1) {
          this.picked = this.picked.slice(0, lastSlashIndex);
        }
        return;
      }
      this.picked = path
      this.currentDir = path.slice(0, path.lastIndexOf("/"))

      this.$request.post('/folder/getAllFilesFromSelectedFolder', {
          path_name: this.picked,
        })
        .then((res) => {
          this.files = res.data;
          this.newPath = this.picked;
        })
        .catch((err) => {
          console.log(err);
        });
    },
    // 隐藏鼠标右键之后的菜单栏
    hideContextMenu() {
        this.showContextMenuFlag = false;
        // document.removeEventListener("click", this.hideContextMenu);
    },
    handleDelete(file) {
      this.$emit('deleteFile', file)
    },
    deleteFile(file) {
      this.currentDir = file.substring(0, file.lastIndexOf("/"));
      // console.log("@@things related delete", deleteItem, this.currentDir);
      this.$request.post('/folder/delete', {
          sent_path: file,
        })
        .then(() => {
          console.log('删除事件', file, '==', this.currentDir)
          this.updateTable(this.currentDir);
        })
        .catch((err) => {
          console.log(err);
        });
    },
    handleRightClick(event, file) {
      this.showContextMenuFlag = true;
      this.contextMenuAlias = file;
      this.contextMenuLeft = `${event.clientX}px`;
      this.contextMenuTop = `${event.clientY}px`;
      // 添加事件监听器，以便在其他地方点击时关闭右键菜单
      document.addEventListener("click", this.hideContextMenu);
      console.log("鼠标右键事件发生了,file and picked is", file, 'picked', this.picked, this.contextMenuAlias);
      event.stopPropagation();
    },
    // 监测路径的改变，向上级传递路径，从而在input框展示路径
    pathChanged(path){
      this.newPath = path
    }
  },
  watch: {
    picked(nv, ov) {
      if (nv !== ov) {
        console.log('=picked changed=', this.picked)
      }
    },
    newPath(nv, ov){
      if (nv !== ov) {
        console.log('newPath changed=', nv)
        this.$emit('pathChanged', nv)
    }
  },
    files(nv, ov) {
      if(nv !== ov) {
        console.log('files changed 递归层级的不同', nv)
      }
    }
  },
};
</script>
<template>
  <div class="tree-item">
    <div>
      <!-- 创建一级文件的input框 -->
      <div v-if="showInputFirst" style="padding-left: 7.5px;">
        <i class="mdi mdi-folder text-warning me-2"></i>
        <input v-model="newFolderName" ref="folderInputFirst" @keyup.enter="confirmCreateFolder" @blur="handleBlurFirst" placeholder="新建文件夹" />
      </div>
      <div v-for="alias2 in treeData" :key="alias2.name">
         <div v-if="alias2.is_dir == true" :class="{ collapsed: picked === alias2.path }">
            <i class="mdi mdi-chevron-right accor-down-icon ms-auto"></i>
            <i class="mdi mdi-folder folder-open-icon text-warning me-2"></i>
            <span :class="{ selected: picked === alias2.path }" @click="selectSwitchFiles(alias2.path, alias2.names, alias2.is_dir, $event)"  @contextmenu.prevent="handleRightClick($event, alias2.path)" 
              class="item-title">{{ alias2.name }}</span>
          </div>
        <div v-show="picked === alias2.path" class="item-childen">
          <!-- 新建文件夹的input框 -->
          <div v-if="folderInputName === alias2.path" style="padding-left: 7.5px;">
            <i class="mdi mdi-folder text-warning me-2"></i>
            <input v-model="newFolderName" ref="folderInput" @keyup.enter="confirmCreateFolder" @blur="handleBlur" placeholder="新建文件夹" />
          </div>
          <myTree :treeData="files" @deleteFile="deleteFile" @pathChanged="pathChanged" ></myTree>
        </div>
      </div>

    </div>
    <!-- 鼠标右键的菜单栏 -->
    <div v-show="showContextMenuFlag" class="context-menu" :style="{ left: contextMenuLeft, top: contextMenuTop }">
      <div @click="creatNewFolder(contextMenuAlias)" class="context-menu-item">新建文件夹</div>
      <div @click="handleDelete(contextMenuAlias)" class="context-menu-item">删除</div>
      <div @click="hideContextMenu" class="context-menu-item">取消</div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.selected {
  background-color: #cbdefa;
}
.tree-item {
  cursor: pointer;
  font-size: 20px;

  .item-title {
    padding: 3px 8px;

    &:hover {
      background: #eee;
    }
  }

  .item-childen {
    padding-left: 20px;
  }
  
  div {
    &.collapsed {
      i.accor-down-icon {
        &:before {
          content: "\F0140";             // "\F0142"
        }
      }
      i.folder-open-icon {
        &:before {
          content: "\F0770";            // "\F024B"     
        }
      }
    }
  }
}

.context-menu {
  position: fixed;
  background-color: white;
  padding: 5px;
  box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3);
  border-radius: 8px;
}

.context-menu-item {
  padding: 8px 16px;
  cursor: pointer;
}

.context-menu-item:hover {
  background-color: #eee;
}
</style>
