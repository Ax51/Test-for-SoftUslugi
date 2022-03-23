<script>
    export default {
        data() {
            return {
                plate: {
                    width: 667,
                    heigth: 500,
                    depth: 80,
                    weigth: 36
                },
                wallWidth:'',
                wallHeigth:'',
                openings:[
                    {
                        id:1,
                        width:'',
                        heigth:''
                    }
                ],
                openingWidth:'',
                openingHeigth:'',
                blockCost:'300',
                workCost: '150',
                getResult: false
            }
        }, 
        methods: {
            calcPlates() {
                const correctWallWidth = +this.wallWidth.replace(/,/g,'.') * 1000  // meters to mm
                const correctWallHeigth = +this.wallHeigth.replace(/,/g,'.') * 1000  // meters to mm
                const wallColumns = Math.ceil(correctWallWidth / this.plate.width)
                const wallRows = Math.ceil(correctWallHeigth / this.plate.heigth)
                const platesForWall = wallRows * wallColumns

                const savedPlatesArr = []
                const plateArea = this.plate.width * this.plate.heigth
                this.openings.forEach(i => {
                    const savedPlatesInOpening = (((+i.width.replace(/,/g,'.') * 10) * (+i.heigth.replace(/,/g,'.') * 10)) / plateArea).toFixed(2)
                    savedPlatesArr.push(+savedPlatesInOpening)
                })
                const allSavedPlates = Math.floor(savedPlatesArr.reduce((prev, curr) => prev + curr, 0))

                const result = platesForWall - allSavedPlates
                return result >= 0 ? result : "Проверьте правильность введенных данных"
            },
            calcCement() {
                const wallArea = Math.ceil(((+this.wallWidth.replace(/,/g,'.')) * (+this.wallHeigth.replace(/,/g,'.'))) * 10) / 10
                const cementConsumption = 2
                const totalCementRequired = wallArea * cementConsumption

                const bagCapacity = 25
                const bagsRequired = Math.ceil(totalCementRequired / bagCapacity)
                return [totalCementRequired, bagsRequired]
            },
            calcWeightPerMeter() {
                return Math.ceil((1000 / this.plate.width) /* amount of plates per meter */ * (this.plate.weigth + 1 /* 2 kilo of cement on every meter. plate is only half heigth of a meter */) * +this.wallHeigth.replace(/,/g,'.'))
            },
            calcAllWeight() {
                return (this.calcPlates() * this.plate.weigth) + (this.calcCement()[0])
            }
        }
    }
</script>

<template>
  <div class="wrapper">
      <h2><i>Расчет перегородок из пазогребневых плит</i></h2>
      <div class="colored-wrapper">
          <h2>Чтобы узнать нужное для перегородки количество пазогребевых плит:</h2>
          <div class="block">
                <p>Введите общую длину стены-перегородки в метрах погонных: <input id='wall-width' v-model="wallWidth" type='text' > м.</p>
                <p>Введите высоту её в метрах: <input id='wall-heigth' v-model="wallHeigth" type=text> м.</p>
          </div>
          <div class="block">
                <h2>Проёмы (для дверей и других целей):</h2>
                <p v-for="(opening, index) in openings" :key="opening.id">
                    ширина <input id='opening-width' v-model="openings[index].width" type=text> см. высота <input id='opening-heigth' v-model="openings[index].heigth" type=text> см.
                    <button @click="this.openings.pop()">
                        X
                    </button>
                    </p>
                <button
                    @click="this.openings.push({
                        id:this.openings[this.openings.length - 1].id + 1,
                        width:'',
                        heigth:''
                    })">
                    Добавить проём
                </button>
          </div>
          <div class="block">
                <p>Цена за 1 блок: <input id='block-cost' v-model="blockCost" type=text> р.</p>
                <p>Стоимость кладки одного блока: <input id='work-cost' v-model="workCost" type=text> р.</p>
          </div>
          <button @click="getResult = !getResult">
                Считать
          </button>
          <div class="result" v-if="getResult && typeof calcPlates() === 'number'">
              <p class="center">Результат:</p>
              <p>На данную перегородку потребуется:</p>
              <p>Пазогребневых гипсовых блоков: {{calcPlates()}} шт. общей стоимостью {{calcPlates() * this.blockCost}}р.</p>
              <p>Сухого клея для пазоргебневых плит: {{calcCement()[0]}}кг. ({{calcCement()[1]}} мешков по 25кг.) </p>
              <p>Стоимость работы по кладке: {{calcPlates() * this.workCost}}р.</p>
              <p>Вес всех плит составит: {{calcPlates() * this.plate.weigth}}кг.</p>
              <p>Общий вес доставки составит: {{calcAllWeight()}}кг.</p>
              <p>Нагрузка на метр погонный: {{calcWeightPerMeter()}}кг./м.п.</p>
          </div>
          <div class="error" v-if="typeof calcPlates() === 'string'">
              <h2>{{calcPlates()}}</h2>
          </div>
      </div>
  </div>
</template>

<style scoped>
    h2 {
        text-align: center;
    }
    .wrapper > h2 {
        color:#660000;
    }
    .colored-wrapper {
        padding: 10px;
        max-width: 1000px;
        background-color: #cef7da;
        display: flex;
        align-items: stretch;
        gap: 10px;
        flex-direction: column;
    }
    .block {
        padding: 10px;
        background-color: #e7f7ec;
        text-align: center;
    }
    .block > h2 {
        margin-top: 0;
    }
    input {
        width: 25px;
    }
    .colored-wrapper > button {
        margin: 5px;
        padding: 10px 30px;
        border-radius: 5px;
        background: linear-gradient(to bottom, rgba(205,235,142,1) 0%,rgba(165,201,86,1) 100%);
        cursor: pointer;
        font-size: 3rem;
        align-self: center;
    }
    .result, .error {
        padding:0 30px;
        background-color: #fafad2;
        border-radius: 10px;
    }
    .center {
        text-align: center;
    }
</style>