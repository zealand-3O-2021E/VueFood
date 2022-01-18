<template>

<div class="div">
  <img ref="myImage" src="http://192.168.1.12:8000/stream.mjpg" width="640" height="480">
  <button class="w-25 rounded-circle " v-on:click="capture()">Screen</button>
</div>
  <input type="file" @change="analyze( $event )"/>
  <ul id="results">
    <li v-for="item in result" :key="item.id">
      {{ item.name }} — {{(item.prob*100).toFixed(2)}}%
    </li>
  </ul>


  <div class="row m-auto " id="results">
    <input class=" text-center " type="text" @change="rec()" v-model="food"/>
    <div class="m-auto mt-2 col-5" v-for="item in recipes" :key="item.id">
      <img v-bind:src ="item.image" width="150" height="150">

      {{ item.title }}
      <button class="btn btn-outline-primary" v-on:click="getrecipes(item.id)"> See the recipy</button>
    </div>
  </div>

  <ul class="mt-5"  id="Ingredients">
    <h2>Ingredients</h2>
    <h6 v-for="item in seerecipes.extendedIngredients" :key="item.id">
     {{item.original}}
    </h6>
  </ul>
  <ul class="mt-2"  id="steps">
    <h2>Steps</h2>
    <h6 v-for="item in seerecipes.analyzedInstructions[0].steps" :key="item.id">
      {{ item.number}} {{item.step}}
    </h6>
  </ul>
  <ul  class="mt-2 row" id="nutrition">
    <h2>Nutrition</h2>
    <div class="col-3 border border-success" v-for="item in seerecipes.nutrition.nutrients" :key="item.id">
      {{ item.name}} {{item.amount}}{{item.unit}}
    </div>
  </ul>
</template>

<script>

export default {
  name: "Home",
  data(){
    return{
      result : [],
      recipes : [],
      food: "",
      seerecipes: {
        analyzedInstructions: [
          {
            steps: [],
          }

        ],
        nutrition: [],
        extendedIngredients: []
      }
    }
  },
  methods:{
    capture(){
      const c = document.createElement('canvas');
      const img = this.$refs["myImage"]
      c.width = img.width;
      c.height = img.height;
      const ctx = c.getContext('2d');

      ctx.drawImage(img, 0, 0);
      const dataURL = c.toDataURL('image/png');
      const formdata = new FormData();
      formdata.append("files",dataURL);
      fetch({
        type: "POST",
        url: "https://localhost:5002/api/Food",
        data: {
          imgBase64: formdata
        }
      }).done(function(response) {
        console.log('saved: ' + response);
      });
    },
    analyze(e){
      const formdata = new FormData();
      formdata.append("files", e.target.files[0]);
      fetch('https://localhost:5002/api/Food', {
        method: 'POST',
        credentials: 'include',
        body: formdata
      })
          .then(response => response.json())
          .then(result => {
            console.log(result)
            this.result = result.recognition_results
            this.cal
          })
    },
    rec(){
      const urlencoded = new URLSearchParams();
      urlencoded.append("name", this.food);
      fetch('https://localhost:5002/api/Food/op',{
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        credentials: 'include',
        body: urlencoded
      })
          .then(response => response.json())
          .then((result) => {
            this.recipes = result
          })

    },
    getrecipes(index){
      const urlencoded = new URLSearchParams();
      urlencoded.append("id", index);
      fetch('https://localhost:5002/api/Food/id',{
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        credentials: 'include',
        body: urlencoded
      })
          .then(response => response.json())
          .then((result) => {
            this.seerecipes = result
            console.log(this.seerecipes)
          })

    }
  }
}
</script>

<style scoped>

</style>