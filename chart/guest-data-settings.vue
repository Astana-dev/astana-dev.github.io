<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!--
User: Nurzhan Sagatov
-->
<template id="guest-data-settings-template">
    <div class="panel-group accordion" id="accordionSettings">

        <div class="panel panel-default">
            <div class="panel-heading">
                <h3 class="panel-title">
                    <a class="accordion-toggle accordion-toggle-styled collapsed" data-toggle="collapse" data-parent="#accordionSettings" href="#collapse_settings_color"> Color setting </a>
                </h3>
            </div>
            <div id="collapse_settings_color" class="panel-collapse collapse">
                <div class="panel-body">
                    <div class="row">
                        <div class="col-md-12">
                            <input type="color" v-model="testColor">
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-12">
                            <table class="table">
                                <tr v-for="color in colors" :style="'background-color: rgb(' + color + ', 0.4)'">
                                    <td>{{color}}</td>
                                    <td><i class="fa fa-close"></i></td>
                                </tr>
                            </table>
                        </div>
                        <div class="col-md-12">
                            <a><i class="fa fa-plus"> Add color</i></a>
                        </div>
                    </div>
                </div>
            </div>
        </div>


    </div>
</template>

<script>
    Vue.component('guest-data-settings', {
        template: '#guest-data-settings-template',
        props: {
            colors: {
                type: Array,
                default: []
            }
        },
        data: function () {
            return {
                testColor: ""
            }
        },
        computed: {
            getColors: function () {
                return app.chartColors;
            }
        },

        watch: {

        },

        methods: {
            getTest: function () {
                
            }
        },
    });
</script>