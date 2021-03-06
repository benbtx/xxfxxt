<template>
    <imp-panel>
        <kr-analysis-page slot="body">
            <kr-graph
                v-kr-loading="loading"
                slot="component-chart"
                :selectOptions="selectOptions"
                @nodeClick="nodeClick"
                @filterClick="filterClick"
                @selectChange="levelChange"
                @dateChange="dateChange"
                @lineClick="lineClick"
                @analysisHistory="analysisHistory"
                @saveAnalysis="openSave"
                model="pathRelation"
                :graph="graph"
                :date="time"
                ref="graph"
                :options="options"
            >
                <div slot="info" style="height: 100%">
                    <dimension-filtering
                        :condition="filter"
                        :visible.sync="filterVisible"
                        @submit="filterSubmit"
                    />
                    <node-details
                        :nodeInfo="item"
                        :visible.sync="nodeVisible"
                    />
                </div>
                <el-dialog
                    class="dialog"
                    slot="info"
                    :title="title"
                    :modal-append-to-body="false"
                    :visible.sync="dialogVisible"
                    :width="title == '保存分析' ? '400px' : '800px'"
                >
                    <div v-kr-loading="historyLoading">
                        <kr-table
                            :options="tableOptions"
                            v-if="title == '分析记录'"
                            :total="historyTotal"
                            :rows="historyTabe"
                            :columns="historyColumns"
                            :showPagination="true"
                            @on-page-change="historyHandlePageChange"
                        ></kr-table>
                        <el-form
                            v-if="title == '保存分析'"
                            :model="ruleForm"
                            :rules="rules"
                            ref="ruleForm"
                            label-width="100px"
                            class="demo-ruleForm"
                        >
                            <el-form-item label="名称" prop="title">
                                <el-input
                                    v-model="ruleForm.title"
                                    ref="focusInput"
                                    style="width: 200px"
                                ></el-input>
                            </el-form-item>
                            <el-form-item
                                label="描述"
                                prop="desc"
                                style="margin-top: 20px"
                            >
                                <el-input
                                    v-model="ruleForm.desc"
                                    type="textarea"
                                    placeholder="请输入内容"
                                    style="width: 200px"
                                    maxlength="50"
                                    show-word-limit
                                ></el-input>
                            </el-form-item>
                        </el-form>
                    </div>
                    <div slot="footer" class="dialog-footer">
                        <el-button @click="dialogVisible = false"
                            >关闭</el-button
                        >
                        <el-button
                            type="primary"
                            v-if="title == '保存分析'"
                            @click="save"
                            >确 定</el-button
                        >
                    </div>
                </el-dialog>
            </kr-graph>
            <kr-analysis-page-table slot="component-table" title="列表详情">
                <div slot="title-button">
                    <kr-export
                        buttonType="primary"
                        :fnGetData="fnGetData"
                        :config="dimensionRelationConfig"
                        :fileName="fileName"
                    ></kr-export>
                </div>
                <div style="padding: 20px">
                    <kr-table
                        :total="total"
                        :rows="pageData"
                        :columns="columns"
                        :current="page"
                        :indexRow="false"
                        @sort-change="sortChange"
                        @on-page-change="handlePageChange"
                    ></kr-table>
                </div>
            </kr-analysis-page-table>
        </kr-analysis-page>
        <kr-choose
            :all="all"
            :checkList="checkList"
            :max="2"
            slot="aside"
            warnTitle="请选择两个对象进行分析"
            @startAnalysis="startAnalysis"
        ></kr-choose>
        <kr-detail-info
            slot="body"
            ref="detailInfo"
            :row="rows"
            :transferAmount="condition.transferAmount.toString()"
            :date="date"
        />
    </imp-panel>
</template>
<script type="text/babel">
import relationPath from "./relationPath";
import dimensionFiltering from "../relation-network-analysis/dimension-filtering";
import nodeDetails from "../relation-network-analysis/node-details";

import { dimensionRelationConfig } from "@/components/kr-excel-export/export.config.js";

