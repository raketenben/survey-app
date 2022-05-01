<template>
    <div class="items">
        <table class="list">
            <tr>
                <td>
                    <input ref="input" type="text" placeholder="Item" autofocus v-on:keypress.enter="addItem">
                </td>
                <td>
                    <img src="/iconmonstr-plus-2.svg" class="icon" alt="Add" @click="addItem"/>
                </td>
            </tr>
            <tr v-for="item in items" :key="item">
                <td>
                    <span>{{item}}</span>
                </td>
                <td>
                    <img src="/iconmonstr-x-mark-2.svg" class="icon" alt="Remove" @click="removeItem(item)">
                </td>
            </tr>
            <tr colspan="2">
                <input type="submit" v-if="items.length >= 2" value="Enter" @click="submit">
                <p class="info" v-else>You need to have at least two items in your list</p>
            </tr>
        </table>
        <div class="row">
        </div>
    </div>
</template>

<script>
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
    props: [
        'options',
        'radius',
    ],
    data: () => ({
        items: [],
    }),
    methods: {
        addItem: function(e) {
            if(this.$refs.input.value.length > 0) {
                this.items.push(this.$refs.input.value);
                this.$refs.input.value = '';
                this.$refs.input.focus();
            }
        },
        removeItem: function(item) {
            this.items = this.items.filter(i => i !== item);
        },
        submit: function() {
            this.$emit('submit', this.items);
        },
        onMounted: function() {
            this.$refs.input.focus();
        },
    },
})
</script>

<style scoped>
    input {
        background: transparent;
        width: 100%;
        padding: 0.1em 0.5em ;
        border-radius: 0.5rem;
        border: solid 2px rgba(255, 255, 255, 0.7);
        color: white;
    }

    input[type="submit"] {
        cursor: pointer;
    }

    .list {
        overflow: auto;
    }

    .info {
        color: #aaa;
        font-size: 0.6em;
        text-align: center;
    }

    .icon {
        width: 1.5em;
        height: 1.5em;
        margin: 0.1em;
        cursor: pointer;
        border-radius: 0.5rem;
        border: solid 2px rgba(255, 255, 255, 0.7);
        padding: 0.4em;
    }

    tr {
        display: flex;
        align-items: center;
        justify-content: space-between;
        flex-flow: row;
        padding: 0.1em;
    }

    td {
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
    }

    td span {
        font-size: 0.8em;
        padding: 0em 0.1em;
    }

    @media (min-width: 768px) {
        .items {
            display: flex;
            flex-flow: column;
            height: 80%;
        }
    }
</style>