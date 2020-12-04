<template>
    <div style="overflow:hidden">
        <el-table ref="custTable" :data="dataList" highlight-current-row fit height="800"
                  style="overflow-x:hidden;" highlight-current-row border element-loading-text="拼命加载中"
                  @selection-change="selectChange">
            <el-table-column
                    type="selection"
            >
            </el-table-column>
            <el-table-column
                    label="序号"
                    type="index"
                    width="180"
                    align="center"
            >
            </el-table-column>
            <!--展示列开始-->
            <el-table-column v-for="(item,index) in cloumns" :key="item.id" :align='item.align' :label="item.label"
                             :width="item.width" :prop="item.prop" show-overflow-tooltip="true">
                <!--普通字符-->
                <template slot-scope="scope">
                    <!--普通字符-->
                    <div v-if="item.type == 'text' || item.type == 'number'">{{ scope.row[item.prop] }}</div>
                    <!--图片-->
                    <img :src="scope.row[item.prop]"  v-else-if="item.type == 'image'" style="width: 50px;height: 50px;" @click="imageClick(scope.row[item.prop])"></img>
                    <!--视频-->
                    <video :src="scope.row[item.prop]" v-else-if="item.type == 'video'"></video>
                    <!--时间戳毫秒-->
                    <div v-else-if="item.type == 'datetime'">{{ scope.row[item.prop] | dateTimeFormatFunction }}</div>
                    <!--时间戳秒-->
                    <div v-else-if="item.type == 'date'">{{ scope.row[item.prop] | dateFormatFunction }}</div>
                    <!--点击事件-->
                    <el-button type="text" v-else-if="item.type == 'click'" @click="rowCellClick(scope.row,item.prop)">{{ item.showText}}</el-button>
                    <!--类型转换-->
                    <div v-else-if="item.type == 'select'" v-for="it in item.selection"><div v-if="it.key == scope.row[item.prop]">{{it.value}}</div></div>
                    <!--调用父节组件函数-->
                    <div v-else-if="item.type == 'function'">{{fatherMethod[item.prop](scope.row[item.prop])}}</div>
                </template>
            </el-table-column>
            <!--展示列结束-->
            <!--操作列开始-->
            <el-table-column  align='center' label="操作" width="300" fixed="right">
                <el-button size="mini">查看</el-button>
                <el-button type="primary" size="mini">修改</el-button>
                <el-button type="danger" size="mini">删除</el-button>
            </el-table-column>
            <!--操作列结束-->
        </el-table>

        <!--分页控件-->
        <el-pagination v-if="isShowPage"
                       @size-change="sizeChange"
                       @current-change="pageChange"
                       :current-page="pageNum"
                       :page-sizes="[10, 25, 30, 50]"
                       :page-size="pageSzie"
                       layout="total, sizes, prev, pager, next, jumper"
                       :total="total">
        </el-pagination>

        <!--查看大图开始-->
        <el-dialog
                :visible.sync="imageVisible"
                width="50%"
                :before-close="imageDialogClose">
            <div style="text-align: center;">
                <img :src="imgUrlForDialog" style="width: 500px;height: 500px;"></img>
            </div>
        </el-dialog>

        <!--查看大图结束-->
    </div>
</template>

<script>
    export default {
        name: 'TableComponent',
        props: {
            dataList: {//展示的数据
                type: Array,
                default: []
            },
            cloumns: {//列
                type: Array,
                default: []
            },
            pageNum: {//当前页数
                type: Number,
                default: 1
            },
            pageSzie: {//分页大小
                type: Number,
                default: 1
            },
            total: {//总行数
                type: Number,
                default: 10
            },
            isShowPage: {//是否显示分页控件
                type: Boolean,
                default: true
            },
            fatherMethod: { //父组件方法
                type: Object,
                default: null
            }
        },
        data() {
            return {
                imageVisible:false,//查看大图开关
                imgUrlForDialog : '',//放大后图片路径
            }
        },
        methods: {
            selectChange(selection, row) {
                // table 选中行函数
                if (selection.length === 0) { // 判断selection是否有值存在
                    this.selection = [];
                    return;
                }
                if (selection.length > 1) {
                    // 不让他多选
                    this.$refs.custTable.clearSelection()
                }
                if (selection.length > 1) {
                    // 多选框实现单选功能
                    this.$refs.custTable.clearSelection()
                    this.$refs.custTable.toggleRowSelection(selection.pop())
                    return
                }
                // 广播回去
                this.$emit("pageSelectCallBack", selection)
            },
            sizeChange(val) {
                // 分页大小改变
                this.$emit("sizeChange", val)
            },
            pageChange(val) {
                // 页码改变
                this.$emit("pageChange", val)
            },
            rowCellClick(row,prop){
                //行中某列点击事件回调
                this.$emit("rowCellClick", row,prop)
            },
            imageClick(param){
                // 图片点击放大事件
                this.imageVisible = true
                this.imgUrlForDialog = param;
            },
            imageDialogClose(){
                // 图片点击放大关闭事件
                this.imageVisible = false;
            },
        },
        filters: {
            dateTimeFormatFunction(value) {//日期格式化毫秒
                if (value == null || value == "" || value == undefined) {
                    return null;
                }
                try {
                    var k = parseInt(value);
                    if (k < 0) {
                        return ""
                    }
                    var date = new Date(value);
                    var month = date.getMonth() + 1 < 10 ? "0" + (date.getMonth() + 1) : date.getMonth() + 1;
                    var day = date.getDate() < 10 ? "0" + date.getDate() : date.getDate();
                    var hours = date.getHours() < 10 ? "0" + date.getHours() : date.getHours();
                    var minutes = date.getMinutes() < 10 ? "0" + date.getMinutes() : date.getMinutes();
                    var seconds = date.getSeconds() < 10 ? "0" + date.getSeconds() : date.getSeconds();
//            var milliseconds = date.getMilliseconds();
                    return date.getFullYear() + "-" + month + "-" + day + " " + hours + ":" + minutes + ":" + seconds;
                } catch (ex) {
                    return "";
                }
            },
            dateFormatFunction(value){//日期格式化秒
                return dateTimeFormatFunction(value*1000)
            }
        }
    };
</script>

<style scoped>

</style>
