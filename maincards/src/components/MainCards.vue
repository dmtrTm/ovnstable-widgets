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
        etsApyMoonstone: null,
        etsApyBsc: null,
        etsApyBscBusd: null,
        etsApyRuby: null,
        etsApyGarnet: null,
        etsApyWmaticUsdcUniV3: null,
        etsApyWethUsdcUniV3: null,

        polygonPcv: null,
        avaxPcv: null,
        bscPcv: null,
        opPcv: null,
        etsPcvPolygon: null,
        etsPcvPolygonUsdc: null,
        etsPcvMoonstone: null,
        etsPcvBsc: null,
        etsPcvBscBusd: null,
        etsPcvRuby: null,
        etsPcvGarnet: null,
        etsPcvWmaticUsdcUniV3: null,
        etsPcvWethUsdcUniV3: null,
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
            const [polygonApy, avaxApy, bscApy, opApy, etsApyPolygon, etsApyPolygonUsdc, etsApyMoonstone, etsApyBsc, etsApyBscBusd, etsApyRuby, etsApyGarnet, etsApyWmaticUsdcUniV3, etsApyWethUsdcUniV3] = await Promise.all([
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_AVAX),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_BSC),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_OP),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xd52caB8AfC8ECd08b7CFa6D07e224a56F943e4c4'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x719ee857Ae6cf85Cbe7284Bc45ad1f99dd5ff0dB'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xbAAc6ED05b2fEb47ef04b63018A27d80cbeA10d1'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xc6eca7a3b863d720393DFc62494B6eaB22567D37'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xA88F8c02eBdE678de623C6BCFC886De82e18ad00'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x270dF474f4bd2B92A45A46228683c971765E81A7'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xF14D4750A638677E1f127ff5317fe5fA2f13b5A0'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x3F413115Cd413BE1dCA955017aBDcfAB7E26ce10')
            ]);

            this.polygonApy = polygonApy;
            this.avaxApy = avaxApy;
            this.bscApy = bscApy;
            this.opApy = opApy;

            this.etsApyPolygon = etsApyPolygon;
            this.etsApyPolygonUsdc = etsApyPolygonUsdc;
            this.etsApyMoonstone = etsApyMoonstone;
            this.etsApyBsc = etsApyBsc;
            this.etsApyBscBusd = etsApyBscBusd;
            this.etsApyRuby = etsApyRuby;
            this.etsApyGarnet = etsApyGarnet;
            this.etsApyWmaticUsdcUniV3 = etsApyWmaticUsdcUniV3;
            this.etsApyWethUsdcUniV3 = etsApyWethUsdcUniV3;
        },

        async updatePcvData() {
            const [polygonPcv, avaxPcv, bscPcv, opPcv, etsPcvPolygon, etsPcvPolygonUsdc, etsPcvMoonstone, etsPcvBsc, etsPcvBscBusd, etsPcvRuby, etsPcvGarnet, etsPcvWmaticUsdcUniV3, etsPcvWethUsdcUniV3] = await Promise.all([
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_AVAX),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_OP),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xd52caB8AfC8ECd08b7CFa6D07e224a56F943e4c4'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x719ee857Ae6cf85Cbe7284Bc45ad1f99dd5ff0dB'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xbAAc6ED05b2fEb47ef04b63018A27d80cbeA10d1'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xc6eca7a3b863d720393DFc62494B6eaB22567D37'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xA88F8c02eBdE678de623C6BCFC886De82e18ad00'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x270dF474f4bd2B92A45A46228683c971765E81A7'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xF14D4750A638677E1f127ff5317fe5fA2f13b5A0'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x3F413115Cd413BE1dCA955017aBDcfAB7E26ce10')
            ]);

            this.polygonPcv = polygonPcv;
            this.avaxPcv = avaxPcv;
            this.bscPcv = bscPcv;
            this.opPcv = opPcv;

            this.etsPcvPolygon = etsPcvPolygon;
            this.etsPcvPolygonUsdc = etsPcvPolygonUsdc;
            this.etsPcvMoonstone = etsPcvMoonstone;
            this.etsPcvBsc = etsPcvBsc;
            this.etsPcvBscBusd = etsPcvBscBusd;
            this.etsPcvRuby = etsPcvRuby;
            this.etsPcvGarnet = etsPcvGarnet;
            this.etsPcvWmaticUsdcUniV3 = etsPcvWmaticUsdcUniV3;
            this.etsPcvWethUsdcUniV3 = etsPcvWethUsdcUniV3;
        },

        async getMainCardsData() {
            await Promise.all([
                this.updateApyData(),
                this.updatePcvData(),
            ]);

            let etsChainDict = {
                'polygon': this.etsApyPolygon,
                'polygonUsdc': this.etsApyPolygonUsdc,
                // 'moonstone': this.etsApyMoonstone,
                'bsc': this.etsApyBsc,
                'bscBusd': this.etsApyBscBusd,
                'ruby': this.etsApyRuby,
                'garnet': this.etsApyGarnet,
                // 'wmaticUsdcUniV3': this.etsApyWmaticUsdcUniV3,
                // 'wethUsdcUniV3': this.etsApyWethUsdcUniV3,
            };

            for(const [key, value] of Object.entries(etsChainDict)) {
                if (!this.apyWeekEts || value > this.apyWeekEts) {
                    this.apyWeekEts = value;

                    if (key === 'polygon' || key === 'polygonUsdc' || key === 'moonstone' || key === 'wmaticUsdcUniV3' || key === 'wethUsdcUniV3') {
                        this.bestChainEts = 'polygon';
                    }

                    if (key === 'bsc' || key === 'bscBusd') {
                        this.bestChainEts = 'bsc';
                    }

                    if (key === 'ruby' || key === 'garnet') {
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
            this.pcv = this.polygonPcv + this.avaxPcv + this.bscPcv + this.opPcv + this.etsPcvPolygon + this.etsPcvMoonstone + this.etsPcvPolygonUsdc + this.etsPcvBsc + this.etsPcvBscBusd + this.etsPcvRuby + this.etsPcvGarnet + this.etsPcvWmaticUsdcUniV3 + this.etsPcvWethUsdcUniV3;

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
