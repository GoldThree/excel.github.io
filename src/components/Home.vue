<template>
  <div>
    <input type="file" ref="upload" accept=".xls,.xlsx" class="outputlist_upload">
    <div style="display: flex;width: 300px;margin-top: 30px; margin-left: auto; margin-right: auto; background-color: #409eff;">
      <div style="color: white">数值</div>
      <div style="margin-left: 180px;color: white">数量</div>
    </div>
    <div style="width: 300px; margin-left: auto; margin-right: auto;">
      <div
        style="display: flex"
        v-for="item in outputs"
        :key="item"
      >
        <div style="width: 50px">{{ item.testData }}</div>
        <div style="margin-left: 180px">{{ item.count }}</div>
      </div>
    </div>
  </div>

</template>

<script>
  import XLSX from 'xlsx'
  export default {
    data() {
      return {
        outputs: []
      }
    },
    mounted() {
      this.$refs.upload.addEventListener('change', e => {//绑定监听表格导入事件
        this.readExcel(e);
      })
    },
    methods: {
      readExcel(e) {//表格导入
        var that = this;
        const files = e.target.files;
        console.log(files);
        if(files.length<=0){//如果没有文件名
          return false;
        }else if(!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())){
          this.$Message.error('上传格式不正确，请上传xls或者xlsx格式');
          return false;
        }

        const fileReader = new FileReader();
        fileReader.onload = (ev) => {
          const data = ev.target.result;
          const workbook = XLSX.read(data, {
            type: 'binary'
          });
          const wsname = workbook.SheetNames[0];//取第一张表
          const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]);//生成json表格内容
          let valueNumber = {}
          for (var i = 1; i < ws.length; i++) {
            let value = ws[i].__EMPTY_1
            if (!valueNumber[value]) {
              valueNumber[value] = 1
            } else {
              valueNumber[value] += 1
            }
          }
          that.outputs = [];//清空接收数据
          for (var key in valueNumber) {
            var sheetData = {
              testData: key,
              count: valueNumber[key]
            }
            that.outputs.push(sheetData);
          }
          console.log('result---------->', that.outputs)
          this.$refs.upload.value = '';
        };
        fileReader.readAsBinaryString(files[0]);
      }
    },
  }
</script>

<style scoped>

</style>