import {
    getAnalysisData,
    saveAnalysisList,
    getAnalysisList,
    delAnalysis
} from "@/api/zhfx/dimension-relation-analysis.js";
import { getSAObjectByDABH } from "@/api/zhfx/relation-network.js";
export default {
    components: {
        dimensionFiltering,
        nodeDetails
    },
    data() {
        const shelf = this;
        return {
            fileName: "",
            height: "",
            options: {
                addNode: false,
                addLink: false
            },
            tableOptions: {
                "max-height": 500
            },
            filter: {},
            checkList: [],
            time: [],
            dialogVisible: false,
            title: "",
            ruleForm: {
                title: "",
                desc: ""
            },
            rules: {
                title: [
                    { required: true, message: "请输入名称", trigger: "blur" },
                    {
                        min: 1,
                        max: 20,
                        message: "长度在 1 到 20 个字符",
                        trigger: "blur"
                    }
                ],
                desc: [
                    { required: true, message: "请输入描述", trigger: "blur" },
                    {
                        min: 1,
                        max: 50,
                        message: "长度在 1 到50 个字符",
                        trigger: "blur"
                    }
                ]
            },
            filterVisible: true,
            nodeVisible: false,
            loading: false,
            historyLoading: false,
            item: {},
            page: 1,
            pageSize: 10,
            historyPage: 1,
            historyPageSize: 10,
            historyTotal: 0,
            all: [],
            from: {
                keyWords: "",
                path: "",
                dimension: ""
            },
            condition: {
                airplaneCount: 0, //飞机同行次数
                busCount: 0, //大巴同行次数
                callAmount: 0, //通话时长
                callCount: 0, //通话次数
                dabh: this.$route.query.dabh, //案件编号
                dimensionality: "01,02,03,04,05,06,07,08,09", //维度
                startTime: null, //开始时间
                endTime: null, //结束时间
                idCards: [],
                keyword: "",
                liveCount: 0, //同住次数
                relationLevel: 5, //关系层级
                trainCount: 0, //火车同
                transferAmount: 0, //交易金额
                transferCount: 0, //交易次数
                isMultipleCases:
                    this.$route.query.isMultipleCases == "true" ? true : false,
                dwbm: this.$route.query.dwbm
            },
            selectOptions: {
                options: [
                    { label: "全部", value: "0" },
                    { label: "一级内", value: "1" },
                    { label: "二级内", value: "2" },
                    { label: "三级内", value: "3" },
                    { label: "四级内", value: "4" },
                    { label: "五级内", value: "5" }
                ],
                title: "关系层级",
                value: ""
            },
            table: [],
            pageData: [],
            total: 0,
            rows: {},
            columns: [
                {
                    title: "路径层级",
                    width: 150,
                    key: "level",
                    sortable: "custom",
                    formatter(row) {
                        return row.length;
                    }
                },
                {
                    title: "路径详情",
                    component: {
                        render(h, row) {
                            return h(relationPath, {
                                props: {
                                    path: row
                                },
                                on: {
                                    ["detail"]: data => {
                                        console.log(data);
                                        let dimensionList = data.sixRelations.map(
                                            value => value.relationType
                                        );
                                        shelf.rows = {
                                            sourceIdCard:
                                                data.startSixNodeId.nodeId,
                                            targetIdCard:
                                                data.endSixNodeId.nodeId,
                                            sourceNodeId:
                                                data.startSixNodeId.nodeId,
                                            targetNodeId:
                                                data.endSixNodeId.nodeId,
                                            dimensionList,
                                            object: `${data.startSixNodeId.realName} 与 ${data.endSixNodeId.realName}`
                                        };
                                        shelf.$refs.detailInfo.isVisible = true;
                                    }
                                }
                            });
                        }
                    }
                }
            ],
            historyColumns: [
                {
                    title: "名称",
                    key: "title",
                    "show-overflow-tooltip": true
                },
                {
                    title: "描述",
                    key: "desc",
                    "show-overflow-tooltip": true
                },
                {
                    title: "保存时间",
                    key: "createTime",
                    formatter(row) {
                        return shelf
                            .$dayjs(row.createTime)
                            .format("YYYY-MM-DD HH:mm:ss");
                    }
                },
                {
                    title: "操作",
                    component: {
                        render(h, row) {
                            return (
                                <div>
                                    <el-button
                                        type="text"
                                        onClick={() => shelf.handleDel(row)}
                                    >
                                        删除
                                    </el-button>
                                    <el-button
                                        type="text"
                                        onClick={() => shelf.handleEdit(row)}
                                    >
                                        编辑
                                    </el-button>
                                </div>
                            );
                        }
                    }
                }
            ],
            graph: {
                nodes: [],
                links: []
            },
            historyTabe: [],
            date: []
        };
    },
    computed: {
        dimensionRelationConfig() {
            return dimensionRelationConfig;
        }
    },
    mounted() {
        this.height = document.body.offsetHeight;
        this.tableOptions["max-height"] = this.height - 350;
        this.getPerson();
    },
    methods: {
        fnGetData() {
            let table = [];
            this.table.map(value => {
                let i = {
                    level: value.length,
                    relation: ""
                };
                value.map((item, index) => {
                    let relation = [];
                    relation = item.sixRelations.map(re => {
                        return re.relationType;
                    });
                    if (index == 0) {
                        i.relation =
                            item.startSixNodeId.realName +
                            "(" +
                            relation.join(",") +
                            ")" +
                            item.endSixNodeId.realName;
                    } else {
                        i.relation =
                            i.relation +
                            "(" +
                            relation.join(",") +
                            ")" +
                            item.endSixNodeId.realName;
                    }
                });
                table.push(i);
            });
            return table;
        },
        handleEdit(row) {
            this.dialogVisible = false;
            this.condition = JSON.parse(row.queryCriteria);
            this.filter = { ...this.condition };
            if (this.condition.startTime) {
                this.time = [this.condition.startTime, this.condition.endTime];
            }
            if (this.condition.relationLevel) {
                this.selectOptions = {
                    options: [...this.selectOptions.options],
                    title: "关系层级",
                    value: this.condition.relationLevel + ""
                };
            }
            this.checkList = this.condition.idCards.map(value => value.idCard);
            this.getAnalysisData();
        },
        handleDel(row) {
            this.historyLoading = true;
            delAnalysis({
                relativeId: row.relativeId
            }).then(data => {
                if (data.code == 0) {
                    this.$message.success("删除成功");
                    this.analysisHistory();
                }
            });
        },
        analysisHistory() {
            this.dialogVisible = true;
            this.historyLoading = true;
            this.title = "分析记录";
            getAnalysisList({
                caseId: this.$route.query.dabh,
                querySaveType: 2,
                pageNum: this.historyPage,
                pageSize: this.historyPageSize
            }).then(data => {
                this.historyLoading = false;
                if (data.code == 0) {
                    this.historyTabe = data.data.rows;
                    this.historyTotal = data.data.total;
                }
            });
        },
        openSave() {
            if (this.$refs.graph.nodesData.length == 0) {
                this.$message.warning("暂无数据，请添加数据后在保存");
                return;
            }
            this.dialogVisible = true;
            this.title = "保存分析";
        },
        save() {
            this.$refs.ruleForm.validate(valid => {
                if (valid) {
                    if (this.title == "保存分析") {
                        this.saveHistory();
                    } else if (this.title == "添加节点") {
                        this.addNode();
                        this.dialogVisible = false;
                    } else if (this.title == "添加关系") {
                        this.addLine();
                        this.dialogVisible = false;
                    }
                } else {
                    return false;
                }
            });
        },
        saveHistory() {
            let save = {
                nodeInfos: [],
                relations: []
            };
            this.$refs.graph.nodesData.map(value => {
                if (value.type == "add") {
                    save.nodeInfos.push(value);
                }
            });
            this.$refs.graph.linksData.map(value => {
                if (value.type == "add") {
                    save.relations.push(value);
                }
            });
            this.dialogVisible = false;
            saveAnalysisList({
                caseId: this.$route.query.dabh,
                content: JSON.stringify(save),
                queryCriteria: JSON.stringify(this.condition),
                querySaveType: 2,
                title: this.ruleForm.title,
                desc: this.ruleForm.desc
            }).then(data => {
                if (data.code == 0) {
                    this.$message.success("保存成功");
                }
            });
        },
        lineClick(data) {
            if (data.relation) {
                let dimensionList = data.relation.map(
                    value => value.relationType
                );
                this.rows = {
                    sourceIdCard: data.source.nodeId,
                    targetIdCard: data.target.nodeId,
                    sourceNodeId: data.source.nodeId,
                    targetNodeId: data.target.nodeId,
                    dimensionList,
                    object: `${data.source.realName} 与 ${data.target.realName}`
                };
                this.$refs.detailInfo.isVisible = true;
            }
        },
        nodeClick(node) {
            this.filterVisible = false;
            this.nodeVisible = true;
            this.item = node;
        },

        filterClick() {
            this.filterVisible = true;
            this.nodeVisible = false;
        },

        levelChange(level) {
            this.condition.relationLevel = level && level != "0" ? level : 5;
            if (this.condition.idCards.length != 2) {
                this.$message.warning("请选择两个分析对象");
                return;
            }
            this.getAnalysisData();
        },

        dateChange(value) {
            this.condition.startTime = value && value[0] ? value[0] : null;
            this.condition.endTime = value && value[1] ? value[1] : null;
            this.date = [this.condition.startTime, this.condition.endTime];
            if (this.condition.idCards.length != 2) {
                this.$message.warning("请选择两个分析对象");
                return;
            }
            this.getAnalysisData();
        },

        initCondition() {
            this.condition.airplaneCount = 0;
            this.condition.busCount = 0;
            this.condition.callAmount = 0;
            this.condition.callCount = 0;
            this.condition.dimensionality = "";
            this.condition.liveCount = 0;
            this.condition.trainCount = 0;
            this.condition.transferAmount = 0;
            this.condition.transferCount = 0;
        },

        filterSubmit(params) {
            let dimensionality = [];
            this.initCondition();
            params.map(value => {
                dimensionality.push(value.id);
                value.conditions.map(item => {
                    this.condition[item.key] =
                        item.value && item.value.trim() ? item.value : 0;
                });
            });
            this.condition.dimensionality = dimensionality.join(",");
            if (this.condition.idCards.length != 2) {
                this.$message.warning("请选择两个分析对象");
                return;
            }
            this.getAnalysisData();
        },

        startAnalysis(persons) {
            if (persons.length != 2) {
                this.$message.warning("请选择两个分析对象");
                return;
            }
            this.fileName =
                (persons[0].zjhm ? persons[0].mc : persons[0].ztmc) +
                "," +
                (persons[1].zjhm ? persons[1].mc : persons[1].ztmc);
            this.condition.idCards = [
                {
                    idCard: persons[0].zjhm
                        ? persons[0].zjhm
                        : persons[0].tyshxydm
                },
                {
                    idCard: persons[1].zjhm
                        ? persons[1].zjhm
                        : persons[1].tyshxydm
                }
            ];
            this.getAnalysisData();
        },

        getAnalysisData() {
            this.loading = true;
            getAnalysisData(this.condition)
                .then(data => {
                    this.loading = false;
                    this.page = 1;
                    this.pageSize = 10;
                    let nodes = [],
                        links = [],
                        lines = [],
                        num = 1,
                        nodeMap = {},
                        table = [];
                    if (data.data.set) {
                        data.data.set.map(value => {
                            nodeMap[value.nodeId] = value;
                            if (value.objectType == 1) {
                                if (num == 1) {
                                    value.isSource = true;
                                    num += 1;
                                } else {
                                    value.isTarget = true;
                                }
                            }
                            nodes.push({
                                ...value,
                                id: value.nodeId,
                                nodeName: value.sex
                                    ? `${
                                          value.realName ? value.realName : ""
                                      }(${value.sex})`
                                    : value.realName
                                    ? value.realName
                                    : ""
                            });
                        });
                    }
                    if (data.data.map) {
                        Object.values(data.data.map).map(value =>
                            value.map(item => {
                                lines.push(item);
                                table.push(item);
                            })
                        );
                        lines.map(item => {
                            item.map(line => {
                                let isHave = false;
                                links.map(value => {
                                    if (
                                        (value.source == line.startSixNodeId &&
                                            value.target ==
                                                line.endSixNodeId) ||
                                        (value.source == line.endSixNodeId &&
                                            value.target == line.startSixNodeId)
                                    ) {
                                        isHave = true;
                                    }
                                });
                                if (!isHave) {
                                    links.push({
                                        source: line.startSixNodeId,
                                        target: line.endSixNodeId,
                                        relation: line.sixRelations
                                    });
                                }
                            });
                        });

                        table.map(value => {
                            value.map(line => {
                                line.startSixNodeId =
                                    nodeMap[line.startSixNodeId];
                                line.endSixNodeId = nodeMap[line.endSixNodeId];
                            });
                        });
                    }

                    this.table = table;
                    this.table = this.table.sort((a, b) => b.length - a.length);
                    this.total = this.table.length;
                    this.getTablePage();
                    this.graph = { nodes, links };
                })
                .catch(() => {
                    this.loading = false;
                });
        },

        getPerson() {
            getSAObjectByDABH({
                dabh: this.$route.query.dabh
            }).then(data => {
                this.all = data.data;
            });
        },

        handlePageChange(current, size) {
            this.page = current;
            this.pageSize = size;
            this.getTablePage();
        },

        historyHandlePageChange(current, size) {
            this.historyPage = current;
            this.historyPageSize = size;
            this.analysisHistory();
        },

        sortChange({ order }) {
            if (order == "ascending") {
                this.table = this.table.sort((a, b) => a.length - b.length);
            } else if (order == "descending") {
                this.table = this.table.sort((a, b) => b.length - a.length);
            }
            this.getTablePage();
        },

        getTablePage() {
            let start = (this.page - 1) * this.pageSize;
            let end = this.page * this.pageSize;
            if (!this.table[end]) {
                end = this.table.length;
            }
            this.pageData = this.table.slice(start, end);
        }
    },
    watch: {
        dialogVisible() {
            if (!this.dialogVisible && this.$refs.ruleForm) {
                this.$refs.ruleForm.clearValidate();
                this.ruleForm = {
                    title: "",
                    desc: ""
                };
            }
            if (this.dialogVisible && this.title == "保存分析") {
                setTimeout(() => {
                    this.$refs.focusInput.focus();
                });
            }
        }
    }
};
</script>

<style scoped></style>
