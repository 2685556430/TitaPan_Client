<template>
  <el-container class="is-vertical">
    <el-header
    >
      <el-row>

        <el-col :span="6" :show-overflow-tooltip="true"
        >
          <div class="grid-content bg-purple">
            <el-image
                v-if="shared_file_info.folder"
                style="width: 30px; height: 30px; margin-top: 8px"
                :src="require('../assets/icon/folder.png')"
            >
            </el-image>
            <el-image
                v-else-if="$FileType.isApk(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/apk.png')"
            ></el-image>
            <el-image
                v-else-if="$FileType.isCode(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/code.png')"
            ></el-image>
            <el-image
                v-else-if="$FileType.isImg(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/image.png')"
            ></el-image>
            <el-image
                v-else-if="$FileType.isPdf(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/pdf.png')"
            ></el-image>
            <el-image
                v-else-if="$FileType.isWord(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/word.png')"
            ></el-image>
            <el-image
                v-else-if="$FileType.isExcel(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/excel.png')"
            ></el-image>
            <el-image
                v-else-if="$FileType.isCompress(shared_file_info.f_name)"
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/compress.png')"
            ></el-image>
            <el-image
                v-else
                style="width: 35px; height: 35px; margin-top: 8px"
                :src="require('../assets/icon/other.png')"
            ></el-image>
            {{
              shared_file_info.f_name.substr(0, 8 > shared_file_info.length ? shared_file_info.length : 8) + (shared_file_info.f_name.length > 8 ? '...' : '')
            }}
          </div>
        </el-col
        >
        <el-col :span="4"
        >
          <div class="grid-content bg-purple-light">
            {{ shared_file_info.f_size }}
          </div>
        </el-col
        >
        <el-col :span="13">???????????????{{ share_link_info.expire_date }}</el-col>
      </el-row>
    </el-header
    >
    <el-main style="line-height: 15px; padding: 0; height: 500px">
      <el-dialog
          title="????????????"
          :visible.sync="dialogTableVisible"
          width="40%"
          height="350"
          top="50px"
      >
        <el-container>
          <el-header>
            <el-button
                type="primary"
                size="mini"
                icon="el-icon-arrow-left"
                @click="last"
                style="float: left; margin-top: 10px;"
                :disabled="userPath[index] === '/'"
            >?????????</el-button
            >
          </el-header>
          <el-main>
            <el-table
                :data="userFolderList"
                height="300"
                @row-click="extractPath"
                style="margin-top: -35px"
            >

              <el-table-column
                  property="fileName"
                  label="?????????"
                  width="475%"
                  style="height: 15px"
                  align="left"
                  :show-overflow-tooltip="true"
              >
                <template slot-scope="scope">
                  <!--??????????????????-->
                  <el-link
                      type="primary"
                      :underline="false"
                      @click.stop="extractPath(scope.row)"
                  >
                    <el-image
                        style="width: 30px; height: 30px;position:absolute; bottom: -2px"
                        :src="require('../assets/icon/folder.png')"
                    >
                    </el-image>
                    <span>
                      &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;{{ scope.row.f_name }}
                    </span>
                  </el-link>
                </template>
              </el-table-column>
            </el-table>
          </el-main>
          <el-footer>
            <el-button type="warning" @click="dialogTableVisible = false;extract();">??????</el-button>
          </el-footer>
        </el-container>
      </el-dialog>

      <div style="width: 150px; align-items: center">
        ????????????
        <el-input
            v-model="form.share_code"
            placeholder="???????????????????????????..."
            maxlength="6"
        ></el-input>
        <el-button type="primary" @click="dialogTableVisible = true;userPath = '/';getUserFolderList()">??????</el-button>
      </div>
    </el-main>
  </el-container>
</template>

