<template>
<div class="admin">
    <div class="page-header">
        <h1>Set up your stand!</h1>
            <div class="rules">
                <p>Here you can create your market stand.
                Items added to your stand can be added/modified/removed until you finalize your stand. 
                Once finalized, your stand will be sent to market to await the start of the next flea auction. 
                Feel free to make as many stands as you like! </p>
            </div>
    </div>

    <div class="form">
        <input v-model="standTitle" placeholder="Stand name">
        <p></p>
        <button @click="createStand">Create Stand</button>
    </div>
    <div class="form" >
        <input v-model="findStandTitle" placeholder="Search your stands">
            <div class="suggestions" v-if="standSuggestions.length > 0">
                <div class="suggestion" v-for="a in standSuggestions" :key="a.id" @click="selectStand(a)">{{a.title}}</div>
            </div>
    </div>

    <div class="upload" v-if="findStand">
        <input v-model="findStand.title">
        <p></p>
    </div>
    <div class="actions" v-if="findStand">
        <button @click="deleteStand()">Delete stand</button>
        <button @click="editStand()">Change stand name</button>
    </div>

<div class="inStand" v-if="findStand">
    <div class="heading">
      <div class="circle">1</div>
      <h2>Add an Item to your stand: {{findStand.title}}</h2>
    </div>
    <div class="add">
      <div class="form">
        <input v-model="title" placeholder="Title">
        <p></p>
        <textarea v-model="description" placeholder="Describe your item here!"></textarea>
        <input type="file" name="photo" @change="fileChanged">
            <p>Starting bid:</p>
            <input v-model="bid" placeholder="0"/>
            <p>Price increase per bid:</p>
            <input v-model="increment" placeholder="0"/>
        <button @click="upload">Upload</button>
      </div>
      <div class="upload" v-if="addItem">
        <img :src="addItem.path" />
        <p>Title:</p>
        <h2>{{addItem.title}}</h2>
        <p>Description: </p>
        <p>{{addItem.description}}</p>
        <p>Starting bid: {{addItem.bid}}</p>
        <p>Price increase per bid: {{addItem.increment}}</p>
      </div>
    </div>
        <div class="heading">
      <div class="circle">2</div>
      <h2>Edit/Delete an Item</h2>
    </div>
    <div class="edit">
      <div class="form">
        <input v-model="findTitle" placeholder="Search">
        <div class="suggestions" v-if="suggestions.length > 0">
          <div class="suggestion" v-for="s in suggestions" :key="s.id" @click="selectItem(s)">{{s.title}}
          </div>
        </div>
      </div>


      <div class="upload" v-if="findItem">
        <img :src="findItem.path" />
        <p>Title:</p>
        <input v-model="findItem.title">
        <p>Description:</p>
        <textarea v-model="findItem.description" placeholder="Describe your item here!"></textarea>
        <p>Starting bid:</p>
        <input v-model="findItem.bid" placeholder="0">
        <p>Price increase per bid:</p>
        <input v-model="findItem.increment" placeholder="0">
      </div>
      <div class="actions" v-if="findItem">
        <button @click="deleteItem()">Delete</button>
        <button @click="editItem()">Edit</button>
      </div>
    </div> 
</div>

    <div class="preview">
        <div class="heading">
      <div class="circle">3</div>
      <h2>Your Stand: </h2>
      <button @click="preview">Preview</button>
    </div>
    <div class="preview" v-if="showPreview">Test text for showPreview</div>
    <!-- UserStand Preview -->
    <UserStand :items="items" :findStand="findStand" v-if="showPreview" @bid="getItems()"/>
    </div>

</div>
</template>




<script>
import axios from 'axios';
import UserStand from "../components/UserStand.vue"

export default {
  name: 'Admin',
  components: {
      UserStand
  },
  data() {
    return {
//FIXME added variables:
      standTitle: "",
      addStand: null,
      findStandTitle: "",
      findStand: null,
      bid: 0,
      increment: 0,
      showPreview: false,


      title: "",
      description: "",
      file: null,
      addItem: null,
      findTitle: "",
      findItem: null,
      items: [],
      stands: [],
    }
  },
  computed: {
    standSuggestions() {
      let stands = this.stands.filter(stand => stand.title.toLowerCase().startsWith(this.findStandTitle.toLowerCase()));
      return stands.sort((a, b) => a.title > b.title);
    },

    suggestions() {
      let items = this.items.filter(item => item.title.toLowerCase().startsWith(this.findTitle.toLowerCase()));
      return items.sort((a, b) => a.title > b.title);
    }
  },
  created() {
      this.getStands();
      // this.getItems();
  },
  methods: {
    preview() {
        this.showPreview = true;
    },
    async editStand() {
      try {
        await axios.put("/api/stands/" + this.findStand._id, {
          title: this.findStand.title,
          atMarket: false,
          expired: false,
        });
        this.findStand = null;
        this.getStands();
      } catch (error) {
        /* console.log(error); */
      }
    },
    async deleteStand() {
        try {
            this.getItems();
               for (const item in this.items) {
                   this.deleteItem(item);
               }
            await axios.delete("/api/stands/" + this.findStand._id);
            this.findStand = null;
            this.getStands();
        } catch (error) {
            /* console.log(error); */
        }
    },
    async getStands() {
        try {
            let response = await axios.get("/api/stands");
            this.stands = response.data;
            return true;
        } catch (error) {
            /* console.log(error); */
        }
    },
    selectStand(stand) {
        this.findStandTitle = "";
        this.findStand = stand;
        this.findItem = null;
        this.addItem = null;
        this.addStand = null;
        this.showPreview = false;
        this.getItems();
    },

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
          /* console.log(error); */
      }
    },

    async editItem() {
      try {
        await axios.put("/api/stands/" + this.findStand._id + "/items/" + this.findItem._id, {
          title: this.findItem.title,
          description: this.findItem.description,
          bid: this.findItem.bid,
          increment: this.findItem.increment,
          hasBid: false,
        })
        this.findItem = null;
        this.getItems();
      } catch (error) {
        /* console.log(error); */
      }
    },
    async deleteItem() {
      try {
        await axios.delete("/api/stands/" + this.findStand._id + "/items/" + this.findItem._id);
        this.findItem = null;
        this.getItems();
      } catch (error) {
        /* console.log(error); */
      }
    },
    selectItem(item) {
      this.findTitle = "";
      this.findItem = item;
      this.addItem = null;
    },
     async getItems() {
        try {
            let response = await axios.get("/api/stands/" + this.findStand._id + "/items");
            this.items = response.data;
            /* return true; */
        } catch (error) {
            /* console.log(error); */
        }
     },
     fileChanged(event) {
         this.file = event.target.files[0]
     },
       async upload() {
    try {
        const formData = new FormData();
        formData.append('photo', this.file, this.file.name);
        let r1 = await axios.post('/api/photos', formData);
        let r2 = await axios.post('/api/stands/' + this.findStand._id + '/items', {
          title: this.title,
          description: this.description,
          path: r1.data.path,
          bid: this.bid,
          increment: this.increment,
          hasBid: false,
        });
        this.addItem = r2.data;
        this.title = "";
        this.description = "";
        this.path = "";
        this.bid = 0;
        this.increment = 0;
        this.getItems();
      } catch (error) {
        /* console.log(error); */
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
