<template>
    <div class="dialog-overlay" v-if="visible">
        <section class="dialog">
            <header class="dialog__header">
                <div class="dialog__title">{{ title }}</div>
                <span class="dialog-header__close" @click="visible=false">X</span>
            </header>
            <div class="dialog__body">
                {{ message  }}
            </div>
            <footer class="dialog__footer">
                <button class="dialog__btn">{{ cancelBtnTxt }}</button>
                <button class="dialog__btn dialog__btn--primary">{{ comfirmBtnTxt }}</button>
            </footer>
        </section>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const props = defineProps({
    title:{
        type:String,
        default:'Title'
    },
    message:String,
    cancelBtnTxt:{
        type:String,
        default:'Cancel'
    },
    comfirmBtnTxt:{
        type:String,
        default:'Comfirm'
    }
})
const visible = ref(true)
const close = ()=>{
    visible.value = false
}
</script>

<style scoped lang="scss">
:global(body){
    font-family: PingFang SC,Microsoft YaHei,Arial,sans-serif;
}
.dialog-overlay{
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 2000;
    height: 100%;
    background-color:rgba(0, 0, 0, .5);
    overflow: auto;
}

.dialog {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    border-radius:2px;
    background: #fff;
    width: 559px;
    min-height: 200px;
    &__header{
        padding: 20px 20px 10px;
    }
    &__title{
        color: #333;
        font-size: 18px;
    }
    &__body{
        padding: 30px 20px;
        font-size: 14px;
        color: #333;
    }
    &-header{
        &__close{
            position: absolute;
            right: 15px;
            top: 15px;
            color: #999;
            font-size: 18px;
        }
    }
    &__btn{
        min-width: 33px;
        padding: 8px 15px;
        background: none;
        border: 1px solid #ddd;
        border-radius: 2px;
        color: #666;
        margin-left: 10px;
        cursor: pointer;
        &--primary{
            background: #409eff;
            color: #fff;
            border-color:#409eff;
        }
    }
    &__footer{
        padding: 10px 20px 20px;
        display: flex;
        justify-content: flex-end;
    }
}
</style>