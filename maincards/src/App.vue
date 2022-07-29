<template>
    <div id="app-maincards">
        <MainCards :network="network"/>
    </div>
</template>

<script>


import MainCards from "./components/MainCards";
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
        MainCards,
    }
}
</script>

<style>

/* mobile */
@media only screen and (max-width: 1400px) {
}

@media only screen and (min-width: 1400px) {
}

#app-maincards {
    font-family: Lato, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    background-color: transparent;

    width: 100%;
}

</style>
