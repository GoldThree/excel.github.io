<template>
  <div>
    <input type="file" ref="upload" accept=".xls,.xlsx" class="outputlist_upload">
    <button @click="downloadExcel">下载</button>
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
        outputs: [],
        excelData: [],
        selectData: [],
        allNumber: 0
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
          this.allNumber = ws.length-1
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

            let count = valueNumber[key]
            let rate = count/this.allNumber
            var sheetData = {
              testData: key,
              count: count,
              all: this.allNumber,
              rate: rate,
              f: key * rate
            }
            that.outputs.push(sheetData);
          }
          that.selectData = that.outputs
          that.excelData = that.outputs
          console.log('result---------->', that.outputs)
          this.$refs.upload.value = '';
        };
        fileReader.readAsBinaryString(files[0]);
      },
      export2Excel() {
        var that = this;
        require.ensure([], () => {
          const { export_json_to_excel } = require('@/excel/export2Excel'); //这里必须使用绝对路径，使用@/+存放export2Excel的路径
          const tHeader = ['数值（x）','数量','样本总数','频率f(x)','xf(x)']; // 导出的表头名信息
          const filterVal = ['testData','count', 'all', 'rate','f']; // 导出的表头字段名，需要导出表格字段名
          const list = that.excelData;
          const data = that.formatJson(filterVal, list);

          export_json_to_excel(tHeader, data, '下载数据');// 导出的表格名称，根据需要自己命名
        })
      },
      downloadExcel() {
        this.excelData = this.selectData //你要导出的数据list。
        this.export2Excel()
        // this.$confirm('确定下载列表文件?', '提示', {
        //   confirmButtonText: '确定',
        //   cancelButtonText: '取消',
        //   type: 'warning'
        // }).then(() => {
        //   this.excelData = this.selectData //你要导出的数据list。
        //   this.export2Excel()
        // }).catch(() => {
        // })
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v => filterVal.map(j => v[j]))
      },
  }
  }
</script>

<style scoped>

</style>
