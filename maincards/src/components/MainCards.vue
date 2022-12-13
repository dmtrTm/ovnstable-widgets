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
                        :ets-name="bestEtsName"
                        :network-ets="bestChainEts"
                        info="Max 30-day APY across all chains"
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
        bestEtsName: null,
        bestChain: 'polygon',
        bestChainEts: 'polygon',

        polygonApy: null,
        avaxApy: null,
        bscApy: null,
        opApy: null,
        etsApyPolygon: null,
        etsApyBscBusd: null,
        etsApyRuby: null,
        etsApyNightOvAr: null,
        etsApyAlpha: null,
        etsApyBeta: null,
        etsApyGamma: null,
        etsApyDelta: null,
        etsApyEpsilon: null,
        etsApyZeta: null,
        etsApyEta: null,
        etsApyUniAlpha: null,
        etsApyUniBeta: null,

        polygonPcv: null,
        avaxPcv: null,
        bscPcv: null,
        opPcv: null,
        etsPcvPolygon: null,
        etsPcvBscBusd: null,
        etsPcvRuby: null,
        etsPcvNightOvAr: null,
        etsPcvAlpha: null,
        etsPcvBeta: null,
        etsPcvGamma: null,
        etsPcvDelta: null,
        etsPcvEpsilon: null,
        etsPcvZeta: null,
        etsPcvEta: null,
        etsPcvUniAlpha: null,
        etsPcvUniBeta: null,

        polygonInsurancePcv: null,
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

            return fetch(widgetApiUrl + '/widget/avg-apy-info/month', fetchOptions)
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

            return fetch(widgetApiUrl + '/hedge-strategies/' + contractAddress + '/avg-apy-info/month', fetchOptions)
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

        async getInsurancePcvData(widgetApiUrl) {
            let fetchOptions = {
                headers: {
                    "Access-Control-Allow-Origin": widgetApiUrl
                }
            };

            return fetch(widgetApiUrl + '/insurance/', fetchOptions)
                .then(value => value.json())
                .then(value => {
                    if (value && value.lastTvl) {
                        let result = 0;

                        if (value.lastTvl < 0.0001) {
                            if (value.payouts && value.payouts.length > 0) {
                                result = value.payouts[value.payouts.length - 1].tvl;
                            }
                        } else {
                            result = value.lastTvl;
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
            [this.polygonApy, this.avaxApy, this.bscApy, this.opApy, this.etsApyPolygon, this.etsApyBscBusd, this.etsApyRuby, this.etsApyNightOvAr, this.etsApyAlpha, this.etsApyBeta, this.etsApyGamma, this.etsApyDelta, this.etsApyEpsilon, this.etsApyZeta, this.etsApyEta, this.etsApyUniAlpha, this.etsApyUniBeta] = await Promise.all([
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_AVAX),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_BSC),
                this.getApyData(process.env.VUE_APP_WIDGET_API_URL_OP),

                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf'),

                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xc6eca7a3b863d720393DFc62494B6eaB22567D37'),

                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xA88F8c02eBdE678de623C6BCFC886De82e18ad00'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x5651D18F066E132bB90102954429aef1a636576F'),

                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4B6a705A26178f4693428526e86a48659dA44433'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x025656862635b670FC62e4BaD9D20744258dbb02'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xdD7e3823d9178CEFBB486b1c56Fd31EE7DcfF323'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4279474D4643269613ff1832ff9aD88077b4E67F'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xB53Ecca6CB52279e80D8d05a0BC741aDf50bfce4'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x0Eec93122d95198baeC9EF16f1BE4f567ea2a750'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x5b72258392d31d6B64e16e885758dcf2910895E4'),

                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xfDe73D45Cb60D14E1Deb3b1bb9b67D9B7E9cc873'),
                this.getEtsApyData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x0FE7AA84db235656D09C5B41deAd2784EFda3b39'),
            ]);
        },

        async updatePcvData() {
            [this.polygonPcv, this.avaxPcv, this.bscPcv, this.opPcv, this.etsPcvPolygon, this.etsPcvBscBusd, this.etsPcvRuby, this.etsPcvNightOvAr, this.etsPcvAlpha, this.etsPcvBeta, this.etsPcvGamma, this.etsPcvDelta, this.etsPcvEpsilon, this.etsPcvZeta, this.polygonInsurancePcv, this.etsPcvEta, this.etsPcvUniAlpha, this.etsPcvUniBeta] = await Promise.all([
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_AVAX),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC),
                this.getPcvData(process.env.VUE_APP_WIDGET_API_URL_OP),

                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4b5e0af6AE8Ef52c304CD55f546342ca0d3050bf'),

                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_BSC, '0xc6eca7a3b863d720393DFc62494B6eaB22567D37'),

                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xA88F8c02eBdE678de623C6BCFC886De82e18ad00'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x5651D18F066E132bB90102954429aef1a636576F'),

                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4B6a705A26178f4693428526e86a48659dA44433'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x025656862635b670FC62e4BaD9D20744258dbb02'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xdD7e3823d9178CEFBB486b1c56Fd31EE7DcfF323'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x4279474D4643269613ff1832ff9aD88077b4E67F'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0xB53Ecca6CB52279e80D8d05a0BC741aDf50bfce4'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x0Eec93122d95198baeC9EF16f1BE4f567ea2a750'),

                this.getInsurancePcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON),

                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_POLYGON, '0x5b72258392d31d6B64e16e885758dcf2910895E4'),

                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0xfDe73D45Cb60D14E1Deb3b1bb9b67D9B7E9cc873'),
                this.getEtsPcvData(process.env.VUE_APP_WIDGET_API_URL_OP, '0x0FE7AA84db235656D09C5B41deAd2784EFda3b39'),
            ]);
        },

        async getMainCardsData() {
            await Promise.all([
                this.updateApyData(),
                this.updatePcvData(),
            ]);

            let etsChainDict = {
                // 'polygon': this.etsApyPolygon,
                // 'bscBusd': this.etsApyBscBusd,
                'ruby': this.etsApyRuby,
                'night_ov_ar': this.etsApyNightOvAr,

                'qs_alpha_wmatic_usdc': this.etsApyAlpha,
                'qs_beta_wmatic_usdc': this.etsApyBeta,
                'qs_gamma_weth_usdc': this.etsApyGamma,
                'qs_delta_weth_usdc': this.etsApyDelta,
                'qs_epsilon_weth_dai': this.etsApyEpsilon,
                'qs_zeta_wbtc_usdc': this.etsApyZeta,
                'uni_eta_wmatic_usdc': this.etsApyEta,

                'uni_alpha_weth_usdc': this.etsApyUniAlpha,
                'uni_beta_weth_dai': this.etsApyUniBeta,
            };

            for(const [key, value] of Object.entries(etsChainDict)) {
                if (!this.apyWeekEts || value > this.apyWeekEts) {
                    this.apyWeekEts = value;
                    this.bestEtsName = key;

                    if (key === 'polygon' || key === 'polygonUsdc' || key === 'qs_alpha_wmatic_usdc' || key === 'qs_beta_wmatic_usdc' || key === 'qs_delta_weth_usdc' || key === 'qs_gamma_weth_usdc' || key === 'qs_epsilon_weth_dai' || key === 'qs_zeta_wbtc_usdc' || key === 'uni_eta_wmatic_usdc') {
                        this.bestChainEts = 'polygon';
                    }

                    if (key === 'bscBusd') {
                        this.bestChainEts = 'bsc';
                    }

                    if (key === 'ruby' || key === 'garnet' || key === 'night_ov_ar' || key === 'uni_alpha_weth_usdc' || key === 'uni_beta_weth_dai') {
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
                // 'avax': this.avaxApy,
                'op': this.opApy,
                // 'bsc': this.bscApy,
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
            this.pcv = this.polygonPcv + this.avaxPcv + this.bscPcv + this.opPcv + this.etsPcvPolygon + this.etsPcvBscBusd + this.etsPcvRuby + this.etsPcvNightOvAr + this.etsPcvAlpha + this.etsPcvBeta + this.etsPcvGamma + this.etsPcvDelta + this.etsPcvEpsilon + this.etsPcvZeta + this.polygonInsurancePcv + this.etsPcvEta + this.etsPcvUniAlpha + this.etsPcvUniBeta;

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
