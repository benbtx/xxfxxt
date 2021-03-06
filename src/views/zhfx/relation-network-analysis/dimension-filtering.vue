<template>
    <div class="dimension" v-if="visible">
        <div class="dimension_top">
            <div class="detail_top_text">维度筛选</div>
            <i class="el-icon-close" @click="close"></i>
        </div>
        <div class="dimension_mid">
            <kr-section-title title="分析维度选择"></kr-section-title>
            <div class="mid_options">
                <button
                    v-for="(item, index) in optionsData"
                    :key="item.id"
                    :class="{
                        options_selected: item.selected,
                        options_button: index == 0
                    }"
                    @click="getConditions(item)"
                >
                    {{ item.name }}
                </button>
            </div>
        </div>
        <div
            v-if="ruleForm.conditions.length == 0 ? false : true"
            class="dimension_bottom"
        >
            <kr-section-title title="维度条件设置"></kr-section-title>
            <el-scrollbar
                style="height: 100%;"
                wrap-style="overflow-x: hidden;"
            >
                <div class="bottom_conditions">
                    <el-form :model="ruleForm" ref="ruleForm">
                        <el-form-item
                            v-for="(item, index) in ruleForm.conditions"
                            :key="index"
                            :prop="`conditions.${index}.value`"
                            :rules="rules"
                            :label="item.label"
                        >
                            <el-input
                                v-model="item.value"
                                :placeholder="item.placeholder"
                            ></el-input>
                        </el-form-item>
                    </el-form>
                </div>
            </el-scrollbar>
        </div>
        <div class="dimension_button">
            <el-button @click="resetForm('ruleForm')">重置</el-button>
            <el-button type="primary" @click="submitForm('ruleForm')"
                >确定</el-button
            >
        </div>
    </div>
</template>

<script>
export default {
    name: "dimension-filtering",
    props: {
        visible: {
            type: Boolean,
            default: true
        },
        condition: {
            type: Object,
            default: () => {}
        }
    },
    data() {
        let reg = /((^[1-9]\d*)|^0)(\.\d{0,2}){0,1}$/;
        let checkValue = (rule, value, callback) => {
            if (value === "") {
                return callback();
            } else if (!reg.test(value)) {
                callback(new Error("请输入数字值"));
            } else {
                callback();
            }
        };
        return {
            ruleForm: {
                conditions: []
            },
            rules: [
                {
                    validator: checkValue,
                    trigger: "change"
                }
            ],
            optionsData: [
                {
                    id: "01",
                    name: "同户",
                    selected: false,
                    conditions: []
                },
                {
                    id: "02",
                    name: "企业主要人员",
                    selected: false,
                    conditions: []
                },
                {
                    id: "03",
                    name: "银行卡转账",
                    selected: false,
                    conditions: [
                        {
                            value: "",
                            key: "transferAmount",
                            placeholder: "交易金额",
                            label: "交易金额"
                        },
                        {
                            value: "",
                            key: "transferCount",
                            placeholder: "交易次数",
                            label: "交易次数"
                        }
                    ]
                },
                {
                    id: "04",
                    name: "同住",
                    selected: false,
                    conditions: [
                        {
                            value: "",
                            key: "liveCount",
                            placeholder: "同住次数",
                            label: "同住次数"
                        }
                    ]
                },
                {
                    id: "05",
                    name: "同行-飞机",
                    selected: false,
                    conditions: [
                        {
                            value: "",
                            key: "airplaneCount",
                            placeholder: "同行次数",
                            label: "飞机同行次数"
                        }
                    ]
                },
                {
                    id: "06",
                    name: "同行-大巴",
                    selected: false,
                    conditions: [
                        {
                            value: "",
                            key: "busCount",
                            placeholder: "同行次数",
                            label: "大巴同行次数"
                        }
                    ]
                },
                {
                    id: "07",
                    name: "同行-火车",
                    selected: false,
                    conditions: [
                        {
                            value: "",
                            key: "trainCount",
                            placeholder: "同行次数",
                            label: "火车同行次数"
                        }
                    ]
                },
                {
                    id: "08",
                    name: "投资",
                    selected: false,
                    conditions: []
                },
                {
                    id: "09",
                    name: "通话",
                    selected: false,
                    conditions: [
                        {
                            value: "",
                            key: "callCount",
                            placeholder: "通话次数",
                            label: "通话次数"
                        },
                        {
                            value: "",
                            key: "callAmount",
                            placeholder: "通话总时长",
                            label: "通话总时长"
                        }
                    ]
                }
            ]
        };
    },
    methods: {
        submitForm(formName) {
            if (!this.$refs[formName]) {
                var list = [];
                this.optionsData.map(value => {
                    if (value.selected) {
                        list.push(value);
                    }
                });
                this.$emit("submit", list);
                return;
            }
            this.$refs[formName].validate(valid => {
                if (valid) {
                    var list = [];
                    this.optionsData.map(value => {
                        if (value.selected) {
                            list.push(value);
                        }
                    });
                    this.$emit("submit", list);
                } else {
                    return false;
                }
            });
        },
        resetForm() {
            this.ruleForm.conditions = [];
            this.optionsData.map(el => {
                el.selected = false;
                el.conditions.map(el => (el.value = ""));
            });
        },
        close() {
            this.$emit("update:visible", !this.visible);
        },
        getConditions(val) {
            if (!val.selected) {
                val.conditions.forEach(el => {
                    this.ruleForm.conditions.push(el);
                });
            } else {
                val.conditions.forEach(el => {
                    let index = this.ruleForm.conditions.indexOf(el);
                    this.ruleForm.conditions.splice(index, 1);
                });
            }
            val.selected = !val.selected;
        }
    },
    watch: {
        condition() {
            let weidu = [];
            this.ruleForm.conditions = [];
            if (this.condition.dimensionality) {
                weidu = this.condition.dimensionality.split(",");
                this.optionsData.map(value => {
                    value.selected = false;
                });
                weidu.map(key => {
                    this.optionsData.map(value => {
                        if (key == value.id) {
                            value.selected = true;
                            value.conditions.map(d => {
                                d.value = this.condition[d.key];
                                this.ruleForm.conditions.push(d);
                            });
                        }
                    });
                });
            }
        }
    }
};
</script>

