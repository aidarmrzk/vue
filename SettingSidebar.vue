<template>
    <nav
        class="bg-white sidebar"
        :class="{ hidden: isSidebarHidden, image: !isSettingSidebar, 'not-verified': !isVerified }"
    >
        <!-- <button class="toggle-button" @click="toggleSidebar">
            Скрыть/Показать сайдбар
        </button> -->
        <div class="scroll-container">
            <template v-if="isSettingSidebar">
                <div class="pt-3 sidebar-sticky">
                    <button
                        type="button"
                        class="btn btn-success w-100 mb-3"
                        @click="newChat"
                    >
                        Новый чат
                    </button>
                </div>
                <div>
                    <button
                        type="button"
                        v-for="(chat, index) in reversedChats"
                        :key="index"
                        @click="
                            handleSelectChat(chat);
                            $emit('selectChat', chat);
                        "
                        :class="{
                            'btn btn-light w-100 mb-1 d-flex justify-content-between': true,
                            selected: chat === selectedChat,
                        }"
                    >
                        <span class="content-title">{{
                            chat.dialogs[0].content
                        }}</span>
                        <i
                            v-if="chat === selectedChat"
                            class="bi bi-trash3 ml-2"
                            @click="deleteChat(chat)"
                        ></i>
                    </button>
                </div>
            </template>
            <template v-else>
                <div class="setting-item">
                    <label for="exampleInputEmail1" class="form-label"
                        >Генеративная модель</label
                    >
                    <select
                        v-model="selectedModel"
                        class="form-select"
                        aria-label="Default select example"
                    >
                        <option value="stable-diffusion-v1-5">
                            Stable Diffusion 1.5
                        </option>
                        <option value="stable-diffusion-xl-beta-v2-2-2">
                            Stable Diffusion XL 0.8
                        </option>
                        <option value="stable-diffusion-xl-1024-v1-0">
                            Stable Diffusion XL 1.0
                        </option>
                    </select>
                </div>
                <div class="setting-item">
                    <label for="sizeSelect" class="form-label"
                        >Размер изображения</label
                    >
                    <select
                        v-model="selectedImageSize"
                        id="sizeSelect"
                        class="form-select"
                        aria-label="Default select example"
                    >
                        <option
                            v-for="size in availableSizes"
                            :value="size"
                            :key="size"
                        >
                            {{ size }}
                        </option>
                    </select>
                </div>
                <div class="setting-item">
                    <label for="exampleInputEmail1" class="form-label"
                        >Стиль</label
                    >
                    <select
                        v-model="selectedImageStyle"
                        class="form-select"
                        aria-label="Default select example"
                    >
                        <option value="3d-model">{{ $t("three_d_model") }}</option>
                        <option value="analog-film">{{ $t("analog_film") }}</option>
                        <option value="anime">{{ $t("anime") }}</option>
                        <option value="cinematic">{{ $t("cinematic") }}</option>
                        <option value="comic-book">{{ $t("comic_book") }}</option>
                        <option value="digital-art">{{ $t("digital_art") }}</option>
                        <option value="enhance">{{ $t("enhance") }}</option>
                        <option value="fantasy-art">{{ $t("fantasy_art") }}</option>
                        <option value="isometric">{{ $t("isometric") }}</option>
                        <option value="line-art">{{ $t("line_art") }}</option>
                        <option value="low-poly">{{ $t("low_poly") }}</option>
                        <option value="modeling-compound">{{ $t("modeling_compound") }}</option>
                        <option value="neon-punk">{{ $t("neon_punk") }}</option>
                        <option value="origami">{{ $t("origami") }}</option>
                        <option value="photographic">{{ $t("photographic") }}</option>
                        <option value="pixel-art">{{ $t("pixel_art") }}</option>
                        <option value="tile-texture">{{ $t("tile_texture") }}</option>
                    </select>
                </div>
                <div class="setting-item">
                    <label for="customRange3" class="form-label"
                        >Количество изображений {{ selectedImageCount }}</label
                    >
                    <input
                        v-model="selectedImageCount"
                        type="range"
                        class="form-range"
                        min="1"
                        max="10"
                        step="1"
                        id="customRange3"
                    />
                </div>
                <div class="setting-item">
                    <label for="customRange4" class="form-label"
                        >Количество шагов {{ selectedImageSteps }}</label
                    >
                    <input
                        v-model="selectedImageSteps"
                        type="range"
                        class="form-range"
                        min="10"
                        max="150"
                        step="10"
                        id="customRange4"
                    />
                </div>

                <div class="price">
                    <p>Стоимость генерации</p>
                    <span>{{ priceImage }}</span>
                </div>
                <div class="price">
                    <p>Текущее количество</p>
                    <p>
                        {{
                            user.subscription.tokens_count <= 0
                                ? 0
                                : user.subscription.tokens_count
                        }}
                    </p>
                </div>

                <div class="setting-item button-price">
                    <button
                        type="button"
                        class="btn chad-button"
                        data-bs-toggle="modal"
                        data-bs-target="#exampleModal"
                    >
                        Таблица стоимости
                    </button>
                </div>
            </template>
        </div>
    </nav>
    <!-- Modal Price Image-->
    <div
        class="modal fade"
        id="exampleModal"
        tabindex="-1"
        aria-labelledby="exampleModalLabel"
        aria-hidden="true"
    >
        <div class="modal-dialog modal-dialog-centered modal-lg">
            <div class="modal-content">
                <div class="modal-header">
                    <h1 class="modal-title fs-5" id="exampleModalLabel">
                        <b>Таблица стоимости* изображений</b>
                    </h1>
                    <button
                        type="button"
                        class="btn-close"
                        data-bs-dismiss="modal"
                        aria-label="Close"
                    ></button>
                </div>
                <div class="modal-body">
                    <div class="container">
                        <div class="column">
                            <div
                                class="col"
                                v-for="(model, index) in models"
                                :key="index"
                            >
                                <div class="chad-box">
                                    <h5
                                        class="card-title text-center mb-2 title-stable"
                                    >
                                        {{ model.name }}
                                    </h5>
                                    <div class="table-container">
                                        <table class="mb-2 table">
                                            <thead>
                                                <tr>
                                                    <th>Размер \ Шаг</th>
                                                    <th
                                                        v-for="(step, stepIndex) in model.screens[0].stepsPrice"
                                                        :key="stepIndex"
                                                    >
                                                        {{ stepIndex }}
                                                    </th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                <tr
                                                    v-for="(
                                                        screen, screenIndex
                                                    ) in model.screens"
                                                    :key="screenIndex"
                                                    :class="{
                                                        'border-none':
                                                            screenIndex ===
                                                            model.screens.length -
                                                                1,
                                                    }"
                                                >
                                                    <td>{{ screen.size }}</td>
                                                    <td
                                                        v-for="(price, priceIndex) in screen.stepsPrice"
                                                        :key="priceIndex"
                                                    >
                                                        {{
                                                            parseFloat((price + 0.35)
                                                                .toFixed(2)
                                                            ).toString()
                                                        }}
                                                    </td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                            <span>* Стоимость указана в атомах</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, inject, computed, onMounted, watch } from "vue";
