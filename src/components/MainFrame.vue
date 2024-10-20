<template>
    <div>
        <main
            class="shop"
            ref="scrollContainer"
            @mousedown="startDrag"
            @mouseup="stopDrag"
            @mouseleave="stopDrag"
            @mousemove="onDrag"
            @wheel="handleScrollWheel"
        >
            <h1 class="fixed">
                Loja do Fortnite ({{ this.getDateWithAdjustment() }})
            </h1>

            <!-- Se houver itens na loja -->
            <div v-if="Object.keys(shopItems).length" class="categories">
                <div
                    v-for="(items, category) in shopItems"
                    :key="category"
                    class="category"
                >
                    <h2 class="category-name">{{ category }}</h2>
                    <div class="items">
                        <!-- Iterar sobre cada item na categoria -->
                        <div
                            v-for="item in items"
                            :key="item.offerId"
                            :class="{ item: true, pack: isPack(item) }"
                            :style="{
                                backgroundImage: `linear-gradient(to bottom, #${
                                    item?.colors?.color1 || 'ffffff'
                                }, #${item?.colors?.color3 || 'ffffff'}, #${
                                    item?.colors?.color2 || 'ffffff'
                                })`,
                            }"
                        >
                            <!-- Imagem do item -->
                            <img
                                v-if="
                                    (item?.newDisplayAsset &&
                                        item?.newDisplayAsset?.renderImages &&
                                        item?.newDisplayAsset?.renderImages
                                            .length > 0) ||
                                    (item?.tracks && item?.tracks.length > 0)
                                "
                                :src="
                                    item?.newDisplayAsset?.renderImages[0]
                                        ?.image || item?.tracks[0].albumArt
                                "
                                :alt="getItemName(item)"
                                :class="{
                                    image: true,
                                    'image-pack': isPack(item),
                                }"
                                draggable="false"
                            />

                            <!-- Informações do item -->
                            <div class="texts">
                                <h3>{{ getItemName(item) }}</h3>
                                <p class="description">
                                    {{ getItemDescription(item) }}
                                </p>
                                <span class="price">
                                    <img
                                        :src="vbucks"
                                        alt="v-bucks icon"
                                        class="vbucks"
                                        draggable="false"
                                    />
                                    <p>
                                        {{
                                            item.finalPrice.toLocaleString(
                                                "pt-BR"
                                            )
                                        }}
                                        V-Bucks
                                        <span>
                                            {{
                                                item.regularPrice !==
                                                item.finalPrice
                                                    ? item.regularPrice.toLocaleString(
                                                          "pt-BR"
                                                      )
                                                    : ""
                                            }}
                                        </span>
                                    </p>
                                </span>
                            </div>
                            <div class="background-effect"></div>
                        </div>
                    </div>
                </div>
            </div>

            <div v-else>
                <p>Carregando itens da loja...</p>
            </div>

            <!-- Controle de rolagem -->
            <input
                type="range"
                class="fixed custom-range"
                ref="scrollbar"
                :value="scrollValue"
                :max="maxScrollValue"
                @input="handleScroll"
                @mousedown.stop="startSliderDrag"
                @mouseup.stop="stopSliderDrag"
                @mouseleave.stop="stopSliderDrag"
                @mousemove.stop="onSliderDrag"
                @wheel="handleScrollWheel"
            />
        </main>
    </div>
</template>

<script>
import axios from "axios";
import vbuck from "@/assets/icon.webp";

