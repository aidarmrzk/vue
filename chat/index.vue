<template>
    <div class="scroll-container">
        <div class="container mt-5">
            <div class="row gx-5">
                <div class="d-flex align-items-center flex-column">
                    <h1>Привет, я Atom Chat</h1>
                    <span>Я использую нейросеть, чтобы общаться с вами. Как я могу вам помочь сегодня?</span>
                </div>
            </div>
        </div>
        <ChatForm />
    </div>
</template>

<script setup>

import ChatForm from '@/components/Admin/ChatForm.vue';

</script>

<style scoped>
    .scroll-container {
        max-height: 85vh;
        overflow-y: auto;
        overflow-x: inherit;
        padding: 10px;
    }
    .scroll-container::-webkit-scrollbar {
        width: 3px; /* толщина ползунка */
    }

    .scroll-container::-webkit-scrollbar-thumb {
        background-color: #d1d5db; /* цвет ползунка */
    }
</style>
