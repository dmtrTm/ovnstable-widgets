<template>
    <v-container class="main">
        <div class="line-tvl-container">
            <template v-if="payoutsTvlData">
                <LineChartTvl :data="payoutsTvlData" :totalUsdPlusValue="totalUsdPlusValue"/>
            </template>
        </div>
    </v-container>
</template>

<script>

import moment from "moment";
import LineChartTvl from "./chart/LineChartTvl";

export default {
    name: 'TvlRate',

    components: {
        LineChartTvl
    },

    props: {
        network: {
            type: String,
            default: 'polygon'
        }
    },

    data: () => ({
        payoutsTvlData: [],
        totalUsdPlusValue: null,
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
            this.getTvlRateData();
        },
    },

    created() {
        this.getTvlRateData();
    },

    methods: {

        fillData(value) {
            let widgetDataDictTvl = {};
            let widgetDataTvl = {
                labels: [],
                datasets: [
                    {
                        fill: false,
                        data: [],
                    }
                ]
            };

            [...value].reverse().forEach(item => {
                widgetDataDictTvl[moment(item.payableDate).format('DD.MM.YYYY')] = item.totalUsdPlus;
            });

            for(let key in widgetDataDictTvl) {
                widgetDataTvl.labels.push(key);
                widgetDataTvl.datasets[0].data.push(widgetDataDictTvl[key]);
            }

            this.payoutsTvlData = widgetDataTvl;

            if (value && value.length > 0) {
                this.totalUsdPlusValue = value[0].totalUsdPlus;
            }
        },

        getTvlRateData() {
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

.line-tvl-container {
    background: #111E37 !important;
    border: 1px solid #4C586D !important;
    box-sizing: border-box;
}

</style>
