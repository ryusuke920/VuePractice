<script setup>
import { ref } from "vue";
import { statuses } from "../const/statuses";
const items = ref(JSON.parse(localStorage.getItem("items")) || []);
const today = new Date(); // 今日の日付

let inputContent = ref(); // タスクの内容
let inputLimit = ref(); // タスクの期限
let inputState = ref(); // タスクのステータス
let ErrorMessage = ref(""); // エラーメッセージの内容
let isShowModal = ref(false); // モーダルを表示するか否か

function onEdit(id) {
    // 他に編集モードのタスクがないかを調べる
    let isOnEditOther = false;
    items.value.map((item) => {
        if (item.onEdit) {
            isOnEditOther = true;
            return;
        }
    });

    if (isOnEditOther) {
        // エラーメッセージの表示
        isErrorMessage.value = true;
        ErrorMessage.value = "他に編集中のタスクがあります。";
        return;
    }

    // タスクの編集
    items.value[id].onEdit = true;
    inputContent.value = items.value[id].content;
    inputLimit.value = items.value[id].limit;
    inputState.value = items.value[id].state;
}

let isErrorMessage = ref(false);
function onUpdate(id) {
    if (inputContent.value == "" || inputLimit.value == "") {
        isErrorMessage.value = true;
        ErrorMessage = "タスクの内容と期限を入力してください。";
        return;
    }
    // タスクの更新
    const newItem = {
        id: id,
        content: inputContent.value,
        limit: inputLimit.value,
        state: inputState.value,
        onEdit: false,
    };

    // id の内容以降を 1 つ newItem で上書き
    items.value.splice(id, 1, newItem);
    localStorage.setItem("items", JSON.stringify(items.value));
    isErrorMessage = false;
}

let deleteItemId = ""; // 削除対象の Item の ID
let deleteItemContent = ref(); // 削除対象の Item の ID
function showDeleteModal(id) {
    isShowModal.value = true;
    deleteItemId = id;
    deleteItemContent = items.value[id].content;
    console.log("削除するid: ", id);
}

function onDeleteItem() {
    isShowModal.value = false;
    items.value.splice(deleteItemId, 1);
    // ID を振り直す
    items.value = items.value.map((item, index) => ({
        id: index, // ここだけ振り直す
        content: item.content,
        limit: item.limit,
        state: item.state,
        onEdit: item.onEdit,
    }));

    // ローカルストレージに再度保存する
    localStorage.setItem("items", JSON.stringify(items.value));
}

function onHideModal() {
    isShowModal.value = false;
}

function sortById() {
    // ID でソートする
    items.value.sort((a, b) => a.id - b.id);
    localStorage.setItem("items", JSON.stringify(items.value));
}

function sortByLimit() {
    // 日付でソートする
    items.value.sort((a, b) => new Date(a.limit) - new Date(b.limit));
    localStorage.setItem("items", JSON.stringify(items.value));
}

</script>

<template>
    <div>
        <div v-if="isShowModal" class="modal">
            <div class="modal-content">
                <p>{{ deleteItemContent }} を削除してもよろしいですか？</p>
                <button @click="onDeleteItem()">はい</button>
                <button @click="onHideModal()">キャンセル</button>
            </div>
        </div>

        <p v-if="isErrorMessage">{{ ErrorMessage }}</p>
        <table>
            <tr>
                <th class="th-id">ID<button @click="sortById()">↓</button></th>
                <th class="th-value">やること</th>
                <th class="th-limit">期限<button @click="sortByLimit()">↓</button></th>
                <th class="th-state">状態</th>
                <th class="th-edit">編集</th>
                <th class="th-delete">削除</th>
            </tr>
            <tr 
                v-for="item in items"
                :key="item.id"
                :class="{ red: new Date(item.limit) < today }"
            >
                <td>{{ item.id }}</td>
                <td>
                    <span v-if="!item.onEdit">{{ item.content }}</span>
                    <input v-else type="text" v-model="inputContent">
                </td>
                <td>
                    <span v-if="!item.onEdit">{{ item.limit }}</span>
                    <input v-else type="date" v-model="inputLimit">
                </td>
                <td>
                    <span v-if="!item.onEdit">{{ item.state.value }}</span>
                    <select v-else v-model="inputState">
                        <option v-for="state in statuses"
                            :key="state.id"
                            :value="state"
                            :selected="state.id == item.state.id"
                        >
                        {{ state.value }}
                        </option>
                    </select>
                </td>
                <td>
                    <button v-if="!item.onEdit" @click="onEdit(item.id)">編集</button>
                    <button v-else @click="onUpdate(item.id)">完了</button>
                </td>
                <td>
                    <button @click="showDeleteModal(item.id)">削除</button>
                </td>
            </tr>
        </table>
    </div>
</template>

<style scoped>
.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal-content {
    background: #fff;
    padding: 20px;
    border-radius: 8px;
}

.red {
    color: red;
}
</style>