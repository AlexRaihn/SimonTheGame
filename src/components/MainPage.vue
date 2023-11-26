<template>
  <div>
    <h2>Simon Says</h2>
    <div>
      <div class="button_container">
        <div :class="stylesForButton[0].color" @click="CheckUsersChange(0)" :style="stylesForButton[0].opacityStyle"></div>
        <div :class="stylesForButton[1].color" @click="CheckUsersChange(1)" :style="stylesForButton[1].opacityStyle"></div>
      </div>
      <div class="button_container">
        <div :class="stylesForButton[2].color" @click="CheckUsersChange(2)" :style="stylesForButton[2].opacityStyle"></div>
        <div :class="stylesForButton[3].color" @click="CheckUsersChange(3)" :style="stylesForButton[3].opacityStyle"></div>
      </div>
    </div>
    <div>
      <button @click="RandomSubsequence()" v-if="isBeginGame== false">Начать</button>
      <button v-else @click="EndGame()">Стоп</button>
    </div>
    <p>Раунд: {{ counterRound }}</p>
    <div> 
      <p v-if="lastGame == 0">игр ещё не было</p>
      <p v-else>Последнняя игра закончилась на: {{ lastGame }} раунде</p>
    </div>
    <div v-if="isBeginGame == false">
      Легко <input type="radio" @click="timerForAnimation = 1500" checked name="complexity">
      Нормально <input type="radio" @click="timerForAnimation = 1000" name="complexity">
      Сложно <input type="radio" @click="timerForAnimation = 400" name="complexity">
    </div>
    <div v-else>
      Игра уже идёт, сложность нельзя изменить!
    </div>
  </div>
</template>

<script>
export default {
  name: 'MainPage',
  data () {
    return {
      //последовательнасть которую выбрал компьютер
      subsequence: [],
      //счётчик раундов + длинна последовательности для компьютера
      counterRound: 0,
      //счётчик, сколько пользователь нажал, до конца раунда
      counterUserClick: 0,
      //проверять начата ли игра и если нет, то не передавать значения
      isBeginGame: false,
      //результат прошлой игры
      lastGame: 0,
      //классы css для смены цвета по клику + звук
      stylesForButton: [
        {
          color: "block blue",
          opacityStyle: "opacity: 0.5;"
        },
        {
          color: "block green",
          opacityStyle: "opacity: 0.5;"
        },
        {
          color: "block red",
          opacityStyle: "opacity: 0.5;"
        },
        {
          color: "block yellow",
          opacityStyle: "opacity: 0.5;"
        }
      ],
      //интервал для анимации последовательности кликов
      startInterval: '',
      //таймер необходим чтобы интервал можно было завершить
      timer: 0,
      //здесь хранится кол-во времени между анимациями в интервале в зависимости от выбранной сложности
      timerForAnimation: 1500
    }
  },
  methods: {
    //метод определяющий последовательность кликов
    RandomSubsequence: function() {
        let randomNumber;
        if(this.counterRound == 0) {
          this.counterRound++;
          //старт игры
          this.isBeginGame = true;
          randomNumber = Math.floor(Math.random() * 4);
          this.subsequence.push(randomNumber);
        } else {
          for(let i = 0; i < this.counterRound; i++) {
            randomNumber = Math.floor(Math.random() * 4);
            this.subsequence.push(randomNumber);
          }
        }
        this.ReturnNewOpacity();
        console.log(this.subsequence);
    },
    //метод сверяющий выбранное пользователем с тем что находится в массиве последовательности
    CheckUsersChange: function(userChange) {
      if(this.isBeginGame == false) {
        alert("Игра не начата");
      } else {
        //включение аудио
        this.AudioPlayer(userChange);
        this.stylesForButton[userChange].opacityStyle = "opacity: 0.5;";
        if(userChange == this.subsequence[this.counterUserClick]) {
          this.counterUserClick++;
          if(this.subsequence.length == this.counterUserClick) {
            //переход на следующий раунд
            this.subsequence.length = 0;
            this.counterRound++;
            this.counterUserClick = 0;
            this.RandomSubsequence();
          }
        } else {
          this.EndGame();
        }
      }
    },
    //включение аудио в зависимости от клика по той или иной кнопке
    AudioPlayer: function(index) {
      let audio = new Audio(require('@/assets/sounds/'+(index+1)+'.mp3'));
      audio.play()
    },
    //отображение для игрока, какие кнопки нужно нажать ему
    ReturnNewOpacity: function() {
      //считываем длинну массива последовательностей, от этого будет зависеть длинна интервала анимации
      let len = this.subsequence.length;

      this.startInterval = setInterval(()=>{
  
        if(this.timer >= len) {
          //остановка интервала
          this.stopInt();
        } else {
          //добавление яркости на определённое время элементу в последовательности
          this.stylesForButton[this.subsequence[this.timer]].opacityStyle = "opacity: 1;"
          this.AudioPlayer(this.subsequence[this.timer]);
          
          setTimeout(()=>{
            this.ReturnOldOpacity(this.timer);
          }, this.timerForAnimation);
        }
      }, this.timerForAnimation + 200)
    },
    //возвращение кнопке исходного цвета
   ReturnOldOpacity: function(index) {
      this.stylesForButton[this.subsequence[index]].opacityStyle = "opacity: 0.5;";
      this.timer++;
   },
   //функция для остановки интервала
   stopInt: function() {
    clearInterval(this.startInterval);
    this.timer = 0;
   },
    //конец игры, если пользователь хочет закончить партию
    EndGame: function() {
          this.stopInt();
          this.lastGame = this.counterRound;
          //обнуляю все данные
          this.isBeginGame = false;
          this.subsequence.length = 0;
          this.counterUserClick = 0;
          this.counterRound = 0;
    }
  }
}
</script>

<style scoped>
.block{
  width: 100px;
  height: 100px;
}
.block:hover{
  cursor: pointer;
}
.blue{
  background-color: blue;
  margin: 1%;
}
.green{
  background-color: green;
  margin: 1%;
}
.red{
  background-color: red;
  margin: 1%;
}
.yellow{
  background-color: yellow;
  margin: 1%;
}
.button_container{
  display: flex;
  justify-content: center;
}
</style>
