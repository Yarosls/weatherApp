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
            gif: "" ,  
          },
          { 
            id: 2,
            date: this.currentDate(),
            city: "Lviv",
            degree: "-",
            gif: "",
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
      nextId: 1,
      searchCity: "",
      latitude: null,
      longitude: null,
      cityIsExist: false,
      cityIsUnknown: false,
      activeColor: 'red',
      filter: "",
      gifPath: "-",
      urlKyiv: "https://api.open-meteo.com/v1/forecast?latitude=50.45&longitude=30.52&hourly=temperature_2m,weathercode&timezone=Africa%2FCairo",
      urlLviv: "https://api.open-meteo.com/v1/forecast?latitude=49.84&longitude=24.02&hourly=weathercode,temperature_120m,temperature_180m&timezone=Europe%2FBerlin",

    }
  },
  methods: {
    async addNewCity(){
      const newWeather = {
        date: this.currentDate(),
        city: this.searchCity,
        degree: '-',
        gif: this.gifPath,
      };


      if(this.searchCity == ""){
        return
      };

      const coordinate = await fetch(
        `https://geocoding-api.open-meteo.com/v1/search?name=${newWeather.city}&count=1`
      )
      const cityData = await coordinate.json();
      
      if(cityData.results == undefined){
          this.unknownCity()
          this.searchCity = "";
          return 
      } 
      else if(this.weathers.find(city => city.city == cityData.results[0].name )){
          this.existedCity()
          return
      }
      else{
          newWeather.city = cityData.results[0].name
          this.latitude = cityData.results[0].latitude
          this.longitude = cityData.results[0].longitude
      
          const normalCity =  this.normalizeCityName(newWeather.city)
          newWeather.city = normalCity

          this.weathers.push(newWeather)
          this.filter = ""
      }
    


  
      const f = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${this.latitude}&longitude=${this.longitude}&hourly=temperature_2m,weathercode&timezone=Africa%2FCairo`
      );
      const data = await f.json();
      const lastCity = this.weathers[this.weathers.length - 1];
      console.log("minin", lastCity)
      lastCity.degree = `${Math.floor(data.hourly.temperature_2m[this.currentTime()])} °C`
      
      const weatherCode = data.hourly.weathercode[this.currentTime()]

      this.getCodePath(weatherCode)
      lastCity.gif = weatherCode
      console.log("pokazis", weatherCode);
    
      this.searchCity = "";
      this.latitude = null;
      this. longitude = null;

      localStorage.setItem("weatherInChoosenCity", JSON.stringify(this.weathers))
    },


    async getCodePath(weatherCode){
      await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${this.latitude}&longitude=${this.longitude}&weathercode&timezone=Africa%2FCairo`
      )
      .then((response) => {return response.json()}) 
      .then ((data) => {console.log("dtaa", data)})






      if(weatherCode == 1 || weatherCode == 2){
        return this.gifPath = "./gifs/2.gif"
      } 
      if(weatherCode == 45 || weatherCode == 48){
        return this.gifPath = "./gifs/45.gif"
      } 
      if(weatherCode == 51 || weatherCode == 5|| weatherCode == 55){
        return this.gifPath = "./gifs/53.gif"
      } 
      if(weatherCode == 63 || weatherCode == 65){
        return this.gifPath = "./gifs/63.gif"
      } 
      if(weatherCode == 71 || weatherCode == 73 || weatherCode == 75){
        return this.gifPath = "./gifs/73.gif"
      }
      if(weatherCode == 80 || weatherCode == 81 || weatherCode == 82){
        return  this.gifPath = "./gifs/63.gif"
      }
      else{
        this.gifPath = `./gifs/${weatherCode}.gif`
        return this.gifPath
      }
      
    },


    unknownCity(){
      this.cityIsUnknown = true
      setTimeout((() => {this.cityIsUnknown = false}), 3000)
      return
    },


    existedCity(){    
        this.cityIsExist = true
        setTimeout((() => {this.cityIsExist = false}), 1000)
        this.searchCity = "";
        return 
    },




    deleteCity(weatherToDelete){
      this.weathers = this.weathers.filter(weather => weather != weatherToDelete)
      localStorage.setItem("weatherInChoosenCity", JSON.stringify(this.weathers))
    },
    

    normalizeCityName(city){
      const str = city.replace(/( |^)[а-яёa-z]/g, function(x){ return x.toUpperCase(); }  );
      return str
    },



    currentDate() {
        const current = new Date();
        const date = current.toLocaleDateString('en-GB', { day: 'numeric', month: 'long'}).replace(/ /g, ' ');
        return date;
      },


    currentTime(){
      const today = new Date();
      const hour = today.getHours()
      return hour
      
    },

    filteredCities(){
      return this.weathers.filter( searchCity => searchCity.city.includes(this.filter) )
    }


  },

  mounted() {
    fetch(this.urlKyiv)
      .then((response) => response.json())
      .then((data) => {
        const currentSituationKyiv = this.weathers.find(city => city.city == "Kyiv")
        currentSituationKyiv.degree = `${Math.round(data.hourly.temperature_2m[this.currentTime()])} °C`
        
        const weatherCode = data.hourly.weathercode[this.currentTime()]
        this.getCodePath(weatherCode)
        currentSituationKyiv.gif = `./src/gifs/${weatherCode}.gif`         
      }),

      
    fetch(this.urlLviv)
      .then((response) => response.json())
      .then((data) => {
        const currentSituationLviv = this.weathers.find(city => city.city == "Lviv")
        currentSituationLviv.degree = `${Math.round(data.hourly.temperature_120m[this.currentTime()])} °C`
        
        const weatherCode = data.hourly.weathercode[this.currentTime()]
        currentSituationLviv.gif = `./src/gifs/${weatherCode}.gif` 
          
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
      <img src="./images/world.svg" alt="" class="bgImage">
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

          <div class="filter">Filter: <input type="text" v-model="filter" /></div>

              <!-- <div class="header__spreadsheet head"> 
                <div class="head__date"> Date:</div>
                <div class="head__city">Your city:</div>
                <div class="head__degree">Degrees:</div>
                <div class="head__icon" >Weather:</div>
            </div> -->
            <div class="spreadsheet__wrapper" v-for="(weather, idx) in filteredCities()" v-bind:key="idx">
              <div class="header__spreadsheet spreadsheet"> 
                <div class="date"> {{ weather.date }}</div>
                <div class="city">{{ weather.city }}</div>
                <div class="degree">{{ weather.degree }}</div>
                <div class="icon" style="width: 100%"><img :src="weather.gif" :alt="icon" class="weatherIcon"></div>
                <div class="icon-delete">
                  <button v-on:click="deleteCity(weather)" class="button__delete" type="button">
                    <div><img src="./images/delete.svg" alt=""></div>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    
  
</template>

<style scoped lang="scss"> 



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
    height: 25vw;

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
    font-family: 'Montserrat', sans-serif;

  }

  .filter{
    font-family: 'Montserrat', sans-serif;
    margin: 1vw 0 0 1vw;
    
    display: flex;
    justify-content: start;
    align-items: center;
  }

  .filter{ 
    input {
      border-radius: 40px;
      border: 1px solid black;
      width: 10vw;
      margin: 0 0 0 1vw;
      padding: 0 7px 0;
      font-family: 'Manrope', sans-serif;
      font-size: 16px;
    }
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
    margin: 0vw 0 0 5vw;
  }
  .header__title p {
    padding-bottom: 3vw;
  }

  .header__spreadsheet {
    display: flex;
    justify-content: space-between;
    text-align: center;
    align-items: center;

    font-family: 'Montserrat', sans-serif;
    font-weight: bold;
    font-size: 16px;
    margin: 6vw 0 0vw 0;
    padding: 0 0 1vw 0;

  }

  .header__spreadsheet{
    div{
      max-width: 11vw;
      
    }
  }
  .head > div {
    margin: 0 1vw 0 17vw;
  }

  .spreadsheet {
    font-family: 'Montserrat', sans-serif;
    font-weight: 400;
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
    font-family: "Montserrat", sans-serif;
    font-weight: 400;
    color: black;
    text-transform: uppercase;
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
