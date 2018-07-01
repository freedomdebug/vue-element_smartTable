<template>
  <div>
    <el-dialog
      title="导入"
      :visible.sync="importDialogVisible"
      width="30%"
      center>
      <span>
        模板下载
      </span>
      <span>
        <el-upload
          class="upload-demo"
          ref="upload"
          action="https://jsonplaceholder.typicode.com/posts/"
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          :file-list="fileList"
          :multiple="false" 
          :show-file-list="false"
          :auto-upload="false">
          <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
          <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
          <div slot="tip" class="el-upload__tip">只能上传xls/xlsx文件，且不超过500kb</div>
        </el-upload>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="importDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleSubmitImport">确 定</el-button>
      </span>
    </el-dialog>

    <el-row style="text-align:right;margin:10px">
      <el-button type="primary"  @click="handleSaveClick">保存</el-button>
      <el-button type="primary" :disabled="selectIndexArray.length===0" @click="handleEditClick">编辑</el-button>
      <el-button type="warning" :disabled="selectIndexArray.length===0" @click="handleDeleteClick">删除</el-button>
      <!-- <el-button @click="toggleSelection([tableData[1], tableData[2]])">切换第二、第三行的选中状态</el-button> -->
      <el-button @click="toggleSelection()">取消选择</el-button>
      <el-button type="primary" @click="handleExport">导出</el-button>
      <el-button type="primary" @click="handleImport">导入</el-button>
    </el-row>

    <el-table ref="multipleTable" :data="tableData"  align="left" tooltip-effect="dark" style="width: 100%" 
    @selection-change="handleSelectChange" 
    height="500">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" width="50">
      </el-table-column>
      <el-table-column prop="level" label="科目级别" width="120">
        <template slot-scope="scope">{{ scope.row.code | level }}</template>
      </el-table-column>
      <el-table-column prop="code" label="科目编码" width="240">
        <template slot-scope="scope">
          <span v-show="!scope.row.editStauts"   :style="{'padding-left':getLevel(scope.row.code)*10+'px'}">
            <i class="el-icon-arrow-up" v-show="getParentChild(scope.row.code).length>0 && !scope.row.editStauts"></i>
            {{ scope.row.code }}
          </span>
          <el-tooltip class="item" effect="light"  content="1001.xx" placement="top-start">
            <el-input v-model="scope.row.code" placeholder="科目编码"  v-show="scope.row.editStauts"></el-input>
          </el-tooltip>
          <el-button type="danger" size="mini" icon="el-icon-plus" circle @click="handleAddClick(scope.row)"  v-show="!scope.row.editStauts"></el-button>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="科目名称" width="120">
        <template slot-scope="scope">
          <span v-show="!scope.row.editStauts">{{scope.row.name}}</span>
          <el-input v-model="scope.row.name" placeholder="科目名称"  v-show="scope.row.editStauts"></el-input>
          </template>
      </el-table-column>
      <el-table-column prop="type" label="科目类型" show-overflow-tooltip>
        <template slot-scope="scope">
           <span v-show="!scope.row.editStauts">{{scope.row.type}}</span>
            <el-select v-model="scope.row.type" placeholder="请选择" v-show="scope.row.editStauts">
              <el-option
                v-for="item in selectData.typeList"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
        </template>
      </el-table-column>
      <el-table-column prop="status" label="启用" show-overflow-tooltip align="center">
        <template slot-scope="scope">
          <el-checkbox v-model="scope.row.status"></el-checkbox>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import Vue from 'vue'
