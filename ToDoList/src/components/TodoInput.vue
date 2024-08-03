<script setup>
import { ref } from "vue";
import { statuses } from "../const/statuses";

const input = ref("");
const inputDate = ref("");
let isErrorMessage = ref(false);

function onSubmitForm() {

    if (input.value == "" || inputDate.value == "") {
        // エラーメッセージの表示
        event.preventDefault();
        isErrorMessage.value = true;
        return;
    }

    const items = JSON.parse(localStorage.getItem("items")) || [];
    const newItem = {
        id: items.length,
        content: input.value,
        limit: inputDate.value,
        state: statuses.NOT_START,
        onEdit: false,
    };

    items.push(newItem);
    localStorage.setItem("items", JSON.stringify(items));

}
</script>

<template>
    <div>
        <p v-if="isErrorMessage">タスク・期限に入力漏れがあります。</p>
        <form @submit="onSubmitForm">
            <label>やること<input type="text" v-model="input"></label>
            <label>期限<input type="date" v-model="inputDate"></label>
            <input type="submit" value="登録！" />
        </form>
    </div>
</template>