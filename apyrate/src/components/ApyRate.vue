<template>
    <v-container class="main">
        <div class="line-apy-container">
            <template v-if="payoutsApyData">
                <LineChartApy :data="payoutsApyData" :network="network" :product="product"/>
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
        },

        product: {
            type: String,
            default: 'usd+'
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

        fillDataEts(value) {
            let clientData = value.timeData;

            let widgetDataDict = {};
            let widgetData = {
                labels: [],
                datasets: [
                    {
                        fill: false,
                        data: [],
                    }
                ]
            };

            [...clientData].forEach(item => {
                widgetDataDict[moment(item.date).format('DD.MM.YYYY')] = parseFloat(item.apy ? item.apy : 0.0).toFixed(4);
            });

            for(let key in widgetDataDict) {
                widgetData.labels.push(key);
                widgetData.datasets[0].data.push(widgetDataDict[key]);
            }

            this.payoutsApyData = widgetData;
        },

        getApyRateData() {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": this.widgetApi
                }
            };

            if (this.product === 'usd+') {
                fetch(this.widgetApi + '/dapp/payouts', fetchOptions)
                    .then(value => value.json())
                    .then(value => {
                        this.fillData(value);
                    }).catch(reason => {
                    console.log('Error get data: ' + reason);
                    this.loading = false;
                })
            } else if (this.product === 'ets') {
                fetch(this.widgetApi + '/hedge-strategies/0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf', fetchOptions)
                    .then(value => value.json())
                    .then(value => {
                        this.fillDataEts(value);
                    }).catch(reason => {
                    console.log('Error get data: ' + reason);
                    this.loading = false;
                })
            } else {
                /* TODO: add widget stub */
            }
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
