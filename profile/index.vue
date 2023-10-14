<template>
    <div class="container mt-4">
        <div class="row gx-5">
            <div class="d-flex">
                <h3>Личный кабинет</h3>
            </div>
        </div>
    </div>
    <div class="container-sm border-0 mt-3">
        <div class="d-flex flex-column align-items-start mb-5 gap-2">
            <h5 class="mb-0">Покупка атомов</h5>
            <div class="d-flex gap-2 mb-1">
                <span>Ваш баланс:</span>
                <p class="mb-0">
                    <span class="title-stable">
                        {{
                            user.subscription.tokens_count <= 0
                                ? 0
                                : user.subscription.tokens_count
                        }}
                    </span>
                    атомов
                </p>
            </div>
            <button
                type="button"
                class="btn chad-button"
                data-bs-toggle="modal"
                data-bs-target="#exampleModal"
            >
                Купить атомы
            </button>
        </div>
        <div class="card-body">
            <h5>Изменение личных данных</h5>
            <form @submit.prevent="submitForm">
                <div class="d-flex w-100">
                    <div class="mb-3 w-50 pe-2">
                        <label for="name" class="form-label">Имя</label>
                        <input
                            type="text"
                            v-model="profile.name"
                            class="form-control"
                            id="name"
                        />
                        <div class="text-danger mt-1">
                            {{ errors.name }}
                        </div>
                        <div class="text-danger mt-1">
                            <div
                                v-for="message in validationErrors?.name"
                                :key="message"
                            >
                                {{ message }}
                            </div>
                        </div>
                    </div>
                </div>
                <div class="mb-3">
                    <button :disabled="isLoading" class="btn chad-button">
                        <div v-show="isLoading" class=""></div>
                        <span v-if="isLoading">Processing...</span>
                        <span v-else>Сохранить</span>
                    </button>
                </div>
            </form>
        </div>
        <!-- Modal -->
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
                            <b>Выберите количество атомов</b>
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
                            <div class="row">
                                <div
                                    class="col"
                                    v-for="(tarif, index) in tarifs"
                                    :key="index"
                                >
                                    <div
                                        class="chad-box"
                                        :class="{ best: index === 1 }"
                                    >
                                        <h5 class="card-title">
                                            {{ tarif.name }}
                                        </h5>
                                        <p class="card-text">
                                            <span class="title-stable">{{
                                                tarif.atoms
                                            }}</span>
                                            <br />
                                            атомов
                                        </p>
                                        <div class="tarif-info">
                                            <div>
                                                <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 40 40" fill="none"> <g filter="url(#filter0_d_232_1852)"> <circle cx="20" cy="20" r="12" fill="url(#paint0_linear_232_1852)"/> </g> <path d="M18.3333 24.1673L15 20.834L16.1667 19.6673L18.3333 21.834L23.8333 16.334L25 17.5007L18.3333 24.1673Z" fill="white"/> <defs> <filter id="filter0_d_232_1852" x="0" y="0" width="40" height="40" filterUnits="userSpaceOnUse" color-interpolation-filters="sRGB"> <feFlood flood-opacity="0" result="BackgroundImageFix"/> <feColorMatrix in="SourceAlpha" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 127 0" result="hardAlpha"/> <feMorphology radius="1" operator="dilate" in="SourceAlpha" result="effect1_dropShadow_232_1852"/> <feOffset/> <feGaussianBlur stdDeviation="3.5"/> <feComposite in2="hardAlpha" operator="out"/> <feColorMatrix type="matrix" values="0 0 0 0 0.501961 0 0 0 0 0.223529 0 0 0 0 0.866667 0 0 0 0.5 0"/> <feBlend mode="normal" in2="BackgroundImageFix" result="effect1_dropShadow_232_1852"/> <feBlend mode="normal" in="SourceGraphic" in2="effect1_dropShadow_232_1852" result="shape"/> </filter> <linearGradient id="paint0_linear_232_1852" x1="8" y1="32" x2="32.2896" y2="30.8883" gradientUnits="userSpaceOnUse"> <stop stop-color="#9B3FD4"/> <stop offset="1" stop-color="#7636DF"/> </linearGradient> </defs> </svg>
                                                <span>~ {{ tarif.words }} слов</span>
                                            </div>
                                            <div>
                                                <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 40 40" fill="none"> <g filter="url(#filter0_d_232_1852)"> <circle cx="20" cy="20" r="12" fill="url(#paint0_linear_232_1852)"/> </g> <path d="M18.3333 24.1673L15 20.834L16.1667 19.6673L18.3333 21.834L23.8333 16.334L25 17.5007L18.3333 24.1673Z" fill="white"/> <defs> <filter id="filter0_d_232_1852" x="0" y="0" width="40" height="40" filterUnits="userSpaceOnUse" color-interpolation-filters="sRGB"> <feFlood flood-opacity="0" result="BackgroundImageFix"/> <feColorMatrix in="SourceAlpha" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 127 0" result="hardAlpha"/> <feMorphology radius="1" operator="dilate" in="SourceAlpha" result="effect1_dropShadow_232_1852"/> <feOffset/> <feGaussianBlur stdDeviation="3.5"/> <feComposite in2="hardAlpha" operator="out"/> <feColorMatrix type="matrix" values="0 0 0 0 0.501961 0 0 0 0 0.223529 0 0 0 0 0.866667 0 0 0 0.5 0"/> <feBlend mode="normal" in2="BackgroundImageFix" result="effect1_dropShadow_232_1852"/> <feBlend mode="normal" in="SourceGraphic" in2="effect1_dropShadow_232_1852" result="shape"/> </filter> <linearGradient id="paint0_linear_232_1852" x1="8" y1="32" x2="32.2896" y2="30.8883" gradientUnits="userSpaceOnUse"> <stop stop-color="#9B3FD4"/> <stop offset="1" stop-color="#7636DF"/> </linearGradient> </defs> </svg>
                                                <span>~ {{ tarif.images }} изображений</span>
                                            </div>
                                        </div>
                                        <div class="bottom-box w-100">
                                            <div class="price">
                                                {{ tarif.price }} ₽
                                            </div>
                                            <a
                                                href="#"
                                                class="chad-button w-100"
                                                >Выбрать</a
                                            >
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { onMounted, reactive, watchEffect, computed } from "vue";
import { useStore, mapGetters } from "vuex";
import { useForm, useField, defineRule } from "vee-validate";
import { required, min } from "../../../validation/rules";
import useProfile from "@/composables/profile";
defineRule("required", required);
// defineRule('email', email)
defineRule("min", min);
const store = useStore();
const user = computed(() => store.getters["auth/user"]);

const schema = {
    name: "required|min:3",
    email: "required",
};

const tarifs = [
    { name: "Мини", atoms: 300, price: 290, words: 75000, images: 150 },
    { name: "Оптимум", atoms: 1000, price: 790, words: 225000, images: 450 },
    { name: "Plus", atoms: 3000, price: 1990, words: 750000, images: 1500 },
];

// Create a form context with the validation schema
const { validate, errors } = useForm({ validationSchema: schema });
// Define actual fields for validation
const { value: name } = useField("name", null, { initialValue: "" });
const { value: email } = useField("email", null, { initialValue: "" });
const {
    profile: profileData,
    getProfile,
    updateProfile,
    validationErrors,
    isLoading,
} = useProfile();
const profile = reactive({
    name,
    email,
});
function submitForm() {
    validate().then((form) => {
        if (form.valid) updateProfile(profile);
    });
}
onMounted(() => {
    getProfile();
});

watchEffect(() => {
    profile.name = profileData.value.name;
    profile.email = profileData.value.email;
});
</script>

<style scoped lang="scss">
.container {
    max-width: 100%;
    .row {
        justify-content: center;
        row-gap: 1rem;
        .col {
            width: 50%;
            max-width: 50%;
        }
    }
}
.container-sm {
    max-width: 1000px;
}
.modal-body {
    padding: 2rem 1rem;
    .row {
        row-gap: 2rem;
        .col {
            min-width: 230px;
            .chad-box {
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                align-items: center;
                padding: 2rem 1rem 1rem;
                height: 300px;
                background: #fcfcfc;
                box-shadow: 0 0 10px #9b3fd433;
                border-radius: 10px;
                border: 2px solid rgba(0, 0, 0, 0.1);
                transition: all 0.5s;
                &:hover {
                    border: 2px solid #9a5dc0;
                }
                &.best {
                    position: relative;
                    &:before {
                        content: "Чаще всего выбирают";
                        background: linear-gradient(87.38deg, #9b3fd4 0%, #df7503 96.98%);
                        position: absolute;
                        top: 0;
                        left: 50%;
                        width: max-content;
                        transform: translate(-50%, -50%);
                        border-radius: 20px;
                        padding: 0.25rem 1rem;
                        font-size: 0.875rem;
                        line-height: 1.25rem;
                        color: #fff;
                    }
                }
                .card-text {
                    margin: 0;
                    text-align: center;
                    .title-stable {
                        font-weight: 700;
                        line-height: 100%;
                        font-size: calc(1.5rem + 0.39vw);
                        color: rgba(155, 63, 212, 1);
                    }
                }
                .tarif-info {
                    display: flex;
                    flex-direction: column;
                    width: 100%;
                    padding: 10px 0 0;
                    div {
                        display: flex;
                        align-items: center;
                        svg {
                            flex-shrink: 0;
                            width: 25px;
                            height: 25px;
                        }
                        span {
                            white-space: nowrap;
                        }
                    }
                }
                .bottom-box div {
                    font-size: 1.125rem;
                    padding: 0.5rem;
                }
            }
        }
    }
}
.chad-button {
    display: inline-block;
    background-image: linear-gradient(
        87deg,
        rgba(155, 63, 212, 1) 0%,
        rgba(118, 54, 223, 1) 98%
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
    padding: 8px 45px;
}
@media (max-width: 600px) {
    form {
        .w-50 {
            max-width: 250px;
            width: 100% !important;
        }
    } 
}
</style>
