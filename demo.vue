<!--
by: xldai 2017/07 2018/01/02
name: 开课管理-教学周期管理-学期维护
nodes:		
-->
<template>
    <div class="content_box">
        <div class="mainContent" v-show="!printModel">
            <div class="top_nav">
                <Breadcrumb separator=">" style="font-size:12px">
                    <Breadcrumb-item>开课管理</Breadcrumb-item>
                    <Breadcrumb-item>教学周期管理</Breadcrumb-item>
                    <Breadcrumb-item>学期维护</Breadcrumb-item>
                </Breadcrumb>
            </div>
            <div class="select_box">
                <Form label-position="top" inline>
                    <Form-item label="状态" style="width:200px">
                        <Select style="width:100%" placeholder="全部" v-model="selectValConfig.selectStatus">
                            <Option :value="allOptionVal" :key="allOption">全部</Option>
                            <Option v-for="item in selectStatusList" :value="item.dmCode" :key="item.dmName">{{ item.dmName }}</Option>
                        </Select>
                    </Form-item>
                    <Form-item label="学期名称" style="width:200px">
                        <Input v-model="selectValConfig.selectPeriodName" placeholder="请输入学期名称"></Input>
                    </Form-item>
                    <Form-item label="学年" style="width:200px">
                        <Input v-model="selectValConfig.selectXn" placeholder="请输入学年"></Input>
                    </Form-item>
                    <Form-item label=" " style="width:200px;">
                        <Button type="primary" long @click="handleOpen">搜索</Button>
                    </Form-item>
                </Form>
            </div>
            <div class="action_button_box">
                <Dropdown @on-click="exportData">
                    <Button type="primary" style="width:90px">
                        导出
                        <Icon type="arrow-down-b"></Icon>
                    </Button>
                    <Dropdown-menu slot="list" style="width:90px">
                        <Dropdown-item name="exportChecked">导出所选</Dropdown-item>
                        <Dropdown-item name="exportAll">导出全部</Dropdown-item>
                    </Dropdown-menu>
                </Dropdown>
                <Dropdown @on-click="printData">
                    <Button type="primary" style="width:90px">
                        打印
                        <Icon type="arrow-down-b"></Icon>
                    </Button>
                    <Dropdown-menu slot="list" style="width:90px">
                        <Dropdown-item name="printChecked">打印所选</Dropdown-item>
                    </Dropdown-menu>
                </Dropdown>
                <Button type="primary" @click="dellMoreAlert">删除</Button>
                <router-link :to="{ path: '/kkgl/termManageAdd.html',params:{type:1}}">
                    <Button type="primary">添加</Button>
                </router-link>
                <Checkbox-group v-model="tableColumnsChecked" @on-change="changeTableColumns" class="screen_btn">
                    <Dropdown placement="bottom-end" trigger="custom" :visible="isShowDrop">
                        <div class="fiterBox">
                            <Button type="primary" @click="showDropFun"> 
                                <span class="icon_filter"></span>
                            </Button>
                        </div>
                        <Dropdown-menu slot="list" style="width:150px">
                            <div @mouseleave="hideDropFun">
                                <Dropdown-item>
                                    <Checkbox label="periodName">学期名称</Checkbox>
                                </Dropdown-item>
                                <Dropdown-item>
                                    <Checkbox label="startTime">学期开始日期</Checkbox>
                                </Dropdown-item>
                                <Dropdown-item>
                                    <Checkbox label="endTime">学期结束日期</Checkbox>
                                </Dropdown-item>
                                <Dropdown-item>
                                    <Checkbox label="status">状态</Checkbox>
                                </Dropdown-item>
                                <Dropdown-item>
                                    <Checkbox label="totalWeeks">学期总周数</Checkbox>
                                </Dropdown-item>
                                <Dropdown-item>
                                    <Checkbox label="xn">学年</Checkbox>
                                </Dropdown-item>
                                <Dropdown-item>
                                    <Checkbox label="term">学期</Checkbox>
                                </Dropdown-item>
                            </div>
                        </Dropdown-menu>
                    </Dropdown>
                </Checkbox-group>
            </div>
            <div class="table_box">
                <Table size="small" :data="tableData" :columns="tableColumns" border stripe @on-selection-change="getSelectRows" @on-sort-change="tableSortChange"></Table>
                <div class="page_box" v-show="booleanMark.pageMark">
                    <Page @on-change="changePage" @on-page-size-change="changeSize" :total="pageConfig.totalCount" :current="pageConfig.currentPage" :placement="pageConfig.placement" :page-size='pageConfig.pageSize' :page-size-opts="pageConfig.sizeList" size="small" show-total show-elevator show-sizer></Page>
                </div>
            </div>
        </div>
        <div class="printBox" v-show="printModel">
            <Table size="small" stripe :data="printTableData" :columns="printColumns" border></Table>
        </div>
        <Modal :mask-closable="false" v-model="dellAlertMark" title="提示信息" @on-ok="okDell">
            <h3 class="tip_style">确认要删除所选吗？</h3>
            <p>删除后无法恢复。</p>
        </Modal>
        <Modal :mask-closable="false" v-model="stopAlertMark" title="提示信息" @on-ok="okStop">
            <h3>确认要{{stopStatusText}}吗？</h3>
        </Modal>
    </div>
