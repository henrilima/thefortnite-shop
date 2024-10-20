<template>
    <FortniteShop />
</template>

<script>
import FortniteShop from "./components/MainFrame.vue";

export default {
    name: "App",
    components: {
        FortniteShop,
    },
};
</script>

<style>
@import url(https://fonts.googleapis.com/css?family=Montserrat:100,200,300,regular,500,600,700,800,900,100italic,200italic,300italic,italic,500italic,600italic,700italic,800italic,900italic);

@font-face {
    font-family: "Fortnite";
    src: url("@/assets/fonts/BurbankBigRegular-Black.otf");
}

* {
    margin: 0;
    padding: 0;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    font-family: "Fortnite", sans-serif;
    font-weight: 500;
    text-decoration: none;
    list-style: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

html {
    scroll-behavior: smooth;
}

a {
    color: var(--white);
}

p {
    display: inline-block;
}

:root {
    --black: #101010;
    --white: #f2f2f2;
    --blurple: #584cda;
    --orange: #feb029;
    --lightgrey: #8f8f8f;
    --grey: #606060;
    --semidarkgrey: #282828;
    --darkgrey: #181818;

    --btn-blurple: #7168d6;
    --btn-orange: #daa240;
}

body {
    background-color: var(--black);
    color: var(--white);
}

.custom-range {
    -webkit-appearance: none; /* Remove o estilo padrão do slider no Safari */
    -moz-appearance: none;
    appearance: none; /* Remove o estilo padrão do slider em outros navegadores */
    height: 5px !important;
    padding: 0.5rem 0;
    background: var(--semidarkgrey); /* Cor da trilha do slider */
    border-radius: 9px; /* Arredonda as bordas da trilha */
    outline: none; /* Remove a borda ao focar */
    opacity: 0.2; /* Opacidade do slider */
    -webkit-transition: opacity 0.2s;
    -o-transition: opacity 0.2s;
    transition: opacity 0.2s; /* Transição suave ao passar o mouse */
}

.custom-range:hover {
    opacity: 0.8; /* Aumenta a opacidade ao passar o mouse */
}

.custom-range::-webkit-slider-thumb {
    -webkit-appearance: none; /* Remove o estilo padrão do thumb no Safari */
    appearance: none; /* Remove o estilo padrão do thumb em outros navegadores */
    width: 14px; /* Largura do thumb */
    height: 14px; /* Altura do thumb */
    background: var(--blurple); /* Cor do thumb */
    border-radius: 50%; /* Arredonda o thumb */
    cursor: pointer; /* Muda o cursor ao passar o mouse */
}

.custom-range::-moz-range-thumb {
    width: 14px; /* Largura do thumb */
    height: 14px; /* Altura do thumb */
    background: var(--blurple); /* Cor do thumb */
    border-radius: 50%; /* Arredonda o thumb */
    cursor: pointer; /* Muda o cursor ao passar o mouse */
}

.custom-range::-ms-thumb {
    width: 14px; /* Largura do thumb */
    height: 14px; /* Altura do thumb */
    background: var(--blurple); /* Cor do thumb */
    border-radius: 50%; /* Arredonda o thumb */
    cursor: pointer; /* Muda o cursor ao passar o mouse */
}
</style>
