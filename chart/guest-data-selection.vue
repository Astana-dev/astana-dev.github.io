<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--
User: Nurzhan Sagatov
-->
<template id="guest-data-selection-template">

    <div class="panel-group accordion">

        <div class="panel panel-default">
            <div class="panel-heading">
                <h4 class="panel-title">
                    <a class="accordion-toggle accordion-toggle-styled" data-toggle="collapse" data-parent="#accordion1" href="#collapse_1"> Data Selection </a>
                </h4>
            </div>

            <div id="collapse_1" class="panel-collapse in">
                <div class="panel-body">
                    <div class="row">

                        <h5>Dimension</h5>

                        <div class="list-group">
                            <a v-if="dimension" class="list-group-item">
                                {{dimension.key}}
                            </a>
                            <a v-else class="list-group-item disabled">
                                Select a dimension from the list below
                            </a>
                        </div>

                        <div style="width: 100%; height:200px; overflow-y:auto;">
                            <div class="list-group">
                                <a class="list-group-item disabled">
                                    RECOMMENDED COLUMNS
                                </a>
                                <a class="list-group-item"
                                   v-for="item in metaArray"
                                   @click="dimension = item"
                                   v-if="(item.type === 'string' || item.type === 'long_string' || item.type === 'small_string' && item.count > 1)"
                                   :style="dimension === item ? 'background-color: #64a2d8' : ''"> {{item.key}}
                                    <span class="badge badge-danger">
                                        {{item.count}}
                                    </span>
                                </a>
                                <a class="list-group-item disabled">
                                    ALL COLUMNS
                                </a>
                                <a class="list-group-item"
                                   v-for="item in metaArray"
                                   @click="dimension = item"
                                   :style="dimension === item ? 'background-color: #64a2d8' : ''"> {{item.key}}
                                    <span class="badge" :class="(item.type === 'string' || item.type === 'long_string' || item.type === 'small_string') ? 'badge-danger' : 'badge-info'">
                                        {{item.count}}
                                    </span>
                                </a>
                            </div>
                        </div>

                        <h5>Measure</h5>

                        <div class="row poster" v-for="(meas, index) in measureList">
                            <div :class="!index ? (!meas.measure ? 'col-md-12' : 'col-md-8') : (!meas.measure ? 'col-md-12' : 'col-md-8')">
                                <select class="form-control" v-model="meas.measure">
                                    <option :value="null">
                                        (Count of Rows)
                                    </option>
                                    <option v-for="item in metaArray" :value="item" v-if="item.type === 'number' || item.type === 'long_number' || item.type === 'small_number'">
                                        # {{item.key}}
                                    </option>
                                </select>
                            </div>
                            <div class="col-md-4" style="padding-left: 0;" v-if="meas.measure">
                                <select class="form-control" v-model="meas.measureType">
                                    <option v-for="type in measureTypeList">{{type}}</option>
                                </select>
                            </div>
                            <div class="destr" v-if="index">
                                <button class="btn btn-danger" @click="deleteMeasure(index)">
                                    x
                                </button>
                            </div>
                        </div>

                        <br>

                        <div class="row">
                            <div class="col-md-12">
                                <a @click="measureList.push({measure: null, measureType: 'Sum'})"><i class="fa fa-plus"> Add Measure</i></a>
                            </div>
                        </div>

                        {{getTest}}
                    </div>
                </div>
            </div>

            <div class="panel-heading">
                <h4 class="panel-title">
                    <a class="accordion-toggle accordion-toggle-styled collapsed" data-toggle="collapse" data-parent="#accordion2" href="#collapse_2"> Display Options </a>
                </h4>
            </div>
            <div id="collapse_2" class="panel-collapse collapse">
                <div class="panel-body" style="height:200px; overflow-y:auto;">
                    <div class="row">
                        <h5>Number of items to display</h5>
                        <div style="width: 100%">
                            <label @click="itemsDisplay.type = 'all'"><input type="radio" checked name="displayItems"> Show all</label>
                            <br>
                            <label @click="itemsDisplay.type = 'not_all'"><input type="radio" name="displayItems"> Show</label>
                            <select v-model="itemsDisplay.l" v-if="itemsDisplay.type === 'not_all'">
                                <option v-for="l in 12" :value="l">{{l}} items</option>
                            </select>
                            <br>
                            <br>
                            <label><input type="checkbox" v-model="itemsDisplay.notEmpty"/> Do not display empty</label>
                        </div>
                    </div>
                </div>
            </div>

        </div>

    </div>

</template>