export default {
    name: "MainFrame",
    data() {
        return {
            shopItems: {},
            vbucks: vbuck,
            isDragging: false,
            startX: 0,
            scrollLeft: 0,
            scrollValue: 0,
            maxScrollValue: 0,
            isSliderDragging: false,
        };
    },
    async created() {
        try {
            const response = await axios.get(
                "https://fortnite-api.com/v2/shop?language=pt-BR"
            );

            if (
                response.data &&
                response.data.data &&
                response.data.data.entries
            ) {
                this.shopItems = response.data.data.entries;
                this.shopItems = this.groupByCategory(this.shopItems);
                delete this.shopItems["Músicas"];
                this.definirPrioridadeCategoria(this.shopItems);

                // Converte o objeto em um array de entradas
                let entries = Object.entries(this.shopItems);

                // 1. Ordenar os itens pela prioridade
                const sortedEntries = entries.sort(([, itemA], [, itemB]) => {
                    const prioridadeA =
                        itemA.categoriaPrioridade !== undefined
                            ? itemA.categoriaPrioridade
                            : -Infinity;

                    const prioridadeB =
                        itemB.categoriaPrioridade !== undefined
                            ? itemB.categoriaPrioridade
                            : -Infinity;

                    return prioridadeB - prioridadeA; // Ordena do maior para o menor
                });

                // 2. Criar um novo array e mover categorias com dois itens ou menos para o final
                const categoriaCount = {};

                // Contar itens em cada categoria
                sortedEntries.forEach(([categoria, item]) => {
                    categoriaCount[categoria] = item.length; // Contando a quantidade de itens em cada categoria
                });

                // Separar os itens em dois grupos
                const itemsWithFew = [];
                const itemsWithMore = [];

                // Separar os itens em grupos
                sortedEntries.forEach(([categoria, item]) => {
                    const count = categoriaCount[categoria] || 0;

                    if (count <= 2) {
                        itemsWithFew.push([categoria, item]); // Adiciona ao grupo com dois ou menos itens
                    } else {
                        itemsWithMore.push([categoria, item]); // Adiciona ao grupo com mais de dois itens
                    }
                });

                // Combinar os grupos, colocando os com mais de dois itens primeiro
                const finalSortedEntries = [...itemsWithMore, ...itemsWithFew];

                // Convertendo de volta para objeto
                this.shopItems = Object.fromEntries(finalSortedEntries);
            } else {
                throw new Error("Erro");
            }
            this.$nextTick(() => {
                this.updateMaxScrollValue();
                this.scrollValue = this.$refs.scrollContainer.scrollLeft;
            });
        } catch (error) {
            throw new Error("Erro");
        }
    },
    methods: {
        definirPrioridadeCategoria(data) {
            for (const categoria in data) {
                const itens = data[categoria];

                // Extrai as prioridades e encontra a prioridade máxima
                const prioridades = itens.map((item) => item.sortPriority);
                let current = 0;

                for (const p of prioridades) {
                    current += p;
                }

                // Adiciona a prioridade da categoria ao objeto
                data[categoria].categoriaPrioridade = current;
            }
        },
        groupByCategory(array) {
            const acumulador = array.reduce((acc, item) => {
                // Verifica se o item tem brItems ou está solto no array
                const brItem =
                    item.brItems && item.brItems.length > 0
                        ? item.brItems[0]
                        : item;

                // Verifica se o item tem 'set', 'series' ou se está solto com 'layout' ou 'instruments'
                const set = brItem.set ? brItem.set.value : null;
                const series = brItem.series ? brItem.series.value : null;
                const layout = item.layout ? item.layout.name : null;
                const instruments =
                    item.instruments && item.instruments.length > 0
                        ? item.instruments[0].name
                        : null;
                const tracks =
                    item.tracks && item.tracks.length > 0
                        ? item.tracks[0].name
                        : null;

                // Define a chave de categorização com 'set', 'series', 'layout', 'instruments' ou "Sem Categoria"
                const chave =
                    set ||
                    series ||
                    layout ||
                    instruments ||
                    tracks ||
                    "Sem Categoria";

                // Adiciona o item à categoria correspondente
                if (!acc[chave]) {
                    acc[chave] = [];
                }
                acc[chave].push(item);

                return acc;
            }, {});

            // Ordena os itens dentro de cada categoria
            const categoriasOrdenadas = Object.entries(acumulador).reduce(
                (obj, [key, items]) => {
                    const itensOrganizados = items.sort(
                        (a, b) => b.finalPrice - a.finalPrice
                    );
                    obj[key] = itensOrganizados;
                    return obj;
                },
                {}
            );

            return categoriasOrdenadas;
        },
        startDrag(event) {
            this.isDragging = true;
            this.startX = event.pageX - this.$refs.scrollContainer.offsetLeft;
            this.scrollLeft = this.$refs.scrollContainer.scrollLeft;
            this.$refs.scrollContainer.style.cursor = "grabbing"; // Muda o cursor para 'grabbing' ao arrastar
        },
        stopDrag() {
            this.isDragging = false;
            this.$refs.scrollContainer.style.cursor = "grab"; // Retorna o cursor para 'grab' quando solto
        },
        onDrag(event) {
            if (!this.isDragging) return; // Não faz nada se não estiver arrastando
            event.preventDefault(); // Evita seleção de texto durante o arrasto
            const x = event.pageX - this.$refs.scrollContainer.offsetLeft; // Acessa a referência corretamente
            const walk = (x - this.startX) * 2.5; // Multiplica para aumentar a sensibilidade
            this.$refs.scrollContainer.scrollLeft = this.scrollLeft - walk; // Move a rolagem
            this.updateScrollValue(); // Atualiza o valor do range enquanto arrasta
        },
        handleScroll(event) {
            const scrollContainer = this.$refs.scrollContainer;
            const scrollPosition = Number(event.target.value); // Posição do scroll a partir do range
            scrollContainer.scrollLeft = scrollPosition; // Define a posição do scroll do container
            this.updateScrollValue(); // Atualiza o valor do range baseado na rolagem
        },
        updateScrollValue() {
            const scrollContainer = this.$refs.scrollContainer;
            this.scrollValue = scrollContainer.scrollLeft; // Atualiza o valor do range baseado na rolagem atual
        },
        updateMaxScrollValue() {
            const scrollContainer = this.$refs.scrollContainer;
            this.maxScrollValue =
                scrollContainer.scrollWidth - scrollContainer.clientWidth; // Atualiza o valor máximo
        },
        startSliderDrag(event) {
            this.isSliderDragging = true; // Inicia o arrasto do slider
            event.preventDefault(); // Previne o comportamento padrão
            this.$refs.scrollContainer.style.cursor = "grabbing";
        },
        stopSliderDrag() {
            this.isSliderDragging = false; // Para o arrasto do slider
            this.$refs.scrollContainer.style.cursor = "grab";
        },
        onSliderDrag(event) {
            if (!this.isSliderDragging) return; // Não faz nada se não estiver arrastando
            const rect = this.$refs.scrollbar.getBoundingClientRect(); // Obtém a posição do slider
            const x = event.clientX - rect.left; // Calcula a posição do mouse em relação ao slider
            const percentage = x / rect.width; // Calcula a porcentagem da posição do mouse
            const newValue = percentage * this.maxScrollValue; // Calcula o novo valor
            this.scrollValue = Math.min(
                Math.max(newValue, 0),
                this.maxScrollValue
            ); // Atualiza o valor do slider
            this.handleScroll({ target: { value: this.scrollValue } }); // Atualiza a rolagem do container
        },
        handleScrollWheel(event) {
            event.preventDefault(); // Previne o comportamento padrão de rolagem da página
            const scrollContainer = this.$refs.scrollContainer;
            const delta = event.deltaY; // Pega a quantidade de movimento da roda do mouse
            // Ajusta a posição do scroll com base na movimentação da roda do mouse
            scrollContainer.scrollLeft += delta;
            // Atualiza o valor do range após a rolagem
            this.updateScrollValue();
        },
        // Função para pegar o nome do item
        getItemName(item) {
            return (
                item?.bundle?.name ||
                (item?.brItems && item.brItems.length > 0
                    ? item.brItems[0].name
                    : null) ||
                (item?.instruments && item.instruments.length > 0
                    ? item.instruments[0].name
                    : null) ||
                (item?.tracks && item.tracks.length > 0
                    ? item.tracks[0].title
                    : null) ||
                (item?.cars && item.cars.length > 0
                    ? item.cars[0].name
                    : null) ||
                (item?.legoKits && item.legoKits.length > 0
                    ? item.legoKits[0].name
                    : null) ||
                "Nome Indisponível"
            );
        },

        // Função para pegar a descrição do item
        getItemDescription(item) {
            return (
                (item?.brItems && item.brItems.length > 0
                    ? item.brItems[0].description
                    : null) ||
                (item?.instruments && item.instruments.length > 0
                    ? item.instruments[0].description
                    : null) ||
                (item?.tracks && item.tracks.length > 0
                    ? item.tracks[0].artist +
                      " (" +
                      item.tracks[0].releaseYear +
                      ")"
                    : null) ||
                (item?.cars && item.cars.length > 0
                    ? item.cars[0].description
                    : null) ||
                ""
            );
        },
        getDateWithAdjustment() {
            // Cria uma nova data com a data atual
            const currentDate = new Date();

            // Verifica se a hora atual é igual ou superior a 21:00
            if (currentDate.getHours() > 21) {
                currentDate.setDate(currentDate.getDate() - 1);
            }

            // Obtém o dia, mês e ano
            const day = String(currentDate.getDate()).padStart(2, "0"); // Formata o dia com dois dígitos
            const month = String(currentDate.getMonth() + 1).padStart(2, "0"); // O mês é baseado em zero
            const year = currentDate.getFullYear(); // Obtém o ano completo

            // Retorna a data formatada
            return `${day}/${month}/${year}`;
        },
    },
    computed: {
        isPack() {
            return (item) => !!item?.bundle?.name;
        },
        isItem() {
            return true;
        },
    },
    watch: {
        scrollValue: {
            handler(newValue) {
                this.$refs.scrollContainer.scrollLeft = Number(newValue); // Atualiza a posição do scroll ao mudar o valor
            },
            immediate: false,
        },
    },
};
</script>