import { useStore, mapGetters } from "vuex";
import { useRoute, useRouter  } from "vue-router";
onMounted(() => {
    updatePriceImage();
})
const store = useStore();
const route = useRoute();
const router = useRouter();
const isSettingSidebar = ref(true);
const selectedChat = ref(null);
let selectedImageStyle = ref("enhance");
let selectedImageCount = ref(1);
let selectedImageSteps = ref(50);
let selectedModel = ref("stable-diffusion-v1-5");
let selectedImageSize = ref("512х512");

let chats = computed(() => store.getters["auth/getUserChats"]);
let reversedChats = computed(() => chats.value.slice().reverse());

router.beforeEach((to, from, next) => {
    selectedModel.value = "stable-diffusion-v1-5"
    next();
});

const availableSizes = computed(() => {
    if (selectedModel.value === "stable-diffusion-v1-5") {
        return ["768х512", "512х768", "512х512"];
    } else if (selectedModel.value === "stable-diffusion-xl-beta-v2-2-2") {
        return ["768х512", "512х768", "512х512"];
    } else if (selectedModel.value === "stable-diffusion-xl-1024-v1-0") {
        return ["1024х1024", "1152х896", "896х1152", "1216х832", "832х1216", "1344х768", "1536х640"];
    } else {
        return [];
    }
});