<script>
    Vue.component('guest-data-selection', {
        template: '#guest-data-selection-template',
        props: {
            metaArray: {
                type: Array,
                default: []
            },

            chartType: {
                type: String,
                default: ""
            }

        },
        data: function () {
            return {
                dimension: null,

                measureList: [
                    {
                        measure: null,
                        measureType: "Sum",
                    }
                ],
                measureTypeList: [
                    "Sum",
                    "Avg",
                    "Median",
                    "Max",
                    "Min",
                    "None?",
                ],

                itemsDisplay: {
                    type: "all",
                    notEmpty: false,
                    l: 4
                }
            }
        },
        computed: {

            getTest: function () { // Вместо watch, для массива объектов
                let obj = {};
                for (let i = 0; i < this.measureList.length; i++) {
                    obj.meas = this.measureList[i].measure;
                    obj.type = this.measureList[i].measureType;
                }
                console.log("GetTest");
                this.getChartData();
                return null;
            }

        },

        watch: {

        },

        methods: {

            getChartData: function () {
                if (!this.dimension) return;
                app.showLoadBlock();
                console.log("START getChartData");
                console.log(new Date().getMinutes(), new Date().getSeconds(), new Date().getMilliseconds());
                let filteredData = app.$refs.guestDataFilter.getFilteredXmlData();
                let keyData = filteredData.map(item => item[this.dimension.key]);
                let chartData = {
                    label: this.dimension.key,
                    labels: [],
                    datasets: []
                };
                this.measureList.forEach((meas, index) => {
                    let keyMeta = {};
                    if (!meas.measure) { // count of rows
                        keyData.forEach(item => {
                            if (!keyMeta[item]) keyMeta[item] = 0;
                            keyMeta[item]++;
                        });
                    } else { // Measure col
                        let keyValData = filteredData.map(item => item[meas.measure.key]);
                        if (meas.measureType === "Sum") {
                            keyData.forEach((item, i) => {
                                if (!keyMeta[item]) keyMeta[item] = 0;
                                keyMeta[item] += keyValData[i];
                            });
                        } else if (meas.measureType === "Avg") {
                            let training = {};
                            keyData.forEach((item, i) => {
                                if (!training[item]) {
                                    training[item] = {count: 0, sum: 0};
                                }
                                training[item].count++;
                                training[item].sum += keyValData[i];
                            });
                            for (let key in training) {
                                if (training.hasOwnProperty(key)) {
                                    keyMeta[key] = training[key].sum / training[key].count;
                                }
                            }
                        } else if (meas.measureType === "Median") {
                            let training = {};
                            keyData.forEach((item, i) => {
                                if (!training[item]) {
                                    training[item] = {count: 0, items: []};
                                }
                                training[item].count++;
                                training[item].items.push(keyValData[i]);
                            });
                            for (let key in training) {
                                if (training.hasOwnProperty(key)) {
                                    training[key].items.sort((a, b) => {
                                        if (a > b) return 1;
                                        if (a < b) return -1;
                                        return 0;
                                    });
                                    if (training[key].items.length % 2) {
                                        keyMeta[key] = training[key].items[(training[key].items.length / 2 + 0.5)];
                                    } else {
                                        keyMeta[key] = (training[key].items[training[key].items.length / 2] + training[key].items[(training[key].items.length / 2 + 1)]) / 2;
                                    }
                                }
                            }
                        } else if (meas.measureType === "Max") {
                            keyData.forEach((item, i) => {
                                if (!keyMeta[item]) keyMeta[item] = 0;
                                if (keyMeta[item] < keyValData[i]) {
                                    keyMeta[item] = keyValData[i];
                                }
                            });
                        } else if (meas.measureType === "Min") {
                            keyData.forEach((item, i) => {
                                if (!keyMeta[item]) keyMeta[item] = Infinity;
                                if (keyMeta[item] > keyValData[i]) {
                                    keyMeta[item] = keyValData[i];
                                }
                            });
                        } else if (meas.measureType === "None") {

                        }
                    }
                    // console.log(keyData, keyMeta);
                    chartData.datasets[index] = {data: [], label: index};
                    for (let key in keyMeta) {
                        if (keyMeta.hasOwnProperty(key)) {
                            if (!index) chartData.labels.push(key);
                            chartData.datasets[index].data.push(keyMeta[key]);
                        }
                    }
                });
                app.chart.data = this.displayOptions(chartData);
                console.log("END getChartData");
                console.log(new Date().getMinutes(), new Date().getSeconds(), new Date().getMilliseconds());
                app.closeLoadBlock();
            },


            displayOptions: function (data) {
                console.log(data);

                if (this.itemsDisplay.notEmpty) {
                    for (let i = 0; i <  data.labels.length; i++) {
                        if (data.labels[i] === "null") {
                            data.labels.splice(i, 1);
                            data.datasets.forEach(datum => datum.data.splice(i, 1));
                            break;
                        }
                    }
                }

                if (this.itemsDisplay.type === "all") {
                    return data;
                } else {
                    data.datasets.forEach(datum => datum.data.splice(this.itemsDisplay.l, Infinity));
                    data.labels.splice(this.itemsDisplay.l, Infinity);
                }


                return data;
            },


            deleteMeasure: function (index) {
                this.measureList.splice(index, 1);
            }

        },
        mounted: function () {

        }
    });
</script>

<style scoped>
    .poster {

    }

    .destr {
        display: none;
        position: absolute;
        left: -25px;
    }

    .poster:hover .destr {
        display: block;
    }

</style>