<style scoped>
main.shop {
    position: relative;
    height: 100vh;
    width: 100%;
    overflow-x: hidden;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: start;
    -ms-flex-align: start;
    align-items: flex-start;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -ms-flex-direction: column;
    flex-direction: column;
    padding: 1rem;
    cursor: -webkit-grab;
    cursor: grab;
}

main.shop h1 {
    font-size: 2.4rem;
    color: var(--btn-blurple);
}

main.shop div.categories {
    display: -webkit-inline-box;
    display: -ms-inline-flexbox;
    display: inline-flex;
    height: auto;
    overflow-x: hidden;
}

main.shop div.categories div.category {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -ms-flex-direction: column;
    flex-direction: column;
    -webkit-box-pack: start;
    -ms-flex-pack: start;
    justify-content: flex-start;
    -webkit-box-align: start;
    -ms-flex-align: start;
    align-items: flex-start;
    height: 500px;
    width: auto;
    margin: 1rem;
    overflow-y: auto;
    white-space: normal;
}

main.shop div.categories div.category .category-name {
    margin-bottom: 0.5rem;
}

main.shop div.categories div.category div.items {
    height: 90%;
    width: 100%;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: start;
    -ms-flex-align: start;
    align-items: flex-start;
    -ms-flex-pack: distribute;
    justify-content: space-around;
}