<style lang="scss" scoped>
@import "@/styles/basic-color.scss";
.dimension {
    width: 250px;
    height: 100%;
    background-color: #fff;
    font-size: 14px;
    .dimension_top {
        display: flex;
        height: 40px;
        width: 250px;
        font-size: 14px;
        justify-content: space-between;
        background-color: rgba(7, 33, 72, 1);
        padding: 10px 15px;
        color: #fff;
        .el-icon-close {
            font-size: 12px;
            padding-top: 5px;
            cursor: pointer;
        }
    }
    .dimension_mid {
        padding: 20px 0;
        height: 270px;
        border-bottom: 1px solid rgba(7, 33, 72, 0.1);
        .mid_options {
            display: flex;
            flex-wrap: wrap;
            padding: 0 15px;
            button {
                width: 100px;
                height: 32px;
                color: #52536b;
                border-radius: 5px;
                background-color: #e3e6f8;
                border: 1px solid #fff;
                border-color: #fff;
                padding: 5px 5px;
                margin-top: 10px;
                margin-left: 10px;
                outline: none;
                cursor: pointer;
                position: relative;
                &:after {
                    content: "";
                    display: block;
                    position: absolute;
                    //扩大伪类元素4个方向各10px
                    top: -8px;
                    left: -8px;
                    right: -8px;
                    bottom: -8px;
                    pointer-events: none;
                    background-color: $color-primary;
                    opacity: 0;
                    transition: all 0.3s;
                    border-radius: 5px;
                }

                &:active:after {
                    opacity: 0.3;
                    //设置初始状态
                    top: 0;
                    left: 0;
                    right: 0;
                    bottom: 0;
                    transition: 0s;
                }
            }
            .options_selected {
                color: #fff !important;
                border-color: #c6e2ff;
                background-color: $color-primary;
            }
        }
    }
    .dimension_bottom {
        padding: 20px 0 15px;
        height: calc(100% - 375px);
        border-bottom: 1px solid rgba(7, 33, 72, 0.1);
        .bottom_conditions {
            text-align: center;
            .el-form {
                padding-top: 15px;
                .el-form-item {
                    display: inline-block;
                    width: 221px;
                    height: 60px;
                }
            }
        }
    }
    .dimension_button {
        display: flex;
        justify-content: space-between;
        padding: 20px 15px;
        .el-button {
            width: 100px;
        }
    }
}
</style>
