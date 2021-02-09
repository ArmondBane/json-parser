<template>
  <div id="app" >
    <h1>Редактор JSON</h1>
    <hr>
    <MainMenu v-bind:controller="controller" @save-file="saveFile()" @hightlight-text="hightLightText()" @make-folding="makeFolding()"/>
    <Coder v-bind:controller="controller" @hightlight-text="hightLightText()" @make-folding="makeFolding()"/>
  </div>
</template>

<script>
import Coder from '@/components/Coder'
import MainMenu from '@/components/MainMenu'

export default {
  name: 'App',
  data() {
    return { //переменные
      controller  : {
        autoformation: false, //автоформатирование
        stringlog: [], //буфер строки JSON
        jsonstr: "", // строка JSON кода
        iterator: -1, // итератор буфера
        jsonhtml: "", // строка для подсветки
        foldingmas: [], // массив стрелок свертывания
        jsoncorrect: false, // корректность JSON кода
        foldedstrs: []  // массив свернутых частей кода
      }
    }
  }, 
  components: {
    Coder,  // Окно кода
    MainMenu // Меню с кнопками
  },
  methods: {
    // Функция сохранения файла
    saveFile: function() {
      // если пустая строка, то не сохранять
      if(this.controller.jsonstr == "")
        return;
      //записываем нашу строку и сохраняем
      const blob = new Blob([this.controller.jsonstr], {type: 'text/plain'})
      const e = document.createEvent('MouseEvents'),
      a = document.createElement('a');
      a.download = "test.txt";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
      e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
      a.dispatchEvent(e);
    },
    // Функция определения нажатия конкретных клавиш
    onKeyDown(event) {
      // Ctrl+S
      if (navigator.platform === "MacIntel" ? event.metaKey : event.ctrlKey && event.key === "s") {
        event.preventDefault()

        this.saveFile();
      }
    },
    // Функция выделения JSON кода
    hightLightText: function(){
        this.controller.jsonhtml = this.controller.jsonstr.replace(/"(\w*|\s*|\d*)*"/gi, str => "<span style=\"color: red;\">" + str + "</span>");
    },
    // Функция отображения стрелок свертывания/развертывания 
    makeFolding: function(){
        this.controller.foldingmas = [];
        if(this.controller.iterator == -1 || this.controller.jsoncorrect == false)
          return;
        var rows = this.controller.jsonstr.split("\n");
        rows.forEach((element, index) => {
            if(element.match(/{|\[/))
            {
                if(this.controller.foldedstrs[index] == undefined)
                  this.controller.foldingmas.push("v");
                else
                  this.controller.foldingmas.push(">");
            }
            else
                this.controller.foldingmas.push(" ");
        });
    }
  },
  // Обработчик нажатия клавиш
  created() {
    document.addEventListener('keydown', this.onKeyDown);
  },
  // Отслеживание глобальных событий
  mounted() {
    // При загрузке страницы
    window.onload = () =>{
      this.controller.jsonstr = window.localStorage.getItem('jsonstr');
      if(this.controller.jsonstr == undefined)
        this.controller.jsonstr = "";
      this.hightLightText();
      this.makeFolding();
    }
    // При закрытии страницы
    window.onbeforeunload = () => {
      window.localStorage.setItem('jsonstr', this.controller.jsonstr);
    }
  }
}
</script>

<style>
.text{
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
