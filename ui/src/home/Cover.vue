<template>
  <el-dialog v-model="dialogVisible" center title="封面">
    <div style="width: 100%;display: flex;justify-content: space-between;">
      <div>
        <el-image :alt="ani.title"
                  :src="`api/file?filename=${ani['cover']}&s=${authorization()}&t=${time}`"
                  fit="cover"
                  style="border-radius: 4px;cursor: pointer;height: 260px;width: 180px;"
        />
      </div>
      <div style="width: 12px;">
      </div>
      <div style="flex: 1">
        <el-form label-width="auto">
          <el-form-item label="URL">
            <div style="width: 100%">
              <div style="width: 100%;display: flex;">
                <el-input v-model:model-value="ani.image" placeholder="https://lain.bgm.tv/pic/cover/1234.jpg"/>
                <div style="width: 8px;"/>
                <el-button :disabled="!ani.image" :loading="reLoadIng" bg icon="Refresh" text @click="reLoad"/>
              </div>
              <div style="margin-top: 8px;">
                <el-upload
                    :action="`api/upload?s=${authorization()}`"
                    :before-upload="beforeAvatarUpload"
                    :on-success="res => {
                      ani['cover'] = res.data.data
                      time = new Date().getTime()
                    }"
                    :show-file-list="false"
                    class="upload-demo"
                    drag
                    multiple
                >
                  <el-icon class="el-icon--upload">
                    <upload-filled/>
                  </el-icon>
                  <div class="el-upload__text">
                    在这里拖放文件或<em>点击上传</em>
                  </div>
                  <template #tip>
                    <div class="el-upload__tip" style="display: flex;justify-content: end;">
                      jpg / png 文件小于 1M
                    </div>
                  </template>
                </el-upload>
              </div>
            </div>
          </el-form-item>
        </el-form>
      </div>
    </div>
    <div style="display: flex;justify-content: end;">
      <el-button :loading="okLoading" bg icon="Check" text @click="ok">确定</el-button>
    </div>
  </el-dialog>
</template>
<script setup>
import {ref} from "vue";
import api from "@/js/api.js";
import {ElMessage} from "element-plus";
import {UploadFilled} from "@element-plus/icons-vue";

let reLoadIng = ref(false)
let reLoad = () => {
  reLoadIng.value = true
  api.post("api/cover", ani.value)
      .then(res => {
        time.value = new Date().getTime()
        ani.value.cover = res.data
      })
      .finally(res => {
        reLoadIng.value = false
      })
}


let dialogVisible = ref(false)

let authorization = () => {
  return window.authorization;
}

let ani = ref({})
let time = ref()

let show = (newAni) => {
  time.value = new Date().getTime()
  ani.value = JSON.parse(JSON.stringify(newAni))
  dialogVisible.value = true;
}

let okLoading = ref(false)
let ok = () => {
  okLoading.value = true
  api.put("api/ani", ani.value)
      .then(res => {
        ElMessage.success(res.message)
        emit('load')
        dialogVisible.value = false
      })
      .finally(() => {
        okLoading.value = false
      })
}

const beforeAvatarUpload = (rawFile) => {
  if (!['image/jpeg', 'image/png'].includes(rawFile.type)) {
    ElMessage.error('Avatar picture must be JPG/PNG format!')
    return false
  }
  if (rawFile.size / 1024 / 1024 > 1) {
    ElMessage.error('Avatar picture size can not exceed 1MB!')
    return false
  }
  return true
}

defineExpose({show})
const emit = defineEmits(['load'])
</script>
