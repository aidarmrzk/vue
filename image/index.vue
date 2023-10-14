<template>
    <div class="wrapper">
        <div class="container pt-5 container-title">
            <div class="row gx-5">
                <div class="d-flex align-items-center flex-column">
                    <h1>Привет, я Stable Diffusion изображения</h1>
                    <span class="title-text">Начните с подробного описания</span>
                </div>
            </div>
        </div>
        <form @submit.prevent="handleSubmit">
            <div class="input-group mt-3 mb-4">
                <input
                    v-model="userInput"
                    type="text"
                    class="form-control"
                    placeholder="Например: Объясни квантовую физику простыми словами"
                    aria-label="Recipient's username"
                    aria-describedby="button-addon2"
                />
                <button
                    class="chad-button"
                    type="button"
                    id="show-settings"
                    data-bs-toggle="modal"
                    data-bs-target="#settingsModal">
                    <i class="bi bi-gear-fill"></i>
                </button>
                <button
                    class="btn btn-outline-secondary chad-button"
                    type="submit"
                    id="button-addon2"
                    :disabled="
                        isLoading ||
                        (user &&
                            user.subscription &&
                            user.subscription.tokens_count < 0)
                    "
                >
                    Сгенерировать
                </button>
            </div>
        </form>
        <div class="spinner">
            <div
                :class="{
                    'spinner-grow text-primary': true,
                    'spinner-show': isLoading,
                }"
                role="status"
            ></div>
            <div
                :class="{
                    'spinner-grow text-primary': true,
                    'spinner-show': isLoading,
                }"
                role="status"
            >
                <span class="visually-hidden">Loading...</span>
            </div>
            <div
                :class="{
                    'spinner-grow text-primary': true,
                    'spinner-show': isLoading,
                }"
                role="status"
            ></div>
        </div>
        <div class="container">
            <div class="row row-cols-1 row-cols-md-3 g-4">
                <div
                    v-for="(imageObject, index) in reversedImagesArray"
                    :key="index"
                    class="col card-col"
                >
                    <div class="card">
                        <img
                            :src="imageObject.url"
                            class="card-img-top"
                            alt="stable-diffusion"
                            data-bs-toggle="modal"
                            data-bs-target="#imageModal"
                            :data-modal="imageObject.modal"
                            :data-style="imageObject.style"
                            :data-size="imageObject.size"
                            :data-steps="imageObject.steps"
                            @click="updateImage(imageObject.url, imageObject.prompt, imageObject.modal, imageObject.style, imageObject.size, imageObject.steps, this)"
                        />
                        <div class="card-body" @click="insertText(imageObject.prompt)">
                            <p class="card-text">{{ imageObject.prompt }}</p>
                        </div>
                        <div class="buttons-box">
                            <i
                                class="delete-icon bi bi-download"
                                @click="downloadImage(imageObject.url)"
                            ></i>
                            <div v-if="imageObject.showDeleteIcon">
                                <i
                                    class="delete-icon bi bi-trash"
                                    @click="
                                        showConfirmationButtons(imageObject)
                                    "
                                ></i>
                            </div>
                            <div class="delete-icons-box" v-else>
                                <i
                                    class="delete-icon bi bi-check"
                                    @click="confirmDelete(imageObject)"
                                ></i>
                                <i
                                    class="delete-icon bi bi-x"
                                    @click="cancelDelete(imageObject)"
                                ></i>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!-- Modal image -->
        <div
            class="modal fade"
            id="imageModal"
            tabindex="-1"
            aria-labelledby="imageModalLabel"
            aria-hidden="true"
        >
            <div class="modal-dialog modal-dialog-centered modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <button
                            type="button"
                            class="bi"
                            data-bs-dismiss="modal"
                            aria-label="Close"
                        ></button>
                    </div>
                    <div class="modal-body">
                        <div class="container">
                            <div class="image-container">
                                <img :src="selectedImage" alt="Image" />
                            </div>
                            <div class="image-info">
                                <h5>{{ $t("prompt") }}</h5>
                                <p class="prompt">{{ selectedPrompt }}</p>
                                <div class="flex">
                                    <span>{{ $t("model") }}</span>
                                    <p>{{ selectedModal }}</p>
                                </div>
                                <div class="flex">
                                    <span>{{ $t("style") }}</span>
                                    <p>{{ selectedStyle }}</p>
                                </div>
                                <div class="box">
                                    <div>
                                        <span>{{ $t("size") }}</span>
                                        <p>{{ selectedSize }}</p>
                                    </div>
                                    <div>
                                        <span>{{ $t("steps") }}</span>
                                        <p>{{ selectedSteps }}</p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Modal settings -->
        <div
            class="modal fade"
            id="settingsModal"
            tabindex="-1"
            aria-labelledby="settingsModalLabel"
            aria-hidden="true"
        >
            <div class="modal-dialog modal-dialog-centered modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-5" id="exampleModalLabel">
                            Настройки генерации
                        </h1>
                        <button
                            type="button"
                            class="bi"
                            data-bs-dismiss="modal"
                            aria-label="Close"
                        ></button>
                    </div>
                    <div class="modal-body">
                        <div class="container">
                            <div class="column">

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

                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button
                            type="button"
                            class="btn chad-button"
                            data-bs-toggle="modal"
                            data-bs-target="#exampleModal"
                        >
                            Таблица стоимости
                        </button>
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

