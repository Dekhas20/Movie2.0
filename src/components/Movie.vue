<template>
  <div class="card" @click="emitOnClick">
    <div v-if="loading == movie.imdbID" class="loader-container">
      <span class="loader"></span>
    </div>

    <img :src="movie.Poster" @error="handleImageError()" :alt="movie.Title" />
    <div class="card-body">
      <h3>{{ movie.Title }}</h3>
      <p>{{ movie.Year }}</p>
    </div>
  </div>
</template>

<script>
const props = ["movie", "loading"];

export default {
  props: props,
  methods: {
    emitOnClick() {
      this.$emit("onclick", this.movie.imdbID);
    },
    handleImageError() {
      this.movie.Poster =
        "https://via.placeholder.com/300x450?text=Image+not+found";
    },
  },
};
</script>

<style scoped>
.card {
  width: 100%;
  height: 100%;
  overflow: hidden;
  border: 1px solid #eee;
  border-radius: 5px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  transition: all 0.3s ease-in-out;
  gap: 10px;
  position: relative;
}

.loader-container {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
}

.card:hover {
  cursor: pointer;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

img {
  width: 100%;
  object-fit: cover;
  border-radius: 5px;
}
</style>