const isSidebarHidden = ref(false);

const toggleSidebar = () => {
    isSidebarHidden.value = !isSidebarHidden.value;
};

const newChat = () => {
    store.commit("auth/CHANGE_CHAT_ID", null);
    selectedChat.value = null;
};

const handleSelectChat = (chat) => {
    selectedChat.value = chat;
};

const deleteChat = (chat) => {
    store.dispatch("auth/deleteChat", chat.id);
};

watch(
    route,
    (to) => {
        if (to.matched.length) {
            isSettingSidebar.value = to.matched[1].path === "/admin/chat";
            store.commit("setting/SET_DEFAULT_SETTINGS");
            selectedImageSize.value = availableSizes.value[0];
            selectedModel.value = "stable-diffusion-v1-5"
            selectedImageSize.value = "512х512";
            selectedImageStyle.value = "cinematic";
            selectedImageCount.value = 1;
            selectedImageSteps.value = 50;
        }
    },
    { immediate: true }
);

watch(selectedImageSize, async (newSize) => {
    store.commit("setting/SET_IMAGE_SIZE", newSize);
    updatePriceImage();
});

watch(selectedImageStyle, async (newStyle) => {
    store.commit("setting/SET_IMAGE_STYLE", newStyle);
});

watch(selectedImageCount, async (count) => {
    store.commit("setting/SET_IMAGE_COUNT", count);
    updatePriceImage();
});

watch(selectedImageSteps, async (steps) => {
    store.commit("setting/SET_IMAGE_STEPS", steps);
    updatePriceImage();
});

watch(selectedModel, async (model) => {
    store.commit("setting/SET_IMAGE_MODEL", model);
    updatePriceImage();
});

watch(selectedModel, () => {
    selectedImageSize.value = availableSizes.value[0];
});

const models = [
    {
        name: "Stable Diffusion XL 1.0",
        simpleName: "stable-diffusion-xl-1024-v1-0",
        screens: [
            {
                size: "Любой",
                allSizes: ["1024х1024", "1152х896", "896х1152", "1216х832", "832х1216", "1344х768", "1536х640"],
                stepsPrice: {
                            10: 6.75,
                            20: 7.5,
                            30: 8,
                            40: 9,
                            50: 10,
                            60: 11.05,
                            70: 12.05,
                            80: 13.15,
                            90: 14.35,
                            100: 15.55,
                            110: 16.85,
                            120: 18.2,
                            130: 19.6,
                            140: 21.1,
                            150: 22.65
                        }
            },
        ],
    },
    {
        name: "Stable Diffusion XL 0.8",
        simpleName: "stable-diffusion-xl-beta-v2-2-2",
        screens: [
            {
                size: "512х512",
                allSizes: ["512х512"],
                stepsPrice: {
                            10: 0.85,
                            20: 1.65,
                            30: 2.5,
                            40: 3.35,
                            50: 4.15,
                            60: 5,
                            70: 5.85,
                            80: 6.65,
                            90: 7.5,
                            100: 8.35,
                            110: 9.15,
                            120: 10,
                            130: 10.85,
                            140: 11.65,
                            150: 12.5
                        }
            },
            {
                size: "512х768, 768х512",
                allSizes: ["512х768", "768х512"],
                stepsPrice: {
                            10: 1.4,
                            20: 2.835,
                            30: 4.235,
                            40: 5.635,
                            50: 7.035,
                            60: 8.47,
                            70: 9.87,
                            80: 11.27,
                            90: 12.67,
                            100: 14.105,
                            110: 15.505,
                            120: 16.905,
                            130: 18.305,
                            140: 19.74,
                            150: 21.14
                        }
            },
        ],
    },

    {
        name: "Stable Diffusion 1.5",
        simpleName: "stable-diffusion-v1-5",
        screens: [
            {
                size: "512х512",
                allSizes: ["512х512"],
                stepsPrice: {
                            10: 0.338333,
                            20: 0.628333,
                            30: 0.966667,
                            40: 1.305,
                            50: 1.595,
                            60: 1.933333,
                            70: 2.271667,
                            80: 2.561667,
                            90: 2.9,
                            100: 3.238333,
                            110: 3.528333,
                            120: 3.866667,
                            130: 4.205,
                            140: 4.495,
                            150: 4.833333
                        }
            },
            {
                size: "512х768, 768х512",
                allSizes: ["512х768", "768х512"],
                stepsPrice: {
                            10: 0.773333,
                            20: 1.546667,
                            30: 2.32,
                            40: 3.093333,
                            50: 3.915,
                            60: 4.688333,
                            70: 5.461667,
                            80: 6.235,
                            90: 7.008333,
                            100: 7.781667,
                            110: 8.555,
                            120: 9.328333,
                            130: 10.10167,
                            140: 10.875,
                            150: 11.69667
                        }
            },
        ],
    },
];

