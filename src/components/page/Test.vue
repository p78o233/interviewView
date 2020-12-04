<template>
    <table-component ref="pageTest" :dataList="tableData.dataList" :cloumns="tableData.cloumns" :pageNum="tableData.pageNum" :pageSzie="tableData.pageSzie"
                     :total="tableData.total" :isShowPage="tableData.showPage"
                      @pageSelectCallBack="selectCallBack" @rowCellClick="rowCellClick"
                     @sizeChange="sizeChange" @pageChange="pageChange"  :fatherMethod="{'func':fatherMethod}">

    </table-component>
</template>

<script>
    import TableComponent from "../common/tableViewComponent/TableComponent"
    export default {
        name: 'Test',
        components: {
            TableComponent
        },
        data() {
            return {
                tableData:{
                    dataList:[],//展示数据
                    cloumns:[//表头
                        {"align":"center","label":"id","width":180,"prop":"id","type":"text"},
                        {"align":"center","label":"userName","width":180,"prop":"userName","type":"text"},
                        {"align":"center","label":"创建时间","width":300,"prop":"createTime","type":"datetime"},
                        {"align":"center","label":"修改时间","width":300,"prop":"modifyTime","type":"datetime"},
                        {"align":"center","label":"头像","width":300,"prop":"headImg","type":"image"},
                    ],
                    pageNum:1,//页码
                    pageSzie:10,//分页大小
                    total:0,//总行数
                    showPage:true,//是否展示分页控件
                }
            }
        },
        methods:{
            selectCallBack(selection){
                // 选中哪一项的回调
                console.log(selection);
            },
            sizeChange(val){
                // 分页大小改变
                console.log(val);
            },
            pageChange(val) {
                // 页码改变
                console.log(val);
            },
            rowCellClick(row,prop){
                console.log("列名",prop)
                console.log(row)
            },
            fatherMethod(param){
                // 子组件调用父组件函数
                console.log(param)
            },
            getDataList(){
                // 获取数据
                var sendData = {
                    "pageNum":this.tableData.pageNum,
                    "pageSize":this.tableData.pageSzie
                }
                this.api.getAllUser(sendData).then(res => {
                    console.log(res.data)
                    this.tableData.dataList = res.data.list;
                    this.tableData.pageNum = res.data.pageNumber;
                    this.tableData.pageSzie = res.data.pageSize;
                    this.tableData.total = res.data.totalRow;
                })
            },
        },
        created(){
            this.getDataList();
        },
    };
</script>

<style scoped>

</style>
