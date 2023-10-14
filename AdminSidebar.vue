<template>
    <nav class="bg-black sidebar" :class="{ 'not-verified': !isVerified }">
        <div class="pt-3 sidebar-sticky">
            <ul id="menu" class="nav flex-column mb-2">
                <router-link to="/" class="nav-link mb-3 main-link">
                    <img class="main-img" src="@/assets/img/chat/bot.svg" alt="chat-gpt" />
                    <p class="mb-0">GetAtom</p>
                </router-link>
                <li class="nav-item">
                    <router-link to="/admin" class="nav-link">
                        <i class="bi bi-house-door"></i>
                        <span class="d-none d-sm-inline ps-2">Маркетплейс</span>
                    </router-link>
                </li>
                <li class="nav-item">
                    <router-link to="/admin/chat" class="nav-link">
                        <i class="bi bi-chat-left-dots"></i>
                        <span class="d-none d-sm-inline ps-2">ChatGPT чат</span>
                    </router-link>
                </li>
                <li class="nav-item">
                    <router-link to="/admin/image" class="nav-link">
                        <i class="bi bi-card-image"></i>
                        <span class="d-none d-sm-inline ps-2">Генератор изображений</span>
                    </router-link>
                </li>
                <li v-if="can('post-list')" class="nav-item">
                    <router-link :to="{ name: 'posts.index' }" class="nav-link">
                        <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" fill="currentColor" class="bi bi-list" viewBox="0 0 16 16">
                            <path fill-rule="evenodd" d="M2.5 12a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5zm0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5zm0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5z"/>
                        </svg>
                        <span class="d-none d-sm-inline ps-2">Posts</span>
                    </router-link>
                </li>
                <li v-if="can('category-list')" class="nav-item">
                    <router-link :to="{ name: 'categories.index' }" class="nav-link">
                        <svg xmlns="http://www.w3.org/2000/svg" width="22" height="22" fill="currentColor" class="bi bi-list" viewBox="0 0 16 16">
                            <path fill-rule="evenodd" d="M2.5 12a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5zm0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5zm0-4a.5.5 0 0 1 .5-.5h10a.5.5 0 0 1 0 1H3a.5.5 0 0 1-.5-.5z"/>
                        </svg>
                        <span class="d-none d-sm-inline ps-2">Categories</span>
                    </router-link>
                </li>

                <li class="nav-item">
                    <router-link to="/admin/profile" class="nav-link">
                        <i class="bi bi-person"></i>
                        <span class="d-none d-sm-inline ps-2">Личный кабинет</span>
                    </router-link>
                </li>
            </ul>
        </div>
        <template v-if="user">
            <hr class="dropdown-divider">
            <div class="sidebar-bottom">
                <ul class="navbar-nav ms-auto mb-2 mb-lg-0">
                        <LocaleSwitcher />
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                                <div class="name-logo">{{ firstLetter }}</div>
                                <span>{{ user.name }}</span>
                                <div class="ellipsis">
                                    <svg stroke="currentColor" fill="none" stroke-width="2" viewBox="0 0 24 24" stroke-linecap="round" stroke-linejoin="round" class="h-4 w-4 flex-shrink-0 text-gray-500" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><circle cx="12" cy="12" r="1"></circle><circle cx="19" cy="12" r="1"></circle><circle cx="5" cy="12" r="1"></circle></svg>
                                </div>
                            </a>
                            <ul class="dropdown-menu dropdown-menu-end border-0 shadow-sm">
                                <!-- <li><a class="dropdown-item" href="#">Настройки</a></li> -->
                                <li>
                                    <a class="dropdown-item" :class="{ 'opacity-25': processing }" :disabled="processing"
                                    href="javascript:void(0)" @click="logout">
                                        <i class="bi bi-box-arrow-left"></i>
                                        <span>Выйти</span>
                                    </a>
                                </li>
                            </ul>
                        </li>
                    </ul>
            </div>
        </template>

    </nav>
</template>

<script setup>
import {useAbility} from '@casl/vue'
const {can} = useAbility();

import {computed} from "vue";
import { useStore } from 'vuex';
import useAuth from "@/composables/auth";

const store = useStore();
const user = computed(() => store.state.auth.user)
// const firstLetter = computed(() => store.state.auth.user?.name.charAt(0));
const firstLetter = computed(() => {
  const user = store.state.auth.user;
  if (user && user.name) {
    return user.name.charAt(0);
  }
  return ''; // или какое-то значение по умолчанию
});

const {processing, logout} = useAuth();

const isVerified = computed(() => store.getters["auth/getIsVerified"]);
</script>

<style scoped lang="scss">
    .bg-black {
        background-color: #202123 !important;
    }
    #menu {
        a {
            color: #fff;
            &:hover {
                background-color: rgba(42,43,50,1);
            }
        }
        .main-link {
            display: flex;
            gap: 0.5rem;
            @media (max-width: 768px) {
                justify-content: center;
                padding-left: 0;
                padding-right: 0;
            }
            .main-img {
                width: 20px;
                height: 20px;
            }
            @media (max-width: 768px) {
                p {
                    display: none;
                }
            }
        }
        @media (max-width: 768px) {
            .nav-item {
                a {
                    justify-content: center;
                    padding-left: 0;
                    padding-right: 0;
                    span {
                        &.d-sm-inline {
                            display: none !important;
                        }
                    }
                }
            }
        }
    }
    .sidebar {
        position: sticky !important;
        top: 0;
        &.not-verified {
            height: calc(100vh - 50px);
            top: 50px;
        }
        @media (max-width: 1024px) {
            &.bg-black {
                min-width: 180px;
                max-width: 180px;
            }
        }
        @media (max-width: 768px) {
            &.bg-black {
                min-width: 50px;
                max-width: 50px;
            }
        }
    }
    .sidebar-bottom {
        border-top: 1px solid #fff;
        position: absolute;
        bottom: 1rem;
        width: 100%;
        padding-top: 8px;
        @media (max-width: 768px) {
            .navbar-nav {
                margin: 0 !important;
                padding-top: 5px;
                .nav-item {
                    a {
                        justify-content: center;
                        padding: 0;
                        width: 50px;
                        .name-logo {
                            margin: 0;
                            width: 25px;
                            height: 25px;
                        }
                        span {
                           display: none;
                        }
                        .ellipsis {
                            display: none;
                        }
                    }
                }
            }
        }
    }
    .nav-item {
        .nav-link {
            color: #fff;
        }
        span {
            line-height: 130%;
        }
        .dropdown-menu {
            width: 100%;
            background-color: #000;
            @media (max-width: 768px) {
                min-width: 50px;
                bottom: 5px !important;
            }
            a {
                &:hover {
                    background-color: rgba(42,43,50,1);
                }
                &.dropdown-item {
                    i {
                        display: none;
                        @media (max-width: 768px) {
                            display: block;
                        }
                    }
                }
            }
        }
        a {
            display: flex;
            color: #fff;
            align-items: center;
            width: 100%;
            padding-left: 16px;
            .ellipsis {
                position: absolute;
                right: 1rem;
                top: 7px;
            }
            &::after {
                content: none;
            }
        }
        .name-logo {
            width: 28px;
            height: 28px;
            margin-right: 8px;
            background: linear-gradient(to right, #d87e8f, #aa73b3, #7e96e1);
            color: #fff;
            align-items: center;
            display: flex;
            justify-content: center;
            border-radius: 6px;
        }
    }
</style>