const isVerified = computed(() => store.getters["auth/getIsVerified"]);
const user = computed(() => store.getters["auth/user"]);

const priceImage = ref('');

const updatePriceImage = () => {
    const imageSettings = computed(() => store.getters["setting/getImageSettings"]);
    const selectedModel = imageSettings.value.model;
    const matchingModel = models.find((model) => model.simpleName === selectedModel);

    if (matchingModel) {
        const selectedSize = imageSettings.value.size
        const matchingSize = matchingModel.screens.find((screen) => screen.allSizes.includes(selectedSize));

        if (matchingSize) {
            const selectedStep = imageSettings.value.steps;
            const matchingStepPrice = matchingSize.stepsPrice[selectedStep];

            if (matchingStepPrice !== undefined) {
                priceImage.value = parseFloat((imageSettings.value.count * (matchingStepPrice + 0.35)).toFixed(2)).toString();
                store.commit("auth/SET_IMAGE_PRICE", priceImage.value);
            }
        }
    }
};
</script>

<style scoped lang="scss">
.toggle-button {
    margin-top: 10px;
    cursor: pointer;
}
.sidebar {
    position: sticky !important;
    width: 250px;
    top: 0;
    height: 100vh;
    padding: 10px;
    box-shadow: 1px 0 10px rgba(0, 0, 0, .1);
    transition: transform 0.3s ease-in-out;
    transform: translateX(0);
    &.not-verified {
            height: calc(100vh - 50px);
            top: 50px;
        }
    &.image {
        background: linear-gradient(90deg, #1e1e25 -30%, #140021 100%);
        width: 300px;
        @media (max-width: 1024px) {
            width: 250px;
        }
        @media (max-width: 900px) {
            display: none;
        }
        .form-label {
            color: #fff;
        }
        select {
            background: url("@/assets/img/sidebar/expand-arrow.png") no-repeat
                right center;
            background-size: 14px 14px;
            background-position: right 10px center;
            background-color: #fff;
            cursor: pointer;

            &:focus {
                border-color: rgb(155, 63, 212);
                box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
            }
            option {
                cursor: pointer;
            }
        }
        input {
            &[type="range"] {
                &::-webkit-slider-thumb {
                    background-color: rgb(155, 63, 212);
                    &:active {
                        border-color: rgb(155, 63, 212);
                        box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
                    }
                    &:focus {
                        border-color: rgb(155, 63, 212);
                        box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
                    }
                }
                &::-moz-range-thumb {
                    background-color: rgb(155, 63, 212);
                    &:active {
                        border-color: rgb(155, 63, 212);
                        box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
                    }
                    &:focus {
                        border-color: rgb(155, 63, 212);
                        box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
                    }
                }
            }
            &:focus {
                &::-webkit-slider-thumb {
                    border-color: rgb(155, 63, 212);
                    box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
                }
                &::-moz-range-thumb {
                    border-color: rgb(155, 63, 212);
                    box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
                }
            }
        }
    }
}
.scroll-container {
    max-height: 97vh;
    overflow-y: auto;
    overflow-x: inherit;
    padding: 10px;
    .price {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin: 0 0 10px;
        p {
            margin: 0;
            line-height: 130%;
            color: #fff;
        }
        span {
            color: #fff;
            background: linear-gradient(to right, #9b3fd4, #aa73b3);
            padding: 4px 7px;
            border-radius: 10px;
        }
    }
}
.scroll-container::-webkit-scrollbar {
    width: 3px; /* толщина ползунка */
}

.scroll-container::-webkit-scrollbar-thumb {
    background-color: #d1d5db; /* цвет ползунка */
}
.sidebar.hidden {
    transform: translateX(90%);
}
.selected {
    background-color: #d3d4d5; /* Define your desired selected color here */
}
.content-title {
    overflow: hidden !important;
    text-overflow: ellipsis;
    white-space: nowrap;
}
.setting-item {
    padding-bottom: 30px;
    &.button-price {
        border-top: 1px solid #d1d5db;
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
        padding: 1rem;
    }
}
.chad-button {
    display: inline-block;
    width: 100%;
    background: linear-gradient(to right, #9b3fd4, #aa73b3);
    box-shadow: 0px 0px 10px 2px rgba(163, 70, 244, 0.3);
    border-radius: 10px;
    font-style: normal;
    font-weight: 400;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.15px;
    color: #fcfcfc;
    font-size: 14px;
    border: none;
    padding: 8px 0;
}
#exampleModal {
    .modal-dialog {
        max-width: none;
        width: max-content;
        @media (max-width: 575px) {
            width: auto;
        }
        .modal-header {
            justify-content: center;

            button {
                position: absolute;
                right: 1rem;
                top: 1rem;
            }
        }
        .modal-body {
            padding: 2rem 1rem 0.5rem;
        }
        .chad-box {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            padding: 1rem 1rem 0.4rem;
            margin-bottom: 1rem;
            background: #fcfcfc;
            box-shadow: 0 0 10px #9b3fd433;
            border-radius: 10px;
            border: 2px solid rgba(0, 0, 0, 0.1);
            .table-container {
                overflow-x: auto;
                width: 100%;
                &::-webkit-scrollbar {
                height: 5px;
                }

                &::-webkit-scrollbar-thumb {
                background-color: #aa73b3;
                border-radius: 10px;
                }

                &::-webkit-scrollbar-track {
                background-color: #f1f1f1;
                border-radius: 10px;
                } 
            }
            .title-stable {
                font-weight: 700;
                line-height: 100%;
                font-size: 1.1rem;
                color: #9b3fd4;
            }

            table {
                margin-bottom: 0.5rem;
                th {
                    text-align: center;
                    border-bottom: 2px solid rgba(0, 0, 0, 0.6);

                    &:first-child {
                        position: sticky;
                        left: 0;
                        text-align: left;
                        border-right: 1px solid rgba(0, 0, 0, 0.6);
                    }
                }
                td {
                    text-align: center;
                    padding: 1rem 0.5rem 0.5rem;
                    color: rgba(155, 63, 212, 1);
                    border-color: rgba(155, 63, 212, 1);
                    border-left: 1px solid rgba(155, 63, 212, 1);

                    &:first-child {
                        position: sticky;
                        left: 0;
                        text-align: left;
                        font-weight: 600;
                        border-left: none;
                    }
                }
                .border-none {
                    td {
                        border-bottom: none;
                        padding: 0.5rem;
                    }
                }
            }
        }
    }
}
</style>
