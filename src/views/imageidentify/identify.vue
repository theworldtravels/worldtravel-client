<template>
  <e-container>
    <div class="content-wrapper top">
      <h1>图片智能识别</h1>
      <el-upload ref="upload" action="http://localhost:8088/ocr/photoRecognize" list-type="picture-card"
        :auto-upload="false" :on-preview="handlePictureCardPreview" :on-remove="handleRemove">
        <i class="el-icon-plus"></i>
      </el-upload>
      <el-dialog :visible.sync="dialogVisible">
        <img width="100%" :src="dialogImageUrl" alt="">
      </el-dialog>
      <br>
      <el-button @click="handleButtonClick" type="primary">智能图片识别</el-button>
    </div>
    <div class="content-wrapper bottom">
      <el-input type="textarea" rows="11" placeholder="上传图片静候片刻即可获得结果>3<" v-model="textarea">
      </el-input>
    </div>
  </e-container>
</template>

<style scoped>
.content-wrapper {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  height: 50vh;
  background-size: cover;
  background-position: center;
}

.top {
  background-image: url('https://ts1.cn.mm.bing.net/th/id/R-C.d1370d3cb22b94779fec3e2093eb2d79?rik=01XfxVmbWGvIsw&riu=http%3a%2f%2fimg.pconline.com.cn%2fimages%2fupload%2fupc%2ftx%2fphotoblog%2f1105%2f06%2fc7%2f7553816_7553816_1304671459501.jpg&ehk=ZPCbD12pAHWmQpuKTn4d9pTdqtJvGK2%2bB%2bzgLGw9xXo%3d&risl=&pid=ImgRaw&r=0');
}

.upload {
  margin-top: 20px;
}

h1 {
  font-family: "Arial", "Microsoft YaHei", "黑体", "宋体", sans-serif;
  color: white;
}

.bottom {
  margin-top: 20px;
}

.el-input__inner {
  overflow-y: auto;
  max-height: 300px;
}
</style>


<script>
import axios from 'axios';

export default {
  data() {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      textarea: '',
      result: '',
      recognizeResult: ''
    };
  },
  methods: {
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    async resolveResult(recognizeResult) {
      try {
        this.result = '你能提取文字中的的景点并帮我讲解一下其中的景点吗，文字如下：' + recognizeResult;
        const response = await axios.post('https://burn.hair/v1/chat/completions', {
          model: 'gpt-4-0125-preview',
          messages: [
            {
              role: 'user',
              content: this.result
            }
          ],
          temperature: 0.6
        }, {
          headers: {
            Authorization: 'Bearer sk-2WuIEbmG5ihT7VTH4057B6D576Bf4c78A214E30bBb92C647'
          }
        });
        const content = response.data.choices[0].message.content;
        this.textarea = content;
      } catch (error) {
        console.error('请求出错：', error);
      }
    },
    async handleButtonClick() {
      try {
        const formData = new FormData();
        formData.append('file', this.$refs.upload.uploadFiles[0].raw);

        const apiUrl = 'http://localhost:8088/ocr/photoRecognize';

        const response = await axios.post(apiUrl, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        });

        this.recognizeResult = response.data;
        this.resolveResult(this.recognizeResult);
      } catch (error) {
        console.error('图片识别出错：', error);
      }
    }
  }
}
</script>
