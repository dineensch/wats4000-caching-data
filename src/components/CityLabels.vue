<template v-for="favorites in favoriteCities" v-bind:key="favorites.id">
      <span>
        <p class="city-label" v-if="newLabel.length != 0"> {{ newLabel }}</p>
        <button v-on:click="showForm = !showForm" class="addLabel">Add Label</button>
        <span v-if="showForm">
          <form v-on:submit="saveLabel(favorites)">
            <input type="text" v-model="newLabel">
            <button type="submit" class="addLabel">Update</button>
          </form>
        </span>           
    </span>
</template>

<script>
export default {
  name: 'CityLabels',
  data () {
    return {
      showForm: false,
      newLabel: "",
      favorites: [],
    }
  },
  props: {
    cityLabel: String,
  },
  methods: {
    saveLabel: function(favorites) {
      this.newLabel.push(favorites);
      this.$ls.set("favoriteCities", this.newLabel);
  }
}
}
</script>

<style scoped>
.favorite-cities {
  list-style-type: none;
  padding: 10px;
  background: #ccc;
  width: 25%;
  float: right;
}
.city-label {
  color: #2c3e50;
  font-weight: 600;
}
.remove {
  font-size: 0.8rem;
  color: white;
  background: #AA0000;
  padding: 2px;
  cursor: pointer;
}

.addLabel {
  font-size: 0.6rem;
  color: white;
  background: black;
  padding: 2px;
  cursor: pointer;
  transition-duration: 0.4s;
}

.addLabel:hover {
  background-color: white;
  color: black;
}
</style>