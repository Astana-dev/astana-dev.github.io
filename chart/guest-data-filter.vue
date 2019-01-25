<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--
User: Nurzhan Sagatov
-->
<template id="guest-data-filter-template">

    <div class="panel-group accordion" id="accordionDictItems">
        <div class="panel panel-default">
            <div class="panel-heading">
                <h3 class="panel-title">
                    <a class="accordion-toggle accordion-toggle-styled collapsed" data-toggle="collapse" data-parent="#accordionDictItems" href="#collapse_meta"> Col Name </a>
                </h3>
            </div>
            <div id="collapse_meta" class="panel-collapse collapse">
                <div class="panel-body">
                    <table class="table">
                        <tr v-for="(meta, index) in metaArray" v-if="dicts[meta.key].length > 1" :class="meta.filter ? 'success' : ''">
                            <td><input type="checkbox" v-model="meta.filter"></td>
                            <td>{{meta.key}}</td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>

        <div class="panel panel-default" v-for="(meta, index) in getFiltersMetaArray">
            <div class="panel-heading">
                <h4 class="panel-title">
                    <a class="accordion-toggle accordion-toggle-styled collapsed" data-toggle="collapse" data-parent="#accordionDictItems" :href="'#collapse_meta' + index"> {{meta.key}} </a>
                </h4>
            </div>
            <div :id="'collapse_meta' + index" class="panel-collapse collapse">
                <div class="panel-body">
                    <table class="table" v-if="!loadSetDictItem">
                        <tr class="info" v-for="item in meta.items"
                            @click="deleteDictItemInFilter(meta.key, item)">
                            <td>{{item}}</td>
                        </tr>
                        <tr v-for="item in dictsCopy[meta.key]"
                            v-if="item" @click="setDictItemToFilter(meta.key, item)">
                            <td>{{item}}</td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    Vue.component('guest-data-filter', {
        template: '#guest-data-filter-template',
        props: {
            metaArray: {
                type: Array,
                default: []
            },

            dicts: {
                type: Object,
                default: {}
            },

            dictsMeta: {
                type: Object,
                default: {}
            }

        },
        data: function () {
            return {
                dictsCopy: {},
                loadSetDictItem: false
            }
        },
        computed: {
            getFiltersMetaArray: function () {
                return this.metaArray.filter(dict => {
                    if (dict.filter) return dict;
                })
            },
        },

        watch: {
            getFiltersMetaArray: function () {
                this.createDictsCopy();
            }
        },

        methods: {

            createDictsCopy: function () {
                this.dictsCopy = {};
                this.getFiltersMetaArray.forEach(meta => {
                    this.dictsCopy[meta.key] = JSON.parse(JSON.stringify(this.dicts[meta.key]));
                    if (typeof meta.items === "undefined") meta.items = [];
                    meta.items.forEach(filteredMeta => {
                        for (let i = 0; i < this.dictsCopy[meta.key].length; i++) {
                            if (this.dictsCopy[meta.key][i] === filteredMeta) {
                                this.dictsCopy[meta.key].splice(i, 1);
                                i--;
                            }
                        }
                    })
                });
            },

            deleteDictItemInFilter: function (key, item) {
                for (let i = 0; i < this.dictsMeta[key].items.length; i++) {
                    if (this.dictsMeta[key].items[i] === item) {
                        this.dictsMeta[key].items.splice(i, 1);
                        this.createDictsCopy();
                        app.$refs.guestDataSelection.getChartData();
                        break;
                    }
                }
            },

            setDictItemToFilter: function (key, item) {
                this.loadSetDictItem = true;
                if (this.dictsMeta[key].filter) {
                    if (!this.dictsMeta[key].items) this.dictsMeta[key].items = [];
                    this.dictsMeta[key].items.push(item);
                } else {
                    this.dictsMeta[key].items = [];
                }
                this.createDictsCopy();
                app.$refs.guestDataSelection.getChartData();
                this.loadSetDictItem = false;
            },

            getFilteredXmlData: function () {
                let filteredXmlDataIndexes = {}; // Хеш-таблица, подсчет индексов проходящих фильтрацию
                let filteredXmlData = []; // Результат фильтрации
                let filteredMetaCount = 0; // Количество используемых фильтров
                this.getFiltersMetaArray.forEach(meta => {
                    if (meta.items && meta.items.length) filteredMetaCount++;
                });
                xmlData.forEach((datum, index) => { // создает Хеш-таблицу
                    filteredXmlDataIndexes[index] = 0;
                });
                for (let key in this.dictsMeta) {
                    if (this.dictsMeta.hasOwnProperty(key) && this.dictsMeta[key].filter && this.dictsMeta[key].items) { // используемые столбцы для фильтра
                        this.dictsMeta[key].items.forEach(item => {
                            xmlData.forEach((datum, index) => {
                                if (item === datum[key]) {
                                    filteredXmlDataIndexes[index]++; // отмечаем проход индекса, через фильтрацию
                                }
                            });
                        });
                    }
                }
                for (let indexKey in filteredXmlDataIndexes) {
                    if (filteredXmlDataIndexes.hasOwnProperty(indexKey) && filteredMetaCount === filteredXmlDataIndexes[indexKey])
                        filteredXmlData.push(xmlData[indexKey]); // добавляем прошедшие индексы
                }
                return filteredXmlData;
            },

        },
        mounted: function () {

        }
    });
</script>