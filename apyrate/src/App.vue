<template>
    <div id="app-apyrate">
        <ApyRate :network="network"/>
    </div>
</template>

<script>


import ApyRate from "./components/ApyRate";
export default {
    name: 'App',

    props: {
        width: {
            type: String,
        },

        network: {
            type: String,
            default: 'polygon'
        }
    },

    components: {
        ApyRate,
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
    #app-apyrate {
        width: 88vw !important;
    }
}

@media only screen and (min-width: 961px) {
    #app-apyrate {
        width: 43vw !important;
    }
}

#app-apyrate {
    font-family: Lato, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    background-color: transparent;
}

</style>
