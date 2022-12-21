<script>

export default({
  name: "App",
  data() {
      return{
        weathers: [
          { 
            id: 1,
            date: this.currentDate(),
            city: "Kyiv",
            degree: "-",
          },
          { 
            id: 1,
            date: this.currentDate(),
            city: "Lviv",
            degree: "-",
          },
        ],
        cities: [
          {
            id: 1,
            name: "Kyiv"
          },
          {
            id: 2,
            name: "Lviv"
          },
        ],
      time: this.currentTime(),
      nextId: 2,
      searchCity: "",
      latitude: null,
      longitude: null,
      cityIsExist: false,
      cityIsUnknown: false,
      activeColor: 'red',
      urlKyiv: "https://api.open-meteo.com/v1/forecast?latitude=50.45&longitude=30.52&hourly=temperature_2m,weathercode&timezone=Africa%2FCairo",
      urlLviv: "https://api.open-meteo.com/v1/forecast?latitude=49.84&longitude=24.02&hourly=weathercode,temperature_120m,temperature_180m&timezone=Europe%2FBerlin",

    }
  },
  methods: {
      async addNewCity(){
        const newWeather = {
          date: this.currentDate(),
          city: this.searchCity,
          degree: '-'
        };

        if(this.searchCity == ""){
          return
        };
        
        
        localStorage.setItem("weatherInChoosenCity", JSON.stringify(this.weathers))

        if(this.existedCity(newWeather.city)){
          return
        }
      
        const coordinate = await fetch(
          `https://geocoding-api.open-meteo.com/v1/search?name=${newWeather.city}&count=1`
        )
        const cityData = await coordinate.json();
        
        if(cityData.results == undefined){
            this.unknownCity()
            this.searchCity = "";
            return 
        }  
        else{
            newWeather.city = cityData.results[0].name
            this.latitude = cityData.results[0].latitude
            this.longitude = cityData.results[0].longitude
        
            console.log(this.latitude, this.longitude)
            const normalCity =  this.normalizeCityName(newWeather.city)
            newWeather.city = normalCity

            this.weathers.push(newWeather)
        }
      


      
      const f = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${this.latitude}&longitude=${this.longitude}&hourly=temperature_2m,weathercode&timezone=Africa%2FCairo`
      );
      const data = await f.json();
      const createdDegree = this.weathers[this.weathers.length - 1];
      createdDegree.degree = `${Math.floor(data.hourly.temperature_2m[this.currentTime()])} °C`
      
      console.log(data);

      
      this.searchCity = "";
      this.latitude = null;
      this. longitude = null;
    },


    unknownCity(){
      this.cityIsUnknown = true
      setTimeout((() => {this.cityIsUnknown = false}), 3000)
      return
    },


    existedCity(city){
      for(let i = 0; i <= this.weathers.length - 1; i++){
        
          if(this.weathers[i].city == city){
            this.cityIsExist = true
            setTimeout((() => {this.cityIsExist = false}), 1000)
            this.searchCity = "";
            return true
        }
      }
    },


    deleteCity(weatherToDelete){
      this.weathers = this.weathers.filter(weather => weather != weatherToDelete)
      localStorage.setItem("weatherInChoosenCity", JSON.stringify(this.weathers))
    },
    

    normalizeCityName(city){
      const normalize = city
      const first = normalize[0].toUpperCase() + normalize.slice(1)
      return first
    },


    currentDate() {
        const current = new Date();
        const date = current.toLocaleDateString('en-GB', { day: 'numeric', month: 'long'}).replace(/ /g, ' ');
        return date;
      },


    currentTime(){
      const today = new Date();
      const mounth = today.toLocaleDateString('en-GB', { month: 'short'}).replace(/ /g, ' ');
      const hour = today.toLocaleTimeString('en-GB', { hour: 'numeric'}).replace(/ /g, ' ');
      console.log('here', hour)
      return hour
      
    },
    
  },

  mounted() {
      fetch(this.urlKyiv)
        .then((response) => response.json())
        .then((data) => {
          const currentDegreeKyiv = this.weathers[0]
          currentDegreeKyiv.degree = `${Math.round(data.hourly.temperature_2m[this.currentTime()])} °C`
        })


      fetch(this.urlLviv)
      .then((response) => response.json())
        .then((data) => {
          const currentDegreeLviv = this.weathers[1]
          currentDegreeLviv.degree = `${Math.round(data.hourly.temperature_120m[this.currentTime()])} °C`
        });

  },


  created(){
    const weathersData = localStorage.getItem("weatherInChoosenCity");
    if(weathersData){
      this.weathers = JSON.parse(weathersData)
    }
  },  
    
})
      
</script>

<template>
  <div class="wrapper">
    <div class="bgImg">
      <img src="./assets/world.svg" alt="" class="bgImage">
      <div class="header__title ">
        <p>World</p>
        <p>weather</p>
        <p>forecast</p> 
      </div>

    </div>
    <div class="header"> 
        <div class="header__spreadsheet-wrapper">
          <div class="button__wrapper">
            <input v-model="searchCity" type="text"  @keyup.enter="addNewCity()" class="form__search" id="select_city" list="inputCities" autocomplete="off" spellcheck="false" aria-label="Select city" data-bs-toggle="dropdown" aria-expanded="false">
            <datalist id="inputCities">
              <option v-for="city in cities"
                      :key="city.id"
                      :value="city.name"
              />              
            </datalist>
              <button v-on:click="addNewCity()" data-glow class="button" type="button">
                <span  class="button__text">Add city</span>
              </button>
          </div>
          <Transition>
            <p class="alert" v-if="cityIsExist" :style="{'color': activeColor}">This city is already exist</p>
          </Transition>
          <Transition>
            <p class="alert" v-if="cityIsUnknown" :style="{'color': activeColor}">Unknown city</p>
          </Transition>
          <!-- <div class="filter"> <button class="pageButton">Назад</button> <button class="filterButton">Вперед</button> </div> <div class="filterInput"> Фильтр: <input type="text" /> </div> -->
              <!-- <div class="header__spreadsheet head"> 
                <div class="head__date"> Date:</div>
                <div class="head__city">Your city:</div>
                <div class="head__degree">Degrees:</div>
                <div class="head__icon" >Weather:</div>
            </div> -->
            <div class="spreadsheet__wrapper" v-for="(weather, idx) in weathers" v-bind:key="idx">
              <div class="header__spreadsheet spreadsheet"> 
                <div class="date"> {{ weather.date }}</div>
                <div class="city">{{ weather.city }}</div>
                <div class="degree">{{ weather.degree }}</div>
                <div class="icon" style="width: 100%"><img src="./assets/sun_weather.svg" alt="sun" class="weatherIcon"></div>
                <div class="icon-delete">
                  <button v-on:click="deleteCity(weather)" class="button__delete" type="button">
                    <div><img src="./assets/delete.svg" alt=""></div>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    
  
</template>

<style scoped>



  .wrapper{
    width: 100vw;
    height: auto;
    margin: 0 auto;
    align-items: center;
    color: black;
    
  }

  .bgImg{
    width: 100vw;
    height: auto;

  }

  .bgImage{
    width: 100vw;
    height: 30vw;

  }

  .header{
    display: flex;
    justify-content: left;
    align-items:flex-start; 
  }

  .alert{
    display: flex;
    justify-content: start;
    margin-left: 20vw;
    position: absolute;

  }

  .filter{
    margin: 1vw 0 0 0;
    display: flex;
    justify-content: start;

  }

  .filter > button{
    margin: 0 1vw 0 1vw;
    padding: 0 1vw 0;
    background-color: rgba(42,96,117,0.9753035003063726);
    border-color: transparent;
    border-radius: 40px;
  }

  .filter > input {
    border-radius: 40px;
    border: 1px solid black;
  }



  .header__spreadsheet-wrapper{
    margin: 0vw auto 10vw  auto;
    background-size: cover;
    border-radius: 10px;
    width: 80vw;
    
    height: auto;
    text-align: center;
    align-items: center;
  
  }

  .header__title {
    font-family: 'Julius Sans One', sans-serif;
    text-transform: uppercase;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;  
    line-height: 1vw;
    font-size: 60px;
    color: white;
    position: absolute;
    top: 3.5vw;
    margin: 0vw 0 0 6vw;
  }
  .header__title p {
    padding-bottom: 3vw;
  }

  .header__spreadsheet {
    display: flex;
    justify-content: space-between;
    text-align: center;
    align-items: center;

    font-family: 'Manrope', sans-serif;
    font-weight: bold;
    font-size: 16px;
    margin: 6vw 0 0vw 0;
    padding: 0 0 1vw 0;

  }

  .header__spreadsheet > div{
    max-width: 11vw;
  }
  .head > div {
    margin: 0 1vw 0 17vw;
  }

  .spreadsheet {
    font-family: 'Abril Fatface', cursive;
    font-weight: bold;
    font-size: 20px;

    background: rgb(8,73,98);
    background: linear-gradient(90deg, rgba(8,73,98,1) 20%, rgba(42,96,117,0.9753035003063726) 42%, rgba(255,255,255,1) 85%); 
    border-radius: 20px;
    box-sizing: border-box;

    margin: 2vw 1vw 0 1vw;
    padding: 0.3vw 0;



    color: white

  }


  .date{
    margin: 0 0 0 4vw !important;
  }

  .head__date{
    margin: 0 0 0 4vw !important;
  }
  .city{
  }
  .degree{
  }
  .icon{
  
  }

  .date{
    
  }

  .form__search{
    background: rgb(255,255,255);
    border: 1px solid black;
    border-right: none;
    border-radius: 25px 0 0 30px;
    color: black;
    font-family: 'Manrope', sans-serif;
    font-size: 16px;
    padding: 0 0.5em;
    position: relative;
    height: 39px;
    width: 30vw;
  }

  input[type="text"]{
    font-size: 22px;
  }


  .weatherIcon{
    width: 3vw;
    height: 3vw; 
    z-index: 10;
  }

  .button__wrapper{ 
    position: relative;
    align-items: center;
    justify-content: center;
    display: flex;
    flex-direction: row;
    margin: 1vw 0 0 0;



  }

  .spreadsheet__wrapper{
    position: relative;
    padding-bottom: 1vw;
    overflow: visible;
    
  }

  .button{
    cursor: pointer;
    height: 39px;
    width: 11vw;
    margin: 0;
    overflow: hidden;
    border: 1px solid rgba(0, 0, 0, 1);
    border-radius: 0 40px 40px 0;
    

  }

  .button__text{
    padding: 0.5vw 1vw;
    font-family: 'Abril Fatface', cursive;
    color: black;
    text-transform: uppercase;
    letter-spacing: 5px;
    position: relative;
    z-index: 2;
    /* box-shadow: 0 0 5px rgba(0, 0, 0, 0.4); */



  }

  .icon-delete{
    position: absolute;
    right: 2vw;
    top: 20%;
    align-self: center;
  
  }

  .button__delete{
    width: 1.2vw;
    height: auto;

    border: none;
    background: transparent;
    cursor: pointer;
    z-index: 2;
    padding: 0px 0px;
  }


  .v-enter-active, .v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from, .v-leave-to {
  opacity: 0;
}

</style>
