<template>
    <v-container class="main">
        <v-row>
            <v-col>
                <MainCardPcv info="ALL Chains" label="Total Value Locked" :value="pcv.toString()"/>
            </v-col>
        </v-row>
        <v-row>
            <v-col>
                <MainCardApy
                        label-usd-plus="USD+ APY"
                        :value-usd-plus="apyWeek"
                        :network-usd-plus="bestChain"
                        :value-ets="apyWeekEts"
                        :network-ets="bestChainEts"
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
        bestChainEts: 'polygon',

        polygonApy: null,
        avaxApy: null,
        bscApy: null,
        opApy: null,
        etsApyPolygon: null,
        etsApyPolygonUsdc: null,
        etsApyBscBusd: null,
        etsApyRuby: null,
        etsApyGarnet: null,
        etsApyNightOvAr: null,

        polygonPcv: null,
        avaxPcv: null,
        bscPcv: null,
        opPcv: null,
        etsPcvPolygon: null,
        etsPcvPolygonUsdc: null,
        etsPcvBsc: null,
        etsPcvBscBusd: null,
        etsPcvRuby: null,
        etsPcvGarnet: null,
        etsPcvNightOvAr: null,
    }),

    computed: {
    },

    created() {
        this.getMainCardsData();
    },

    methods: {
        async getPcvData(widgetApiUrl) {
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

        async getApyData(widgetApiUrl) {
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

        async getEtsApyData(widgetApiUrl, contractAddress) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/hedge-strategies/' + contractAddress + '/avg-apy-info/week', fetchOptions)
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

        async getEtsPcvData(widgetApiUrl, contractAddress) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/hedge-strategies/' + contractAddress, fetchOptions)
                .then(value => value.json())
                .then(value => {
                    if (value && value.tvl) {
                        let result = 0;

                        if (value.tvl < 0.0001) {
                            if (value.timeData && value.timeData.length > 0) {
                                result = value.timeData[value.timeData.length - 1].tvl;
                            }
                        } else {
                            result = value.tvl;
                        }

                        return result;
                    } else {
                        return 0;
                    }
                }).catch(reason => {
                    console.log('Error get data: ' + reason);
                });
        },

        async updateApyData() {
            const [polygonApy, avaxApy, bscApy, opApy, etsApyPolygon, etsApyPolygonUsdc, etsApyBscBusd, etsApyRuby, etsApyGarnet, etsApyNightOvAr] = await Promise.all([
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_AVAX),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_BSC),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_OP),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xd52caB8AfC8ECd08b7CFa6D07e224a56F943e4c4'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xc6eca7a3b863d720393DFc62494B6eaB22567D37'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xA88F8c02eBdE678de623C6BCFC886De82e18ad00'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x270dF474f4bd2B92A45A46228683c971765E81A7'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x5651D18F066E132bB90102954429aef1a636576F')
            ]);

            this.polygonApy = polygonApy;
            this.avaxApy = avaxApy;
            this.bscApy = bscApy;
            this.opApy = opApy;

            this.etsApyPolygon = etsApyPolygon;
            this.etsApyPolygonUsdc = etsApyPolygonUsdc;
            this.etsApyBscBusd = etsApyBscBusd;
            this.etsApyRuby = etsApyRuby;
            this.etsApyGarnet = etsApyGarnet;
            this.etsApyNightOvAr = etsApyNightOvAr;
        },

        async updatePcvData() {
            const [polygonPcv, avaxPcv, bscPcv, opPcv, etsPcvPolygon, etsPcvPolygonUsdc, etsPcvBscBusd, etsPcvRuby, etsPcvGarnet, etsPcvNightOvAr] = await Promise.all([
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_AVAX),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_OP),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xd52caB8AfC8ECd08b7CFa6D07e224a56F943e4c4'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xc6eca7a3b863d720393DFc62494B6eaB22567D37'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xA88F8c02eBdE678de623C6BCFC886De82e18ad00'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x270dF474f4bd2B92A45A46228683c971765E81A7'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x5651D18F066E132bB90102954429aef1a636576F')
            ]);

            this.polygonPcv = polygonPcv;
            this.avaxPcv = avaxPcv;
            this.bscPcv = bscPcv;
            this.opPcv = opPcv;

            this.etsPcvPolygon = etsPcvPolygon;
            this.etsPcvPolygonUsdc = etsPcvPolygonUsdc;
            this.etsPcvBscBusd = etsPcvBscBusd;
            this.etsPcvRuby = etsPcvRuby;
            this.etsPcvGarnet = etsPcvGarnet;
            this.etsPcvNightOvAr = etsPcvNightOvAr;
        },

        async getMainCardsData() {
            await Promise.all([
                this.updateApyData(),
                this.updatePcvData(),
            ]);

            let etsChainDict = {
                'polygon': this.etsApyPolygon,
                // 'polygonUsdc': this.etsApyPolygonUsdc,
                'bscBusd': this.etsApyBscBusd,
                'ruby': this.etsApyRuby,
                // 'garnet': this.etsApyGarnet,
                'nightOvAr': this.etsApyNightOvAr,
            };

            for(const [key, value] of Object.entries(etsChainDict)) {
                if (!this.apyWeekEts || value > this.apyWeekEts) {
                    this.apyWeekEts = value;

                    if (key === 'polygon' || key === 'polygonUsdc') {
                        this.bestChainEts = 'polygon';
                    }

                    if (key === 'bscBusd') {
                        this.bestChainEts = 'bsc';
                    }

                    if (key === 'ruby' || key === 'garnet' || key === 'nightOvAr') {
                        this.bestChainEts = 'op';
                    }
                }
            }

            if (this.apyWeekEts) {
                this.apyWeekEts = this.$utils.formatMoney(this.apyWeekEts, 0) + '%';
            } else {
                this.apyWeekEts = '—';
            }

            let chainDict = {
                'polygon': this.polygonApy,
                'avax': this.avaxApy,
                'bsc': this.bscApy,
                'op': this.opApy,
            };

            this.bestChain = 'polygon';

            for(const [key, value] of Object.entries(chainDict)) {
                if (!this.apyWeek || value > this.apyWeek) {
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
            this.pcv = this.polygonPcv + this.avaxPcv + this.bscPcv + this.opPcv + this.etsPcvPolygon + this.etsPcvPolygonUsdc + this.etsPcvBscBusd + this.etsPcvRuby + this.etsPcvGarnet + this.etsPcvNightOvAr;

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