import uuidv1 from "uuid/v1"
export default {
  data() {
    return {
      importDialogVisible:false,
      fileList: [],
      selectData: {
        typeList:[{
          label:"保安",
          value:"0"
        },{
          label:"总经办",
          value:"1"
        }]
      },
      tableData: [
        {
          id: "0409ac1a-0840-4c65-a8c9-547bf5a77bea",
          level: 0,
          code: "1001",
          name: "root",
          type: "0",
          status: true
        }
      ],
      selectIndexArray: []
    };
  },
  mounted(){
    this.tableData.map(item=>{item.editStauts = false})
  },
  filters:{
    level(parentCode){
      let level
      let num = parentCode.split('.').length
      let changeNum = ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九']; //changeNum[0] = "零"
      let unit = ["", "十", "百", "千", "万"];
      num = parseInt(num);
      let getWan = (temp) => {
          let strArr = temp.toString().split("").reverse();
          let newNum = "";
          for (var i = 0; i < strArr.length; i++) {
              newNum = (i == 0 && strArr[i] == 0 ? "" : (i > 0 && strArr[i] == 0 && strArr[i - 1] == 0 ? "" : changeNum[strArr[i]] + (strArr[i] == 0 ? unit[0] : unit[i]))) + newNum;
          }
          return newNum;
      }
      let overWan = Math.floor(num / 10000);
      let noWan = num % 10000;
      if (noWan.toString().length < 4) noWan = "0" + noWan;
      level = overWan ? getWan(overWan) + "万" + getWan(noWan) : getWan(num);
      return level + "级"
    }
  },
  methods: {
    getLevel(parentCode){
      return  parentCode.split('.').length
    },
    getParentChild(parentCode){
      let _this = this
      let _ChildRow = this.tableData.filter(item=>{
          return item.code.indexOf(parentCode)>-1 && item.code.length === parentCode.length + 3
      })
      return _ChildRow
    },
    getInsertRowIndex(parentIndex,parentCode){
      let _insertIndex = 0 
      let _ChildRowArray = this.tableData.filter(item=>{
        return item.code.substr(0,parentCode.length+1) === parentCode+"."
      })
      _insertIndex = parentIndex + _ChildRowArray.length + 1
      return _insertIndex
    },
    getChildMaxCode(parentCode){
      let _maxCodeNum = 0
      let _filterMaxCodeRow = null
      let _ChildRow = this.tableData.filter(item=>{
        return item.code.indexOf(parentCode)>-1 && item.code.length === parentCode.length + 3
      })
      let _bigChildRow = _ChildRow.sort((item1,item2)=>{
        return parseInt(item1.code.replace(/\./g, ''))<parseInt(item2.code.replace(/\./g, ''))
      })
      if(_bigChildRow.length>0){
        if(_bigChildRow[0].code === parentCode){
          _maxCodeNum = 0
        }else{
        _maxCodeNum = _bigChildRow.length>0?parseInt(_bigChildRow[0].code.replace(parentCode,'').replace(/\./g, '')):0
        }
      }
      return _maxCodeNum
    },
    generateCode(parentCode){
      let _myCode = ""
      let _myCodeNum = this.getChildMaxCode(parentCode)+1
      _myCode = parentCode + "." + (_myCodeNum<10?'0'+_myCodeNum.toString():_myCodeNum.toString())
      return _myCode
    },
    getDefaultRow(index,parentRow) {
      return {
        id: uuidv1(),
        level: 0,
        code: this.generateCode(parentRow.code),
        name: "root",
        type: "0",
        status: true
      };
    },
    getROwIndex(row) {
      let _index = this.tableData.lastIndexOf(row);
      return _index;
    },
    addRowToTable(index, parentRow) {
      let _row = this.getDefaultRow(index,parentRow);
      this.tableData.splice(index, 0, _row);
    },
    deleteRowFromTable(index) {
      this.tableData.splice(index, 1);
    },
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
      this.selectIndexArray = [];
    },
    handleSelectChange(selection){
      console.log(selection)
      let _this = this;
      _this.selectIndexArray = []
      selection.map(row=>{
        let selectRowIndex = this.getROwIndex(row);
        if (selectRowIndex > -1) {
            console.log(selectRowIndex);
            _this.selectIndexArray.push(selectRowIndex);
        }
      })
    },
    handleEditClick() {
      let _this = this
      this.selectIndexArray.map(index=>{
        let _newItem = _this.tableData[index]
        _newItem.editStauts = true
        Vue.set(_this.tableData, index, _newItem)
      })
    },
    handleAddClick(row) {
      let _index = this.getROwIndex(row);
      let _insertIndex = this.getInsertRowIndex(_index,row.code)
      this.addRowToTable(_insertIndex,row);
    },
    handleDeleteClick() {
      let _this = this
      if(this.selectIndexArray.indexOf(0)>-1){
        this.$message({
          message: 'root节点不可删除',
          type: 'warning'
        });
        return
      }
      let i = 0 //js array+splice index change，so becareful
      _this.selectIndexArray.map(item => {
        console.log(item)
        _this.tableData.splice(item-i, 1);
        i++
      });
      _this.selectIndexArray = [];
    },
    handleSaveClick(){
      let _this = this
      this.selectIndexArray.map(index=>{
        let _newItem = _this.tableData[index]
        _newItem.editStauts = false
        Vue.set(_this.tableData, index, _newItem)
      })
      _this.selectIndexArray = []
      this.toggleSelection()
    },
    handleExport(){
      if(this.selectIndexArray.length>0){
        this.$message({
          message: '当前有未保存信息',
          type: 'warning'
        });
      }else{
        //api
      }
    },
    handleImport(){
       if(this.selectIndexArray.length>0){
        this.$message({
          message: '当前有未保存信息',
          type: 'warning'
        });
      }else{
        this.importDialogVisible = true
      }
      
    },
    handleSubmitImport(){
      this.importDialogVisible = false
    },
    submitUpload() {
      this.$refs.upload.submit();
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    }
  }
};
</script>