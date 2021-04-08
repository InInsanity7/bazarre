<template>
<div class="home">

    <div class="create-market">
        <button @click="createMarket">Create Market</button>
    </div>
    <div class="form" >
        <input v-model="findMarketDate" placeholder="Search the markets">
            <div class="suggestions" v-if="marketSuggestions.length > 0">
                <div class="suggestion" v-for="a in marketSuggestions" :key="a.id" @click="selectMarket(a)">{{a.openDate}}</div>
            </div>
    </div>





  <!-- <section class="image-gallery">
    <div class="image" v-for="item in items" :key="item.id">
      <h2>{{item.title}}</h2>
      <img :src="item.path" />
      <p>{{item.description}}</p>
    </div>
  </section> -->
</div>
</template>

<script>
// @ is an alias to /src
import axios from 'axios';
/* import MarketStands from '../components/MarketStands.vue'; */

export default {
  name: 'Home',
    data() {
    return {
     markets: [],
     addMarket: null,
     findMarketDate: "",
     findMarket: null,
    }
  },
    created() {
    this.getMarkets();
  },
  computed: {
    marketSuggestions() {
      let markets = this.markets.filter(market => market.openDate.startsWith(this.findMarketDate));
      return markets.sort(function(a, b){return b.openDate - a.openDate});
    },

    /* marketSuggestions() { */
    /*   let markets = this.markets.filter(market => this.findMarketDate); */
    /*   return markets.sort((a, b) => a.findMarketDate > b.findMarketDate); */
    /* } */
  },

    methods: {
    selectMarket(market) {
        this.findMarketDate = "";
        this.findMarket = market;
        /* this.findItem = null; */
        this.addMarket = null;
        /* this.addStand = null; */
        /* this.showPreview = false; */
        this.getMarkets();
    },

    async getMarkets() {
      try {
        let response = await axios.get("/api/markets");
        this.markets = response.data;
      } catch (error) {
        /* console.log(error); */
      }
    },
    async createMarket() {
        try {
            const market = await axios.post("/api/markets", {
               /* expired: false, */
               /* open: false, */
               openDate: Date.now(),
               /* stands: null, */
            });
            this.addMarket = market.data;
            this.getMarkets();
        } catch (error) {
            console.log(error);
            console.log(Date.now());
        }
    },

/*
    async createStand() {
      try {
        let r1 = await axios.post("/api/stands", {
          atMarket: false,
          expired: false,
          title: this.standTitle,
        });
        this.addStand = r1.data;
        this.standTitle = ""
        this.getStands();
      } catch (error) {
          console.log(error);
      }
    },
*/




  }
}
</script>






<style scoped>
.image h2 {
  font-style: italic;
  font-size: 1em;
}

.heading {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
}

.heading h2 {
  margin-top: 8px;
  margin-left: 10px;
}

.add,
.edit {
  display: flex;
}

.circle {
  border-radius: 50%;
  width: 18px;
  height: 18px;
  padding: 8px;
  background: #333;
  color: #fff;
  text-align: center
}

/* Form */
input,
textarea,
select,
button {
  font-family: 'Montserrat', sans-serif;
  font-size: 1em;
}

.form {
  margin-right: 50px;
}

/* Uploaded images */
.upload h2 {
  margin: 0px;
}

.upload img {
  max-width: 300px;
}
/* Suggestions */
.suggestions {
  width: 200px;
  border: 1px solid #ccc;
}

.suggestion {
  min-height: 20px;
}

.suggestion:hover {
  background-color: #5BDEFF;
  color: #fff;
}
</style>