</template>

<script>
import nozzle from "../../libs/interface.js";
export default {
    name: 'topSelect',
    data: function () {
        return {
            allOption: "all",
            allOptionVal: "",
            stopIndex: "",
            dellId: "",
            stopId: "",
            stopStatus: "",
            stopStatusText: "",
            selectStatusList: [],
            selectedRows: "",
            selectedRowsId: "",
            term12Id: "",
            pageConfig: {
                currentPage: 1,
                pageSize: 20,
                placement: 'top',
                totalCount: 0,
                sizeList: [20, 50, 100]
            },
            selectValConfig: {
                selectStatus: "",
                selectPeriodName: "",
                selectXn: "",
                orderWay: "",
                orderType: ""
            },
            tableData: [],
            tableColumns: [],
            tableColumnsChecked: ['periodName', 'startTime', 'endTime', 'status', 'totalWeeks', 'xn', 'term'],
            defaultSort: ['periodName', 'startTime', 'endTime', 'status', 'totalWeeks', 'xn', 'term'],
            dellAlertMark: false,
            stopAlertMark: false,
            modal_loading: false,
            printModel: false,
            isShowDrop: false,
            booleanMark: {
                pageMark: false
            },
            printTableData: [],
            printColumns: [
                {
                    title: '学期名称',
                    key: 'periodName'
                },
                {
                    title: '学期开始时间',
                    key: 'startTime'
                },
                {
                    title: '学期结束日期',
                    key: 'endTime'
                },
                {
                    title: '状态',
                    key: 'status'
                },
                {
                    title: '学期总周数',
                    key: 'totalWeeks'
                },
                {
                    title: '学年',
                    key: 'xn'
                },
                {
                    title: '学期',
                    key: 'term'
                }
            ]

        }
    },
    methods: {
        /**
        * 显示表头信息下拉框
        */
        showDropFun() {
            this.isShowDrop = !this.isShowDrop;
        },
        /**
        * 隐藏表头信息下拉框
        */
        hideDropFun() {
            this.isShowDrop = false;
        },
        /**
        * 表格排序
        * @param _data {Object}
        */
        tableSortChange(_data) {
            var _this = this;
            this.tableSortColumns = _data.column;
            this.selectValConfig.orderWay = _data.key;
            if (_data.order == "desc") {
                this.selectValConfig.orderType = 2;
            } else if (_data.order == "asc") {//升序˝
                this.selectValConfig.orderType = 1;
            } else {
                this.selectValConfig.orderWay = "";
                this.selectValConfig.orderType = "";
            }
            setTimeout(function () {
                _this.loadMajorDataAjax({
                    pageSize: _this.pageConfig.pageSize,
                    currentPage: _this.pageConfig.currentPage,
                    status: _this.selectValConfig.selectStatus,
                    periodName: (_this.selectValConfig.selectPeriodName).replace(/(^\s*)|(\s*$)/g, ""),
                    xn: (_this.selectValConfig.selectXn).replace(/(^\s*)|(\s*$)/g, ""),
                    orderWay: _this.selectValConfig.orderWay,
                    orderType: _this.selectValConfig.orderType
                });
            }, 0);
        },
        /**
        * 打印
        * @param name {String}
        */
        printData(name) {
            var _this = this;
            if (name === "printChecked") {
                if (this.selectedRows.length === 0) {
                    this.$Message.info('请至少勾选一项');
                    return false;
                }
                this.printModel = true;
                this.printTableData = this.selectedRows;
                window.setTimeout(function () {
                    window.print();
                    _this.printModel = false;
                    _this.printTableData = [];
                }, 100);
                return false;
            } else if (name === "printAll") {
                return false;
            }
        },
        /**
        * 搜索
        */
        handleOpen() {
            this.loadMajorDataAjax({
                pageSize: this.pageConfig.pageSize,
                currentPage: 1,
                status: this.selectValConfig.selectStatus,
                periodName: (this.selectValConfig.selectPeriodName).replace(/(^\s*)|(\s*$)/g, ""),
                xn: (this.selectValConfig.selectXn).replace(/(^\s*)|(\s*$)/g, ""),
                orderWay: this.selectValConfig.orderWay,
                orderType: this.selectValConfig.orderType
            });
        },
        /**
        * 表头信息
        */
        getTableColumns() {
            var _this = this;
            const tableColumnList = {
                selection: {
                    title: ' ',
                    key: 'selection',
                    type: 'selection',
                    width: 60,
                    align: 'center'
                },
                index: {
                    title: ' ',
                    key: 'index',
                    type: 'index',
                    width: 60,
                    align: 'center'
                },
                periodName: {
                    title: '学期名称',
                    key: 'periodName',
                    align: 'center',
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.periodName
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.periodName
                                    }
                                })
                            ])
                        }
                },
                startTime: {
                    title: '学期开始日期',
                    key: 'startTime',
                    align: 'center',
                    sortable: true,
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.startTime
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.startTime
                                    }
                                })
                            ])
                        }
                },
                endTime: {
                    title: '学期结束日期',
                    key: 'endTime',
                    align: 'center',
                    sortable: true,
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.endTime
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.endTime
                                    }
                                })
                            ])
                        }
                },
                status: {
                    title: '状态',
                    key: 'status',
                    align: 'center',
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.status
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.status
                                    }
                                })
                            ])
                        }
                },
                totalWeeks: {
                    title: '学期总周数',
                    key: 'totalWeeks',
                    sortable: true,
                    align: 'center',
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.totalWeeks
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.totalWeeks
                                    }
                                })
                            ])
                        }
                },
                xn: {
                    title: '学年',
                    key: 'xn',
                    align: 'center',
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.xn
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.xn
                                    }
                                })
                            ])
                        }
                },
                term: {
                    title: '学期',
                    key: 'term',
                    align: 'center',
                    render (h,params) {
                            return h('Tooltip',{
                                props:{
                                    placement: (params.index > 10) ? "top" : "bottom",
                                    delay: 1000
                                }
                            },[
                                h('div',{
                                slot: "content",
                                style:{
                                    whiteSpace: 'normal'
                                },
                                domProps:{
                                    innerHTML: params.row.term
                                }
                            }),
                                h('div',{
                                    style:{
                                        wordBreak:'keep-all',
                                        whiteSpace: 'nowrap',
                                        overflow: 'hidden',
                                        textOverflow: 'ellipsis',
                                    },
                                    domProps:{
                                        innerHTML: params.row.term
                                    }
                                })
                            ])
                        }
                },
                action: {
                    title: '操作',
                    key: 'status',
                    align: 'center',
                    width: 200,
                    render: (h, params) => {
                        return h('div', 
                            {
                                class: {
                                    tableOperateBtnGroup: true
                                },
                            },
                        [
                            h('Button', {
                                props: {
                                    type: 'text',
                                    size: 'small'
                                },
                                class: {
                                    disableBtn:false,
                                    operateBtn: true,
                                    editBtn:true
                                },
                                on: {
                                    click: () => {
                                        _this.$router.push({
                                            path: '/kkgl/termManageAdd.html',
                                            query: {
                                                periodId: params.row.periodId,
                                                status: params.row.status == "已停用" ? "0" : "1",
                                                startTime: params.row.startTime,
                                                endTime: params.row.endTime,
                                                xn: params.row.xn,
                                                term: params.row.term,
                                                periodName: params.row.periodName
                                            }
                                        })
                                    }
                                }
                            }, '修改'),
                            h('span',{
                              },'|'),
                            h('Button', {
                                class: {
                                    disableBtn:false,
                                    operateBtn: true
                                },
                                props: {
                                    type: 'text',
                                    size: 'small'
                                },
                                on: {
                                    click: () => {
                                        _this.stopAlert(params.index, params.row.periodId, params.row.status);
                                    }
                                }
                            }, params.row.status == '已停用' ? '启用' : '停用'),
                            h('span',{
                              },'|'),
                            h('Button', {
                                class: {
                                    disableBtn:false,
                                    operateBtn: true
                                },
                                props: {
                                    type: 'text',
                                    size: 'small'
                                },
                                on: {
                                    click: () => {
                                        _this.dellAlert(params.row.periodId);
                                    }
                                }
                            }, '删除')
                        ]);
                    }
                }
            };

            let data = [tableColumnList.selection, tableColumnList.index];
            _this.tableColumnsChecked.sort(function (newData, oldData) {
                return _this.defaultSort.indexOf(newData) - _this.defaultSort.indexOf(oldData);
            });
            _this.tableColumnsChecked.forEach(col => data.push(tableColumnList[col]));
            data.push(tableColumnList.action);
            return data;
        },
        /**
        * 修改表头信息
        */
        changeTableColumns() {
            this.tableColumns = this.getTableColumns();
        },
        /**
        * 页码
        * @param page {String}
        */
        changePage(page) {
            this.pageConfig.currentPage = page;
            this.loadMajorDataAjax({
                pageSize: this.pageConfig.pageSize,
                currentPage: this.pageConfig.currentPage,
                status: this.selectValConfig.selectStatus,
                periodName: (this.selectValConfig.selectPeriodName).replace(/(^\s*)|(\s*$)/g, ""),
                xn: (this.selectValConfig.selectXn).replace(/(^\s*)|(\s*$)/g, ""),
                orderWay: this.selectValConfig.orderWay,
                orderType: this.selectValConfig.orderType
            });
        },
        /**
        * 改变分页pageSize
        * @param size {String}
        */
        changeSize(size) { 
            this.pageConfig.pageSize = size;
            this.loadMajorDataAjax({
                pageSize: this.pageConfig.pageSize,
                currentPage: this.pageConfig.currentPage,
                status: this.selectValConfig.selectStatus,
                periodName: (this.selectValConfig.selectPeriodName).replace(/(^\s*)|(\s*$)/g, ""),
                xn: (this.selectValConfig.selectXn).replace(/(^\s*)|(\s*$)/g, ""),
                orderWay: this.selectValConfig.orderWay,
                orderType: this.selectValConfig.orderType
            });
        },
        /**
        * 获取选中行
        * @param selection {String}
        */
        getSelectRows(selection) { 
            this.selectedRows = selection;
            var dimension = [];
            this.selectedRows.forEach(item => {
                dimension.push(item.periodId)
            });
            this.selectedRowsId = dimension;
        },
        /**
        * 导出数据
        * @param name {String}
        */
        exportData(name) {
            var _this = this;
            if (name === "exportChecked") {
                if (_this.selectedRows.length === 0) {
                    _this.$Message.info('请至少勾选一项');
                    return false;
                }
                window.location.href = nozzle.term.exporttermlist + "?isAll=0&periodId=" + _this.selectedRowsId
                    + "&status=" + _this.selectValConfig.selectStatus + "&periodName=" + _this.selectValConfig.selectPeriodName
                    + "&xn=" + _this.selectValConfig.selectXn + "&orderWay=" + _this.selectValConfig.orderWay
                    + "&orderType=" + _this.selectValConfig.orderType + "&columns=" + _this.tableColumnsChecked;
                // 导出
            } else if (name === "exportAll") {
                if (_this.pageConfig.totalCount == 0) {
                    _this.$Message.info('暂无可导出数据');
                } else {
                    window.location.href = nozzle.term.exporttermlist + "?isAll=1&status=" + _this.selectValConfig.selectStatus
                        + "&periodName=" + _this.selectValConfig.selectPeriodName + "&xn=" + _this.selectValConfig.selectXn
                        + "&orderWay=" + _this.selectValConfig.orderWay + "&orderType=" + _this.selectValConfig.orderType
                        + "&columns=" + _this.tableColumnsChecked;
                }
            }
        },
        /**
        * "启用"or"停用"
        * @param index {String}
        * @param id {String}
        * @param status {String}
        */
        stopAlert(index, id, status) {
            this.stopAlertMark = true;
            this.stopIndex = index;
            this.stopId = id;
            this.stopStatusText = (status == "已停用" ? "启用" : "停用");
            this.stopStatus = (status == "已停用" ? "1" : "0");
        },
        /**
        * 确定"启用"or"停用"
        */
        okStop() {
            this.stopGradeAjax({
                periodId: this.stopId,
                status: this.stopStatus
            });
        },
        /**
        * 单个删除 
        * 我是adev
        * @param id {String}
        */
        dellAlert(id) {
            this.dellAlertMark = true;
            this.dellId = id;
        },
        /**
        * 批量删除 
        */
        dellMoreAlert() {
            if (this.selectedRowsId.length > 0) {
                this.dellAlert(this.selectedRowsId.join(","));
            } else {
                this.$Message.info('请至少勾选一项');
            }
        },
        /**
        * 确认删除所选
        */
        okDell() {
            this.dellGradeAjax({
                periodId: this.dellId
            });
        },
        /**
        * 行政班_字典表查询 ajax
        * @param _data {Object}
        */
        selectStatusDataAjax(_data) {
            var _this = this;
            _this.$loadingView.show("termManage");
            util.ajax.get(nozzle.findDataDictionary.findDataDictionary, {
                params: _data
            }).then(function (res) {
                util.checkAjaxJson(res).thenSuccess(function (json) {
                    _this.selectStatusList = json.data;
                }).autoRun("error", "login", "termManage");
            }).catch(function (error) {
                util.checkAjaxError(error).autoRun("termManage");
            });
        },
        /**
        * 查询学期列表 ajax
        * @param _data {Object}
        */
        loadMajorDataAjax(_data) {
            var _this = this;
            this.selectedRows = [];
            this.selectedRowsId = [];            
            _this.$loadingView.show("termManage");
            util.ajax.get(nozzle.term.selecttermlist, {
                params: _data
            }).then(function (res) {
                util.checkAjaxJson(res).thenSuccess(function (json) {
                    _this.tableData = json.data.dataList;
                    _this.tableData.forEach(function (ele, i) {
                        ele.status == 1 ? ele.status = "使用中" : ele.status = "已停用";
                    })
                    _this.pageConfig.totalCount = json.data.pageConfig.totalCount;
                    _this.pageConfig.currentPage = json.data.pageConfig.currentPage;
                    if (json.data.pageConfig.totalCount == 0) {
                        _this.booleanMark.pageMark = false;
                    } else {
                        _this.booleanMark.pageMark = true;
                    }
                }).autoRun("error", "login", "termManage");
            }).catch(function (error) {
                util.checkAjaxError(error).autoRun("termManage");
            });
        },
        /**
        * 学期停用 ajax
        * @param _data {Object}
        */
        stopGradeAjax(_data) {
            var _this = this;
            _this.$loadingView.show("termManage");
            util.ajax.get(nozzle.term.stopterm, {
                params: _data
            }).then(function (res) {
                util.checkAjaxJson(res).thenSuccess(function (json) {
                    _this.$Message.info('修改成功');
                    setTimeout(function () {
                        _this.loadMajorDataAjax({
                            pageSize: _this.pageConfig.pageSize,
                            currentPage: _this.pageConfig.currentPage,
                            status: _this.selectValConfig.selectStatus,
                            periodName: (_this.selectValConfig.selectPeriodName).replace(/(^\s*)|(\s*$)/g, ""),
                            xn: (_this.selectValConfig.selectXn).replace(/(^\s*)|(\s*$)/g, ""),
                            orderWay: _this.selectValConfig.orderWay,
                            orderType: _this.selectValConfig.orderType
                        });
                    }, 0);
                }).autoRun("error", "login", "termManage");
            }).catch(function (error) {
                util.checkAjaxError(error).autoRun("termManage");
            });
        },
        /**
        * 删除学期 ajax
        * @param _data {Object}
        */
        dellGradeAjax(_data) {
            var _this = this;
            _this.$loadingView.show("termManage");
            util.ajax.get(nozzle.term.deleteterm, {
                params: _data
            }).then(function (res) {
                util.checkAjaxJson(res).thenSuccess(function (json) {
                    _this.$Message.info('删除成功');
                    setTimeout(function () {
                        _this.loadMajorDataAjax({
                            pageSize: _this.pageConfig.pageSize,
                            currentPage: _this.pageConfig.currentPage,
                            status: _this.selectValConfig.selectStatus,
                            periodName: (_this.selectValConfig.selectPeriodName).replace(/(^\s*)|(\s*$)/g, ""),
                            xn: (_this.selectValConfig.selectXn).replace(/(^\s*)|(\s*$)/g, ""),
                            orderWay: _this.selectValConfig.orderWay,
                            orderType: _this.selectValConfig.orderType
                        });
                    }, 0);
                }).autoRun("error", "login", "termManage");
            }).catch(function (error) {
                util.checkAjaxError(error).autoRun("termManage");
            });
        }
    },
    mounted() {
        this.loadMajorDataAjax({
            pageSize: this.pageConfig.pageSize,
            currentPage: this.pageConfig.currentPage
        });
        this.selectStatusDataAjax({
            dictType: "7000006"
        });
        this.changeTableColumns();
    }
}
</script>