const store = useStore();
const user = computed(() => store.getters["auth/user"]);
const imageSettings = computed(() => store.getters["setting/getImageSettings"]);
const imagePrice = computed(() => store.getters["auth/getImagePrice"]);
const imagesArray = computed(() => store.getters["auth/getImages"]);
let reversedImagesArray = computed(() => imagesArray.value.slice().reverse());
const userInput = ref("");
const isLoading = ref(false);

const handleSubmit = () => {
    const userRequest = userInput.value.trim();
    if (userRequest && user.value.subscription.tokens_count > 0) {
        isLoading.value = true;
        userInput.value = "";
        store
            .dispatch("auth/createImage", {
                message: userRequest,
                settings: imageSettings.value,
                imagePrice: parseFloat(imagePrice.value)
            })
            .then(() => {
                isLoading.value = false;
            })
            .catch(() => {
                isLoading.value = false;
            });
    }
};

const downloadImage = (imageUrl) => {
    const link = document.createElement("a");
    link.href = imageUrl;
    link.download = "image.jpg";

    const clickEvent = new MouseEvent("click", {
        view: window,
        bubbles: false,
        cancelable: true,
    });
    link.dispatchEvent(clickEvent);
};

const showConfirmationButtons = (imageObject) => {
    imageObject.showDeleteIcon = false;
};
const confirmDelete = (imageObject) => {
    store.dispatch("auth/deleteImage", imageObject.url);
    imageObject.showDeleteIcon = true;
};
const cancelDelete = (imageObject) => {
    imageObject.showDeleteIcon = true;
};

const insertText = (text) => {
  userInput.value = text;
};

const selectedImage = ref('');
const selectedPrompt = ref('');
const selectedModal = ref('');
const selectedStyle = ref('');
const selectedSize = ref('');
const selectedSteps = ref('');

