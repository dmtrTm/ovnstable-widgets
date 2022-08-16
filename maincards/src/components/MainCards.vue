<template>
    <v-container class="main">
        <v-row>
            <v-col>
                <MainCardPcv info="ALL Chains" label="Total Value Locked" :value="pcv"/>
            </v-col>
        </v-row>
        <v-row>
            <v-col>
                <MainCardApy
                        label-usd-plus="USD+ APY"
                        :value-usd-plus="apyWeek"
                        :network-usd-plus="bestChain"
                        :value-ets="apyWeekEts"
                        info="Max 7-day APY across all chains"
                />
            </v-col>
        </v-row>
    </v-container>
</template>

<script>

import MainCardPcv from "./card/MainCardPcv";
import MainCardApy from "./card/MainCardApy";

export default {
    name: 'MainCards',

    components: {
        MainCardPcv,
        MainCardApy,
    },

    props: {
    },

    data: () => ({
        pcv: 0,
        apyWeek: null,
        apyWeekEts: null,
        bestChain: 'polygon',
    }),

    computed: {
    },

    created() {
        this.getMainCardsData();
    },

    methods: {
        getPcvData(widgetApiUrl) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/dapp/strategies', fetchOptions)
                .then(value => value.json())
                .then(value => {
                    if (value) {
                        let sum = 0.0;

                        for (let i = 0; i < value.length; i++) {
                            sum += value[i].netAssetValue;
                        }

                        return sum;
                    } else {
                        return 0;
                    }
                }).catch(reason => {
                    console.log('Error get data: ' + reason);
                });
        },

        getApyData(widgetApiUrl) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/widget/avg-apy-info/week', fetchOptions)
                .then(value => value.json())
                .then(value => {
                    if (value.value) {
                        return value.value;
                    } else {
                        return 0;
                    }
                }).catch(reason => {
                    console.log('Error get data: ' + reason);
                });
        },

        getEtsApyData(widgetApiUrl) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/hedge-strategies/0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf/avg-apy-info/week', fetchOptions)
                .then(value => value.json())
                .then(value => {
                    if (value.value) {
                        return value.value;
                    } else {
                        return 0;
                    }
                }).catch(reason => {
                    console.log('Error get data: ' + reason);
                });
        },

        getEtsPcvData(widgetApiUrl) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/hedge-strategies/0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf', fetchOptions)
                .then(value => value.json())
                .then(value => {
                    if (value && value.tvl) {
                        return value.tvl;
                    } else {
                        return 0;
                    }
                }).catch(reason => {
                    console.log('Error get data: ' + reason);
                });
        },

        async getMainCardsData() {
            this.bestChain = 'polygon';

            let polygonApy = await this.getApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON);
            let avaxApy = await this.getApyData(process.env.VUE_APP_WIDGET_API_URL_AVAX);
            let bscApy = await this.getApyData(process.env.VUE_APP_WIDGET_API_URL_BSC);
            let opApy = await this.getApyData(process.env.VUE_APP_WIDGET_API_URL_OP);
            let etsApy = await this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON);

            if (etsApy) {
                this.apyWeekEts = this.$utils.formatMoney(etsApy, 0) + '%';
            } else {
                this.apyWeekEts = '—';
            }

            let chainDict = {
                'polygon': polygonApy,
                'avax': avaxApy,
                'bsc': bscApy,
                'op': opApy,
            };

            for(const [key, value] of Object.entries(chainDict)) {
                if (value < 20 && (!this.apyWeek || value > this.apyWeek)) {
                    this.apyWeek = value;
                    this.bestChain = key;
                }
            }

            if (this.apyWeek) {
                this.apyWeek = this.$utils.formatMoney(this.apyWeek, 0) + '%';
            } else {
                this.apyWeek = '—';
            }

            this.pcv = 0.0;

            let polygonPcv = await this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON);
            let avaxPcv = await this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_AVAX);
            let bscPcv = await this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC);
            let opPcv = await this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_OP);
            let etsPcv = await this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON);

            this.pcv = polygonPcv + avaxPcv + bscPcv + opPcv + etsPcv;

            if (this.pcv) {
                this.pcv = '$ ' + this.$utils.formatMoneyComma(this.pcv, 3);
            } else {
                this.pcv = '—';
            }
        },
    }
}
</script>


<style scoped>

.main {
    font-style: normal;
    width: 100%;
    display: inline-block;

    background: none;
}

</style>
