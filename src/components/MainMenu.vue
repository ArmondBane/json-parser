<template>
    <div class="menu">
        <div class="checkbox_item item">
            <input type="checkbox" class="checkbox" @change="controller.autoformation = !controller.autoformation">
            <label class="text item">автоматическое форматирование кода</label> 
        </div>
        <a type="submit" class="button item" @click="previous()">Шаг назад</a>
        <a type="submit" class="button item" @click="next()">Шаг вперед</a>
        <a type="submit" class="button item" @click="saveFile()">Сохранить</a>
    </div>
</template>

<script>
export default {
    props: ['controller'],
    methods: {
        // Функция шага назад по коду
        previous(){
            if(this.controller.iterator >= 0)
            {
                this.controller.iterator -= 1;
                this.controller.jsonstr = this.controller.stringlog[this.controller.iterator];
                this.hightLightText() // подсветить код
                this.$emit('make-folding'); // выставить стрелочки сворачивания
            }
        },
        // Функция шага вперед по коду
        next(){
            if(this.controller.iterator < this.controller.stringlog.length - 1)
            {
                this.controller.iterator += 1;
                this.controller.jsonstr = this.controller.stringlog[this.controller.iterator];
                this.hightLightText(); // подсветить код
                this.$emit('make-folding'); // выставить стрелочки сворачивания
            }
        },
        // Функция сохранение файла
        saveFile() {
            this.$emit('save-file');
        },
        // Функция подсветки кода
        hightLightText: function(){
            if(this.controller.iterator != -1)
                this.$emit('hightlight-text');
            else
                this.controller.jsonhtml = "";
        }
    }
}
</script>

<style>
    .item{
        margin: auto;
        margin: 5px auto 5px auto;
    }
    .button{
        padding: 10px;
        border: 3px;
        border-color: gray;
        border-radius: 5px;
        background-color: #2c3e50;
        color: white;
        cursor: pointer;
    }
    .button:hover{
        background-color: #586c81;
    }
    .menu{
        display: flex;
        flex-wrap: wrap;
        padding: 5px 20px;
        align-content: space-between;
    }
    .checkbox_item{
        display: flex;
        align-items: center;
    }
    .checkbox{
        margin: 0;
        margin-right: 15px;
        height: 20px;
        transform: scale(1.5);
    }
</style>