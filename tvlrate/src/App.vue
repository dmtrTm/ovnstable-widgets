<template>
    <div id="app-tvlrate">
        <TvlRate :network="network" :product="product" :address="address"/>
    </div>
</template>

<script>


import TvlRate from "./components/TvlRate";
export default {
    name: 'App',

    props: {
        width: {
            type: String,
        },

        network: {
            type: String,
            default: 'polygon'
        },

        product: {
            type: String,
            default: 'usd+'
        },

        address: {
            type: String,
            default: ''
        }
    },

    components: {
        TvlRate,
    }
}
</script>

<style>

/* mobile */
@media only screen and (max-width: 1400px) {
}

@media only screen and (min-width: 1400px) {
}

@media only screen and (max-width: 960px) {
    #app-tvlrate {
        width: 88vw !important;
    }
}

@media only screen and (min-width: 961px) {
    #app-tvlrate {
        width: 43vw !important;
    }
}

#app-tvlrate {
    font-family: Lato, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    background-color: transparent;
}

</style>
