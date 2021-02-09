<template>
    <div>
        <div>
            <div class="text-danger" v-if="controller.jsonstr && jsonerror">{{ jsonerror }}</div>
            <div class="text-success" v-if="!jsonerror">Valid JSON</div>
            <div class="edit_window">
                <!-- окно ввода -->
                <textarea class="editor form-control" type="text" 
                    @keydown.tab.prevent="{addTab}"
                    tabindex="-1"
                    rows="10"
                    @input="{prettyFormat,makeLog,hightLightText,makeFolding}"
                    v-model="controller.jsonstr" 
                    ref="jsonText" 
                    placeholder="скопируйте или начните вводить ваш JSON код...">
                </textarea>
                <!-- подсветка -->
                <div contenteditable="true" class="hightlighter_box" ref="jsonBack">
                    <pre class="hightlighter" v-html="controller.jsonhtml"></pre>
                </div>
                <!-- стрелочки -->
                <div class="folding">
                    <div class="folding_item" v-for="(item, index) in controller.foldingmas" :key="index" @click="turnCode(index)">{{item}}</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ['controller'],
    data() {
        return {
            jsonerror: ""
        }
    },   
    computed: {
        // Функция автоформатирования кода и указания ошибки
        prettyFormat: function() {
            this.jsonerror = ""
            let jsonValue = ""

            try {
                jsonValue = JSON.parse(this.controller.jsonstr)
                this.controller.jsoncorrect = true;

                var chekmass = this.controller.jsonstr.split("");
                var lastChar = chekmass[chekmass.length - 1];

                if(this.controller.autoformation)
                    if(lastChar == '}' || lastChar == '\"' || lastChar == ']' || lastChar == ',' || lastChar == ' ')
                        setTimeout(() => {
                            this.controller.jsonstr = JSON.stringify(jsonValue, null, 2);
                            this.$emit('hightlight-text');
                        }, 100);
            }
            catch(e) {
                this.jsonerror = JSON.stringify(e.message)
                this.controller.jsoncorrect = false;
                return ""
            }
        },
        // Функция переопределения нажатия Tab
        addTab: function() {
            var textarea = this.$refs.jsonText;
            var lastPosition = textarea.selectionStart; // добавление отступа вместо перехода к другому элементу
            this.controller.jsonstr = this.controller.jsonstr.substr(0,lastPosition) + "\t" + this.controller.jsonstr.substr(lastPosition);
            setTimeout(() => {
                textarea.selectionStart = textarea.selectionEnd =  lastPosition + 1;
            }, 1);
        },
        // Функция записи кода в буфер
        makeLog: function() {
            if(this.controller.iterator >= this.controller.stringlog.length){
                this.controller.stringlog.splice(-(this.controller.stringlog.length - this.controller.iterator));
                this.controller.iterator = this.controller.stringlog.length - 1; 
            }
            this.controller.stringlog.push(this.controller.jsonstr);
            this.controller.iterator += 1;
        },
        // Функция подсветки кода
        hightLightText: function(){
            this.$emit('hightlight-text');
        },
        // Функция отображения стрелок свертывания/развертывания 
        makeFolding: function(){
            this.$emit('make-folding');
        }
        
    },
    methods: {
        // Функция сворачивания кода
        turnCode: function(indexToHide){
            if(this.controller.foldingmas[indexToHide] == ">")  // если нажатая стрелка >
            {
                var rows = this.controller.jsonstr.split("\n"); // то найти где свернуто
                var count = 0;
                rows.some(function(element, index) {
                    if(element.match(/\.\.\./))
                    {
                        count += 1;
                        if(index >= indexToHide)
                            return element;
                    }
                });
                setTimeout(() => {  // и развернуть код
                    var itr = 0;
                    var txt = this.controller.foldedstrs[indexToHide]; 
                    this.controller.jsonstr = this.controller.jsonstr.replace(/\.\.\./g, function(dots){
                        itr += 1;
                        if(itr == count)
                            return txt;
                        else
                            return dots;
                    });
                    this.controller.foldedstrs[indexToHide] = undefined;
                    this.$emit('hightlight-text');
                    this.$emit('make-folding');
                }, 10);
            }
            else if (this.controller.foldingmas[indexToHide] == "v") // если нажатая стрелка v
            {
                var deep = 0;   // то найти, что сворачивать
                var strToHight = "";
                var rows = this.controller.jsonstr.split("\n");
                this.controller.foldingmas.some(function(element, index) {
                    if(index == indexToHide)
                        strToHight += rows[index].substr(rows[index].search(/{|\[/) + 1) + "\n";
                    if(index > indexToHide)
                    {
                        if (rows[index].match(/{|\[/))
                        {
                            deep += 1;
                        }
                        if (rows[index].match(/}|\]/))
                        {
                            if(deep == 0)
                            {
                                strToHight += rows[index].substr(0, rows[index].search(/}|\]/))+ "\n";
                                return strToHight; 
                            }
                            else
                                deep -= 1;
                        }
                        strToHight += rows[index]+ "\n";
                    }
                });
                strToHight = strToHight.trim();  // и свернуть код
                setTimeout(() => {
                    this.controller.foldingmas[indexToHide] = ">";
                    this.controller.jsonstr = this.controller.jsonstr.replace(strToHight, "...");
                    this.controller.foldedstrs[indexToHide] = strToHight;
                    this.$emit('hightlight-text');
                    this.$emit('make-folding');
                }, 10);
            }
        }
    }
}
</script>

<style>
    .folding{
        position: absolute;
        left: 12px;
        top: 8px;
    }
    .folding_item{
        height: 15px;
        cursor: pointer;
        opacity: 0.7;
    }
    .hightlighter_box{
        z-index: -1;
        position: absolute;
        width: 90%;
        top: 8px;
        left: 6px;
        letter-spacing: 0.18px;
    }
    .hightlighter{
        text-align: start;
        padding-left: 20px;
        line-height: 15px;
        margin: 0;
    }
    .edit_window{
        position: relative;
        display: flex;
        padding: 5px;
    }
    .editor{    
        background: none;
        color: rgba(0, 0, 0, 0.5);
        padding-left: 20px;
        width: 100%;
        height: 500px;
        resize: none;
        line-height: 15px;
    }
    .text-success{
        color: green;
    }
    .text-danger{
        color: red;
    }
</style>