const updateImage = (imageUrl, imagePromt, imageModal, imageStyle, imageSize, imageSteps, vueInstance) => {
    selectedImage.value = imageUrl;
    selectedPrompt.value = imagePromt;
    selectedSize.value = imageSize;
    selectedSteps.value = imageSteps;

    const modalNames = {
        "stable-diffusion-v1-5": "Stable Diffusion 1.5",
        "stable-diffusion-xl-beta-v2-2-2": "Stable Diffusion XL 0.8",
        "stable-diffusion-xl-1024-v1-0": "Stable Diffusion XL 1.0",
    };
    const styleNames = {
        "3d-model": vueInstance.$t("three_d_model"),
        "analog-film": vueInstance.$t("analog_film"),
        "anime": vueInstance.$t("anime"),
        "cinematic": vueInstance.$t("cinematic"),
        "comic-book": vueInstance.$t("comic_book"),
        "digital-art": vueInstance.$t("digital_art"),
        "enhance": vueInstance.$t("enhance"),
        "fantasy-art": vueInstance.$t("fantasy_art"),
        "isometric": vueInstance.$t("isometric"),
        "line-art": vueInstance.$t("line_art"),
        "low-poly": vueInstance.$t("low_poly"),
        "modeling-compound": vueInstance.$t("modeling_compound"),
        "neon-punk": vueInstance.$t("neon_punk"),
        "origami": vueInstance.$t("origami"),
        "photographic": vueInstance.$t("photographic"),
        "pixel-art": vueInstance.$t("pixel_art"),
        "tile-texture": vueInstance.$t("tile_texture"),
    };

    if (modalNames.hasOwnProperty(imageModal)) {
        selectedModal.value = modalNames[imageModal];
    }
    if (styleNames.hasOwnProperty(imageStyle)) {
        selectedStyle.value = styleNames[imageStyle];
    }

};

