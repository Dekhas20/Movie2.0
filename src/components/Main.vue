<template>
  <div>
    <h2 class="title">OMDb API</h2>
    <div class="tools">
      <div class="search-container">
        <input type="text" v-model="searchValue" @keyup.enter="searchMovie()" />
        <button @click="searchMovie()" class="search-button">Buscar</button>
      </div>

      <div class="types">
        <span :class="{ active: selectedType === '' }" @click="selectType('')"
          >Todo</span
        >
        <span
          :class="{ active: selectedType === 'movie' }"
          @click="selectType('movie')"
          >Peliculas</span
        >
        <span
          :class="{ active: selectedType === 'series' }"
          @click="selectType('series')"
          >Series</span
        >
      </div>
    </div>

    <div class="typeview">
      <button
        v-if="movies.length"
        @click="typeOfView = typeOfView == 'grid' ? 'table' : 'grid'"
      >
        <span v-if="typeOfView == 'grid'">Cambiar a tabla</span>
        <span v-if="typeOfView == 'table'">Cambiar a cuadricula</span>
      </button>
    </div>

    <div v-if="loading == 1" class="loader-container">
      <span class="loader"></span>
    </div>

    <template v-if="movies">
      <div class="movies-container" :class="{ table: typeOfView === 'table' }">
        <template v-if="typeOfView == 'grid'">
          <Movie
            @onclick="getMovie"
            v-for="movie in movies"
            :key="movie.imdbID"
            :movie="movie"
            :loading="loading"
          />
        </template>

        <table v-if="typeOfView == 'table'">
          <thead>
            <tr>
              <th>Nombre</th>
              <th>Año</th>
              <th>Tipo</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="movie in movies"
              :key="movie.imdbID"
              @click="getMovie(movie.imdbID)"
            >
              <td class="table-name">
                {{ movie.Title }}
                <span
                  v-if="loading == movie.imdbID"
                  class="loader-container row"
                >
                  <span class="loader row"></span>
                </span>
              </td>
              <td>{{ movie.Year }}</td>
              <td>{{ formattedType(movie.Type) }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-if="loading == 2" class="loader-container">
        <span class="loader"></span>
      </div>

      <div
        class="search-more"
        v-if="pages > 1 && lastPage < pages && loading == 0"
      >
        <button @click="searchMovie(lastPage + 1)">Cargar mas</button>
      </div>
    </template>

    <div v-if="error" class="msg">
      <h3>{{ error }}</h3>
    </div>

    <div v-if="error.length == 0 && movies.length == 0" class="msg">
      <h3>Escriba una palabra para buscar</h3>
    </div>

    <ModalData v-if="modalData" :movie="modalData" @closeModal="closeModal()" />
  </div>
</template>

<script>
import ModalData from "./ModalData.vue";
import Movie from "./Movie.vue";
import { computed } from "vue";

export default {
  components: {
    ModalData,
    Movie,
  },
  data() {
    return {
      searchValue: "",
      urlApi: "http://www.omdbapi.com/?apikey=dbe9de33",
      movies: [],
      pages: 1,
      lastPage: 1,
      modalData: undefined,
      error: "",
      selectedType: "",
      loading: 0,
      typeOfView: "grid",
    };
  },
  computed: {
    formattedType() {
      return (type) => {
        if (type === "movie") {
          return "Película";
        } else if (type === "series") {
          return "Serie";
        } else if (type === "game") {
          return "Juego";
        } else {
          return "Desconocido";
        }
      };
    },
    processedItems() {
      return this.movies.map((item) => ({
        ...item,
        formattedType: this.formattedType(item.type),
      }));
    },
  },
  mounted() {
    window.addEventListener("scroll", this.handleScroll);
  },
  methods: {
    handleScroll() {
      const scrollPosition = window.scrollY;
      const windowHeight = window.innerHeight;
      const documentHeight = document.documentElement.scrollHeight;

      if (
        scrollPosition + windowHeight >=
        documentHeight - (windowHeight / 100) * 30
      ) {
        if (!this.loading && this.lastPage < this.pages) {
          this.searchMovie(this.lastPage + 1);
        }
      }
    },
    async searchMovie(page = 1) {
      try {
        this.error = "";
        page == 1 ? (this.loading = 1) : (this.loading = 2);
        const response = await fetch(
          this.urlApi +
            "&s=" +
            this.searchValue +
            "&page=" +
            page +
            "&type=" +
            this.selectedType
        );
        if (!response.ok) {
          throw new Error("La solicitud no fue exitosa");
        }
        const data = await response.json();

        this.loading = 0;
        if (data.Response === "False") {
          if (data.Error === "Too many results.") {
            this.error = "Demasiados resultados, por favor sea más especifico";
            throw new Error("La solicitud no fue exitosa");
          } else if (data.Error === "Movie not found!") {
            this.error = "No se encontraron resultados";
            throw new Error("La solicitud no fue exitosa");
          } else {
            this.error = data.Error;
            throw new Error("La solicitud no fue exitosa");
          }
        }

        if (page > 1) {
          this.movies = [...this.movies, ...data.Search];
        } else {
          this.movies = data.Search;
        }
        this.pages = Math.ceil(data.totalResults / 10);
        this.lastPage = page;
      } catch (error) {
        console.error("Error al obtener datos:", error);
        this.error = error;
      }
    },
    async getMovie(id) {
      try {
        this.loading = id;
        console.log("ID:", id);
        const response = await fetch(this.urlApi + "&i=" + id);
        if (!response.ok) {
          throw new Error("La solicitud no fue exitosa");
        }
        this.loading = 0;
        const data = await response.json();
        this.modalData = data;
        console.log("Movie:", this.modalData);
      } catch (error) {
        console.error("Error al obtener datos:", error);
        this.error = error;
      }
    },
    closeModal() {
      this.modalData = undefined;
    },
    selectType(type) {
      this.selectedType = type;

      if (this.searchValue) {
        this.searchMovie();
      }
    },
  },
};
</script>

<style>
.msg {
  text-align: center;
}

.tools {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px 0;
  gap: 10px;
  height: 30px;
}

.search-container {
  display: flex;
  height: 100%;
  gap: 10px;
}

input {
  outline: none;
  height: 100%;
  border-radius: 5px;
  border: 1px solid #2979ff;
  text-indent: 10px;
  transition: all 0.3s ease-in-out;
  min-height: 30px;
}

input:focus {
  border: 1px solid #0056bf;
}

button {
  height: 100%;
  padding: 0 10px;
  box-sizing: border-box;
  text-align: center;
  background: #2979ff;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
  color: #fff;
  min-height: 30px;
}

button:hover {
  background: #0056bf;
}

.movies-container {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 20px;
  place-items: center;
  margin: 0 10px;
}

.movies-container.table {
  grid-template-columns: 1fr !important;
}

.search-more {
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 5px;
  margin: 20px 0;
}

.search-more button {
  padding: 10px 15px;
}

.types span {
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
  border: 1px solid #aaa;
  margin: 5px;
}

.active {
  border: 1px solid#0056bf !important;
  font-weight: bold;
  color: #0056bf;
}

.loader-container {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px 0;
}

.loader-container.row {
  position: absolute;
  top: 0;
  bottom: 0;
}

.loader {
  width: 48px;
  height: 48px;
  border: 5px solid #eee;
  border-bottom-color: #0056bf;
  border-radius: 50%;
  display: inline-block;
  box-sizing: border-box;
  animation: rotation 1s linear infinite;
}

.loader.row {
  width: 20px;
  height: 20px;
}

table {
  width: 70%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

tr {
  position: relative;
}

tr:hover {
  background: #d3f8ff;
  cursor: pointer;
}

.typeview {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20px;
  margin-right: 10px;
}

@keyframes rotation {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@media screen and (max-width: 1300px) {
  .movies-container {
    grid-template-columns: repeat(4, 1fr);
  }

  table {
    width: 85%;
  }
}

@media screen and (max-width: 1000px) {
  .movies-container {
    grid-template-columns: repeat(3, 1fr);
  }

  table {
    width: 100%;
  }
}

@media screen and (max-width: 800px) {
  .movies-container {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media screen and (max-width: 650px) {
  .movies-container {
    grid-template-columns: repeat(1, 1fr);
  }
  .tools {
    flex-direction: column;
    gap: 20px;
    margin-bottom: 20px;
    height: auto;
  }
}
</style>