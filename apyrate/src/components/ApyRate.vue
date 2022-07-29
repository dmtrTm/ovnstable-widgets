<template>
    <v-container class="main">
        <div class="line-apy-container">
            <template v-if="payoutsApyData">
                <LineChartApy :data="payoutsApyData" :network="network"/>
            </template>
        </div>
    </v-container>
</template>

<script>

import moment from "moment";
import LineChartApy from "./chart/LineChartApy";

export default {
    name: 'ApyRate',

    components: {
        LineChartApy
    },

    props: {
        network: {
            type: String,
            default: 'polygon'
        }
    },

    data: () => ({
        payoutsApyData: [],
    }),

    computed: {
        widgetApi() {
            if (this.network === null || this.network === 'polygon') {
                return process.env.VUE_APP_WIDGET_API_URL_POLYGON;
            } else if (this.network === 'avax') {
                return process.env.VUE_APP_WIDGET_API_URL_AVAX;
            } else if (this.network === 'bsc') {
                return process.env.VUE_APP_WIDGET_API_URL_BSC;
            } else {
                /* TODO: add widget stub */
                return '';
            }
        },
    },

    /* eslint-disable no-unused-vars,no-undef */
    watch: {
        network: function (newVal, oldVal) {
            this.getApyRateData();
        },
    },

    created() {
        this.getApyRateData();
    },

    methods: {

        fillData(value) {
            let widgetDataDictApy = {};
            let widgetDataApy = {
                labels: [],
                datasets: [
                    {
                        fill: false,
                        data: [],
                    }
                ]
            };

            [...value].reverse().forEach(item => {
                widgetDataDictApy[moment(item.payableDate).format('DD.MM.YYYY')] = parseFloat(item.annualizedYield ? item.annualizedYield : 0.0).toFixed(4);
            });

            for(let key in widgetDataDictApy) {
                widgetDataApy.labels.push(key);
                widgetDataApy.datasets[0].data.push(widgetDataDictApy[key]);
            }

            this.payoutsApyData = widgetDataApy;
        },

        getApyRateData() {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": this.widgetApi
                }
            };

            fetch(this.widgetApi + '/dapp/payouts', fetchOptions)
                .then(value => value.json())
                .then(value => {
                    this.fillData(value);
                }).catch(reason => {
                console.log('Error get data: ' + reason);
                this.loading = false;
            })
        },
    }
}
</script>


<style scoped>

/* mobile */
@media only screen and (max-width: 1400px) {
}

@media only screen and (min-width: 1400px) {
}

.main {
    font-style: normal;
    width: 100%;
    display: inline-block;

    background: none;
}

.line-apy-container {
    background: #111E37 !important;
    border: 1px solid #4C586D !important;
    box-sizing: border-box;
}

</style>
