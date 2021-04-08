<template>

<div class="wrapper">
    <div class="upload">
        <h1>IN PREVIEW!!!</h1>

        <!-- <div class="stand" v-for="stand in stands" :key="stand.id"> -->
            <div class="stand-header">
                <h3>{{findStand.title}}</h3>
            </div>
            <div class="item" v-for="item in items" :key="item.id">
                <div class="item-header">
                    <p>{{item.title}}</p>
                </div>
                <div class="item-img">
                    <img :src="item.path"/>
                </div>
                <div class="item-description">
                    <p>{{item.description}}</p>
                </div>
                <div class="item-bid">
                    <p>Current bid: {{item.bid}}</p>
                </div>
                <div class="item-increment">
                    <button @click="bid(item)">Bid +${{item.increment}}</button>
                </div>
            </div>
           <!-- <button @click="toMarket()">Send to market!</button> -->
        <!-- </div>  -->
    </div>
</div>

</template>

<script>
import axios from 'axios';
export default {
    name: 'UserStand',
    props: {
        items: Array,
        findStand: Object,
    },
    data() {
        return {
         }   
    },
    methods: {
        async bid(item) {
      try {
        const upBid = item.bid + item.increment;
        await axios.put("/api/stands/" + this.findStand._id + "/items/" + item._id, {
          title: item.title,
          description: item.description,
          bid: upBid,
          increment: item.increment,
          hasBid: true,
        })
        this.findItem = null;
        this.$emit('bid');
      } catch (error) {
          /*console.log(error); */
      }
        },
    },


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
