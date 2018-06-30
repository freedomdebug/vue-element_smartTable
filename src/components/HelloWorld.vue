<template>
  <div>
    <el-button type="primary" :disabled="selectIndexArray.length===0" @click="handleEditClick">编辑</el-button>
    <el-button type="warning" :disabled="selectIndexArray.length===0" @click="handleDeleteClick">删除</el-button>
    <!-- <el-button @click="toggleSelection([tableData[1], tableData[2]])">切换第二、第三行的选中状态</el-button> -->
    <el-button @click="toggleSelection()">取消选择</el-button>
    <el-table ref="multipleTable" :data="tableData"  align="left" tooltip-effect="dark" style="width: 100%" 
    @select-all="handleSelectionAll" 
    @select="handleSelect"
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
          <i class="el-icon-arrow-up" v-show="getParentChild(scope.row.code).length>0"></i>
          {{ scope.row.code }}
          <el-button type="danger" size="mini" icon="el-icon-plus" circle @click="handleAddClick(scope.row)"></el-button>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="科目名称" width="120">
        <template slot-scope="scope">{{ scope.row.name }}</template>
      </el-table-column>
      <el-table-column prop="type" label="科目类型" show-overflow-tooltip>
        <template slot-scope="scope">
            <el-select v-model="scope.row.type" placeholder="请选择">
              <el-option
                v-for="item in selectData.typeList"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
        </template>
      </el-table-column>
      <el-table-column prop="status" label="启用" show-overflow-tooltip>
        <template slot-scope="scope">
          <el-checkbox v-model="scope.row.status"></el-checkbox>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import uuidv1 from "uuid/v1";
export default {
  data() {
    return {
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
          status: 0
        }
      ],
      selectIndexArray: []
    };
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
      let _ChildRowArray = []
      _ChildRowArray = this.getParentChild(parentCode)
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
        status: 0
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
    handleSelect(selection, row){
      let _this = this;
      let selectRowIndex = this.getROwIndex(row);
      var index = this.selectIndexArray.indexOf(selectRowIndex);
      if(index>-1){
      this.selectIndexArray.splice(index, 1);
      }else{
        if (selectRowIndex !== -1) {
          console.log(selectRowIndex);
          _this.selectIndexArray.push(selectRowIndex);
        }
      }

    },
    handleSelectionAll(selection) {
      let _this = this;
      if(selection.length>0){
        selection.map((val, index) => {
          _this.handleSelect(null,val)
        });
      }
    },
    handleEditClick() {},
    handleAddClick(row) {
      let _index = this.getROwIndex(row);
      let _insertIndex = this.getInsertRowIndex(_index,row.code)
      this.addRowToTable(_insertIndex,row);
    },
    handleDeleteClick() {
      this.selectIndexArray.map(item => {
        this.deleteRowFromTable(item);
      });
      this.selectIndexArray = [];
    }
  }
};
</script>