<script>
export default {
  name: "ExtractMain",
  data() {
    return {
      dialogTableVisible: false,
      share_code: "",
      shared_file_info: {},
      share_link_info: {},
      // ?????????????????????????????????
      userPath: '/',
      userPathStack: ['/'],
      index: 0,
      // ????????????????????????
      form: {
        savePath: this.userPath,
        share_uuid: this.$route.params.share_uuid,
        share_code: this.$route.params.share_code,
      },
      userFolderList: [],
    };
  },
  methods: {
    changeFileSize(file) {
      if (file.folder) return "-";
      var fileSizeByte = file.f_size;
      var fileSizeMsg = "";
      if (fileSizeByte < 1048576)
        fileSizeMsg = (fileSizeByte / 1024).toFixed(1) + "KB";
      else if (fileSizeByte == 1048576) fileSizeMsg = "1MB";
      else if (fileSizeByte > 1048576 && fileSizeByte < 1073741824)
        fileSizeMsg = (fileSizeByte / (1024 * 1024)).toFixed(1) + "MB";
      else if (fileSizeByte > 1048576 && fileSizeByte == 1073741824)
        fileSizeMsg = "1GB";
      else if (fileSizeByte > 1073741824 && fileSizeByte < 1099511627776)
        fileSizeMsg = (fileSizeByte / (1024 * 1024 * 1024)).toFixed(1) + "GB";
      else fileSizeMsg = "-";
      return fileSizeMsg;
    },
    getUserFolderList(){
      this.$http({
        method: "POST",
        headers:{
          "Content-Type": "multipart/form-data"
        },
        url: "/user/getUserFolderList",
        data:{ savePath: this.userPath },
      }).then((res)=>{
        if(res.status === 200 && res.data.data){
          this.userFolderList = res.data.data;
        }
        else{
          this.$message({
            type: "error",
            message: "???????????????",
          })
        }
      })
    },
    // ??????????????????
    extractPath(rowData){
      this.$http({
        method: "POST",
        headers:{
          "Content-Type": "multipart/form-data"
        },
        url: "/user/toFolder",
        data:{ savePath: this.userPath + rowData.f_name + "/" },
      }).then((res)=>{
        if(res.data.status === 200){
          // ??????????????????
          this.userPath = this.userPath + rowData.f_name + "/";
          this.userPathStack.push(this.userPath);
          this.index += 1;
          // ?????????????????????
          this.getUserFolderList();
        }
        else{
          this.$message({
            type: "error",
            message: "???????????????",
          })
        }
      })

    },
    // ????????????????????? ??????????????????
    extract() {
      this.form.savePath = this.userPath;

      this.$http({
        method: "POST",
        headers: {
          "Content-Type": "multipart/form-data",
        },
        url: "/extract",
        data: this.form
      }).then((response) => {
        if (response.status === 200) {
          if (response.data.msg === "extractSuccess") {
            this.$message({
              type: "success",
              message: "????????????!",
            });
          }
          else if(response.data.msg === "shareLinkDeleted"){
            this.$message({
              type: "error",
              message: "??????????????????????????????!",
            });
          }
          else if(response.data.msg === "shareCodeError"){
            this.$message({
              type: "error",
              message: "???????????????!",
            });
          }
          else if(response.data.msg === "shareLinkExpired"){
            this.$message({
              type: "error",
              message: "???????????????!",
            });
          }
          else if(response.data.msg === "fileNameRepetitive"){
            this.$message({
              type: "error",
              message: "?????????????????????????????????!",
            });
          }
          else{
            this.$message({
              type: "error",
              message: "????????????!",
            });
          }

        } else {
          this.$message({
            type: "error",
            message: "???????????????!",
          });
        }
      });
    },
    last(){
      this.userPath = this.userPathStack[this.index - 1];
      this.index -= 1;
      this.userPathStack.pop();
      this.getUserFolderList();
    }
  },
  components: {},
  mounted() {

    this.$http({
      method: "POST",
      headers: {
        "Content-Type": "multipart/form-data",
      },
      url: "/getSharedFileInfo",
      data: {
        share_uuid: this.form.share_uuid,
      },
    }).then((response) => {
      if (response.status === 200) {
        if (response.data.msg === "getSharedFileInfoSuccess") {
          this.shared_file_info = response.data.data;
          this.shared_file_info.f_size = this.changeFileSize(
              this.shared_file_info
          );

        } else {
          this.$message({
            type: "error",
            message: "????????????!",
          });
          this.$router.push("/user/home");
        }
      } else {
        this.$message({
          type: "error",
          message: "????????????!",
        });
        this.$router.push("/user/home");
      }
    });

    this.$http({
      method: "POST",
      headers: {
        "Content-Type": "multipart/form-data",
      },
      url: "/getShareLinkInfo",
      data: {
        share_uuid: this.form.share_uuid,
      },
    }).then((response) => {
      if (response.status === 200) {
        if (response.data.msg === "getShareLinkInfoSuccess") {
          this.share_link_info = response.data.data;
        } else {
          this.$message({
            type: "error",
            message: "???????????????!",
          });
          this.$router.push("/user/home");
        }
      } else {
        this.$message({
          type: "error",
          message: "????????????!",
        });
        this.$router.push("/user/home");
      }
    });
  },
};
</script>

<style scoped>
.el-container {
  background-color: white;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  margin-bottom: 40px;
}

.el-header {
}
</style>