main.shop div.categories div.category div.items div.item {
    position: relative;
    height: 100%;
    width: 280px;
    text-align: left;
    margin: 0 1rem;
    border-radius: 0.5rem;

    display: -webkit-box;

    display: -ms-flexbox;

    display: flex;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -ms-flex-direction: column;
    flex-direction: column;
    -webkit-box-pack: end;
    -ms-flex-pack: end;
    justify-content: flex-end;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    overflow: hidden;
    -webkit-transition: all 0.4s ease;
    -o-transition: all 0.4s ease;
    transition: all 0.4s ease;
}

main.shop div.categories div.category div.items div.item:hover {
    -webkit-transform: scale(1.025);
    -ms-transform: scale(1.025);
    transform: scale(1.025);
}

main.shop div.categories div.category div.items div.item:first-of-type {
    margin-left: 0;
}

main.shop div.categories div.category div.items div.item div.background-effect {
    position: absolute;
    height: 100%;
    width: 100%;
    z-index: 1;
    background-image: -webkit-gradient(
        linear,
        left top,
        left bottom,
        color-stop(65%, #00000000),
        to(#000000)
    );
    background-image: -o-linear-gradient(top, #00000000 65%, #000000);
    background-image: linear-gradient(to bottom, #00000000 65%, #000000);
}

main.shop div.categories div.category div.items div.item.pack {
    width: calc(280px * 2.5);
    -webkit-box-orient: horizontal;
    -webkit-box-direction: reverse;
    -ms-flex-direction: row-reverse;
    flex-direction: row-reverse;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: end;
    -ms-flex-align: end;
    align-items: flex-end;
    overflow: hidden;
}

main.shop div.categories div.category div.items div.item img.image {
    height: 90%;
    width: auto;
}

main.shop div.categories div.category div.items div.item img.image.image-pack {
    position: absolute;
    top: 0;
    height: 160%;
    width: auto;
}

main.shop div.categories div.category div.items div.item div.texts {
    position: absolute;
    bottom: 0;
    left: 0;
    padding: 1rem;
    z-index: 2;

    display: -webkit-box;

    display: -ms-flexbox;

    display: flex;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -ms-flex-direction: column;
    flex-direction: column;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: start;
    -ms-flex-align: start;
    align-items: flex-start;
}

main.shop div.categories div.category div.items div.item div.texts > * {
    font-family: "Montserrat";
}

main.shop div.categories div.category div.items div.item div.texts h3 {
    font-size: 1.8rem;
    font-weight: 800;
}

main.shop
    div.categories
    div.category
    div.items
    div.item
    div.texts
    p.description {
    font-size: 1rem;
    font-weight: 600;
}

main.shop div.categories div.category div.items div.item div.texts span.price {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    gap: 0.5rem;
    margin-top: 0.5rem;
}

main.shop div.categories div.category div.items div.item div.texts span.price p,
main.shop
    div.categories
    div.category
    div.items
    div.item
    div.texts
    span.price
    p
    span {
    font-family: "Montserrat";
    font-weight: 800;
}

main.shop
    div.categories
    div.category
    div.items
    div.item
    div.texts
    span.price
    p
    span {
    color: var(--lightgrey);
    text-decoration: line-through;
}

main.shop
    div.categories
    div.category
    div.items
    div.item
    div.texts
    span.price
    img.vbucks {
    width: 24px;
    height: auto;
}

/* Drag and Drop */
.fixed {
    position: fixed;
    left: 50%;
    -webkit-transform: translateX(-50%);
    -ms-transform: translateX(-50%);
    transform: translateX(-50%);
}

h1.fixed {
    top: 3rem;
}

input.fixed {
    width: 90%;
    bottom: 3rem;
}

@media screen and (max-width: 768px) {
    main.shop {
        width: 100%;
        height: 100%;
        justify-items: center;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
        margin: 2rem 0 4rem 0;
        text-align: center;
    }

    main.shop div.categories {
        width: 100%;
        height: auto;
        -webkit-box-orient: vertical;
        -webkit-box-direction: normal;
        -ms-flex-direction: column;
        flex-direction: column;
        -webkit-box-pack: center;
        -ms-flex-pack: center;
        justify-content: center;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
        margin-top: 2rem;
    }

    main.shop div.categories div.category {
        width: 100%;
        height: auto;
        -webkit-box-orient: vertical;
        -webkit-box-direction: normal;
        -ms-flex-direction: column;
        flex-direction: column;
        -webkit-box-pack: center;
        -ms-flex-pack: center;
        justify-content: center;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
        padding: 0;
        margin: 4rem 0 0 0;
    }

    main.shop div.categories div.category .category-name {
        display: block;
        width: 90%;
        text-align: center;
        margin-bottom: 0.5rem;
    }

    main.shop div.categories div.category:first-of-type {
        margin-top: 0;
    }

    main.shop div.categories div.category div.items {
        height: 100%;
        width: 100%;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
        -webkit-box-pack: center;
        -ms-flex-pack: center;
        justify-content: center;
        -webkit-box-orient: vertical;
        -webkit-box-direction: normal;
        -ms-flex-direction: column;
        flex-direction: column;
        gap: 2rem;
    }

    main.shop div.categories div.category div.items div.item {
        position: relative;
        height: 320px;
        width: 90% !important;
        max-width: 100%;
        margin: 0;
    }

    main.shop div.categories div.category div.items div.item img.image {
        width: 90% !important;
        height: auto !important;
    }

    .fixed {
        position: static;
        left: auto;
        -webkit-transform: none;
        -ms-transform: none;
        transform: none;
    }

    input {
        display: none;
    }
}
</style>