// Модалка с настройками
onMounted(() => {
    updatePriceImage();
})
const route = useRoute();
const router = useRouter();
const isSettingSidebar = ref(true);
let selectedImageStyle = ref("enhance");
let selectedImageCount = ref(1);
let selectedImageSteps = ref(50);
let selectedModel = ref("stable-diffusion-v1-5");
let selectedImageSize = ref("512х512");


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
.wrapper {
    margin: 0 -0.75rem;
    padding: 0 0 1rem;
    height: 100%;
    background: linear-gradient(
        311deg,
        rgba(25, 33, 46, 1) 0%,
        rgba(52, 16, 61, 1) 100%
    );
    .container-title {
        @media (max-width: 900px) {
            padding-top: 1rem !important;
        }
    }
    .input-group {
        padding: 0 0.75rem;
        max-width: 900px;
        margin-left: auto;
        margin-right: auto;
        @media (max-width: 1100px) {
            display: flex;
            justify-content: flex-end;
            align-items: flex-end;
            gap: 1rem;
            margin-bottom: 1rem !important;
        }

        input {
            background: linear-gradient(
                90deg,
                rgb(30, 30, 37) -30%,
                rgb(20, 0, 33) 100%
            );
            color: #fff5f5bf;
            border: none;
            @media (max-width: 1100px) {
                width: 100%;
                border-radius: 0.375rem !important;
            }
            &::placeholder {
                color: #fff5f5bf;
            }
            &:focus {
                border-color: rgb(155, 63, 212);
                box-shadow: 0 0 0 0.25rem rgba(163, 70, 244, 0.3);
            }
        }
        #show-settings {
            display: none;
            @media (max-width: 900px) {
                display: inline-block;
                width: 50px;
                margin-left: 0;
                background: linear-gradient(to right, #9b3fd4, #aa73b3);
                box-shadow: 0px 0px 10px 2px rgba(163, 70, 244, 0.3);
            }
        }
        .chad-button {
            display: inline-block;
            background-image: linear-gradient(
                87deg,
                rgb(155, 63, 212) 0%,
                rgb(118, 54, 223) 98%
            );
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
            padding: 8px 16px;
            @media (max-width: 1100px) {
                width: 150px;
                border-radius: 0.375rem !important;
            }
        }
    }
    .spinner {
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 20px;
        margin: 20px 0 30px;
        @media (max-width: 1100px) {
            margin: 1rem 0 1.5rem;
        }
        .spinner-grow {
            display: none;
            background: linear-gradient(
                87deg,
                rgb(155, 63, 212) 0%,
                rgb(118, 54, 223) 98%
            );
        }
        .spinner-show {
            display: block !important;
        }
    }
    .container {
        max-width: 100%;

        h1 {
            text-align: center;
            color: #d1d5db;
            @media (max-width: 900px) {
                font-size: calc(1rem + 1.2vw);
            }
        }
        .title-text {
            text-align: center;
            color: #d1d5db;
        }

        .card-col {
            width: 33.33%;
            @media (max-width: 1280px) {
            width: 50%;
            }
            @media (max-width: 640px) {
            width: 100%;
            }
            .card {
                height: 100%;
                border: none;
                overflow: hidden;
                cursor: pointer;
                .card-img-top {
                    width: 100%;
                    aspect-ratio: 1;
                    object-fit: cover;
                    object-position: center;
                    border-top-left-radius: initial;
                    border-top-right-radius: initial;
                }
                .card-body {
                    position: absolute;
                    bottom: 0;
                    left: 0;
                    width: 100%;
                    background: linear-gradient(
                        311deg,
                        rgb(25, 33, 46, 0.5) 0%,
                        rgb(52, 16, 61, 0.5) 100%
                    );
                    backdrop-filter: blur(1px);
                    transition: all 0.5s;
                    opacity: 0;
                    cursor: pointer;
                    overflow: hidden;
                    .card-text {
                        position: relative;
                        line-height: 1.2em;
                        max-height: 3.6em;
                        overflow: hidden;
                        color: rgba(255, 255, 255, 0.8);
                        transition: all 0.5s;
                    }
                    &:hover {
                        .card-text {
                            color: #fff;
                        }
                    }
                }
                .buttons-box {
                    position: absolute;
                    display: flex;
                    justify-content: flex-end;
                    gap: 5px;
                    top: 0;
                    right: 0;
                    padding: 10px;
                    width: 100%;
                    opacity: 0;
                    background: linear-gradient(
                        311deg,
                        rgb(25, 33, 46, 0.5) 0%,
                        rgb(52, 16, 61, 0.5) 100%
                    );
                    backdrop-filter: blur(1px);
                    transition: all 0.5s;
                    .bi {
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        width: 25px;
                        height: 25px;
                        cursor: pointer;
                        background-color: initial;
                        color: #fff;
                        font-size: 16px;
                        border-radius: 5px;
                        transition: all 0.5s;

                        &:hover {
                            background-color: rgba(155, 63, 212, 1);
                        }
                    }
                    .delete-icons-box {
                        display: flex;
                        background-color: #000;
                        border-radius: 5px;
                        gap: 3px;
                        .delete-icon {
                            font-size: 24px;
                            &.bi-check {
                                color: #ef4444bf;
                                &:hover {
                                    background-color: #000;
                                    color: #ef4444;
                                }
                            }
                            &.bi-x {
                                color: #fffc;
                                &:hover {
                                    background-color: #000;
                                    color: #fff;
                                }
                            }
                        }
                    }
                }
                &:hover {
                    .card-body {
                        opacity: 1;
                    }
                    .buttons-box {
                        opacity: 1;
                    }
                }
            }
        }
    }
    #imageModal {
        background-color: #18181b;
        .modal-dialog {
            width: 100%;
            height: 100%;
            max-height: 80%;
            max-width: 80%;
            margin-left: auto;
            margin-right: auto;
            @media (max-width: 834px) {
                max-height: 90%;
                max-width: 95%;
            }
            .modal-content {
                max-height: 100%;
                max-width: 100%;
                height: 100%;
                width: 100%;
                padding: 0;
                border: 1px solid rgb(63 63 70);
                background-color: #18181b;
                .modal-header {
                    padding: 0.5rem;
                    border-bottom: 1px solid rgb(63 63 70);
                    button {
                        &.bi {
                            display: flex;
                            align-items: center;
                            justify-content: center;
                            width: 28px;
                            height: 28px;
                            cursor: pointer;
                            font-size: 26px;
                            background-color: rgb(39 39 42);
                            color: rgba(255, 255, 255, 0.8);
                            box-shadow: none;
                            border: none;
                            border-radius: 5px;
                            margin: 0 0 0 auto;
                            transition: all 0.5s;
                            &::before {
                                content: "\f62a";
                            }
                            &:hover {
                                background-color: rgb(63 63 70);
                                color: rgba(255, 255, 255);
                            }
                        }
                    }
                }
                .modal-body {
                    height: calc(100% - 50px);
                    @media (max-width: 834px) {
                        padding: 10px;
                    }
                    .container {
                        display: flex;
                        justify-content: space-between;
                        gap: 10px;
                        height: 100%;
                        padding: 0;
                        @media (max-width: 834px) {
                            flex-direction: column;
                        }
                        .image-container {
                            display: flex;
                            justify-content: center;
                            align-items: center;
                            height: 100%;
                            width: 100%;
                            background-color: #000;
                            border-radius: 7px;
                            @media (max-width: 834px) {
                                height: 50%;
                            }
                            img {
                                height: auto;
                                width: auto;
                                max-height: 100%;
                                max-width: 100%;
                            }
                        }
                        .image-info {
                            width: 300px;
                            flex-shrink: 0;
                            font-size: 1rem;
                            @media (max-width: 1024px) {
                                width: 250px;
                                
                            }
                            @media (max-width: 834px) {
                                width: 100%;
                                font-size: 0.9rem;
                                flex-grow: 1;
                            }
                            h5 {
                                color: #fff;
                                opacity: .75;
                                @media (max-width: 834px) {
                                    margin: 0;
                                }
                                @media (max-width: 834px) {
                                    font-size: 0.9rem;
                                }
                            }
                            span {
                                color: #fff;
                                opacity: .75;
                            }
                            p {
                                margin: 0;
                                word-wrap: break-word;
                                color: #fff;
                                &.prompt {
                                    border-bottom: 1px solid rgb(63, 63, 70);
                                    margin: 0 0 1rem;
                                    padding: 0 0 10px;
                                    @media (max-width: 834px) {
                                        margin: 0 0 0.7rem;
                                    }
                                }
                            }
                            .flex {
                                display: flex;
                                gap: 10px;
                                margin: 0 0 1rem;
                                @media (max-width: 834px) {
                                    margin: 0;
                                }
                            }
                            .box {
                                display: flex;
                                margin: 0 0 1rem;
                                gap: 30px;
                                @media (max-width: 834px) {
                                    margin: 0;
                                }
                                div {
                                    display: flex;
                                    gap: 10px;
                                    width: 50%;
                                    p {
                                        margin: 0;
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    #settingsModal {
        .modal-content {
            background: linear-gradient(90deg, #1e1e25 -30%, #140021 100%);
            box-shadow: 0 0px 5px 0px rgba(141, 141, 141, 0.65);
            .modal-header {
                padding: 0.5rem;
                border-bottom: 1px solid #dee2e6;
                h1 {
                    color: #dee2e6;
                }
                button {
                    &.bi {
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        width: 28px;
                        height: 28px;
                        cursor: pointer;
                        font-size: 26px;
                        background-color: rgb(39 39 42 / 0%);
                        color: #dee2e6;
                        box-shadow: none;
                        border: none;
                        border-radius: 5px;
                        margin: 0 0 0 auto;
                        transition: all 0.5s;
                        &::before {
                            content: "\f62a";
                        }
                        &:hover {
                            background-color: rgb(63 63 70);
                            color: rgba(255, 255, 255);
                        }
                    }
                }
            }
            .modal-body {
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
                .setting-item {
                    padding-bottom: 30px;
                    @media (max-width: 575px) {
                        padding-bottom: 15px;
                    }
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
            .modal-footer {
                .chad-button {
                    display: inline-block;
                    width: 200px;
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
            }
        }
    }
}
</style>
