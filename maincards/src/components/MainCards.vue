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
                        try {
                            return value.tvl;
                        } catch (e) {
                            return 0;
                        }
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
            let etsApy = await this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON);

            if (etsApy) {
                this.apyWeekEts = this.$utils.formatMoney(etsApy, 0) + '%';
            } else {
                this.apyWeekEts = '—';
            }

            if (polygonApy >= avaxApy && polygonApy >= bscApy) {
                this.bestChain = 'polygon';
                this.apyWeek = polygonApy;
            } else if (avaxApy >= polygonApy && avaxApy >= bscApy) {
                this.bestChain = 'avax';
                this.apyWeek = avaxApy;
            } else if (bscApy >= polygonApy && bscApy >= avaxApy) {
                this.bestChain = 'bsc';
                this.apyWeek = bscApy;
            } else {
                this.bestChain = 'polygon';
                this.apyWeek = polygonApy;
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
            let etsPcv = await this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON);

            this.pcv = polygonPcv + avaxPcv + bscPcv + etsPcv;

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