<style scoped lang="less">
@import "../../../../libs/less/theme/theme.less";


.top_nav {
    height: 26px;
    line-height: 24px;
    font-size: 12px;
    margin-bottom: 10px;
}

.select_box {
    .baseBorder();
    width: 100%;
    padding: 20px;
    background: #ffffff;
}

.select_title {
    height: 24px;
    line-height: 14px;
    width: 100%;
    padding-right: 10px;
}

.select_content_box {
    height: 32px;
    width: 100%;
    padding-right: 10px;
}

.action_button_box {
    margin-top: 20px;
}

.action_button_box Button {
    width: 70px;
}

.action_button_box .ivu-select-selection {
    text-align: center;
}

.screen_btn {
    float: right;
}

.table_box {
    width: 100%;
    margin-top: 10px;
}
.fiterBox{
    .ivu-btn{
        width: 33px;
        padding: 6px 0px;
    }
    .icon_filter{
        display: inline-block;
        width: 14px;
        height: 14px;
        position: relative;
        top: 3px;
        background: url("../../assets/images/icon_filter.png") no-repeat center center;
    }
}
.page_box {
    width: 100%;
    background: #ffffff;
    text-align: center;
    height: 50px;
    line-height: 50px;
    border-bottom: solid 1px #e0e1e2;
    border-right: solid 1px #e0e1e2;
    border-left: solid 1px #e0e1e2;
}

.tip_style {
    color: @baseThemeColor;
}

.ivu-table .ivu-btn {
    border: none;
    color: @baseThemeColor;
    background-color: transparent;
}

.ivu-table th {
    background-color: #ffffff;
}

.ivu-table .demo-table-info-row td {
    background-color: #f8f8f8;
}

.ivu-table-cell div {
    color: @baseThemeColor;
}

a {
    color: #444444;
}

a:hover {
    color: #444444;
}
</style>
