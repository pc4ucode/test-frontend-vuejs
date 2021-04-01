<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.svg')"
          class="my-3"
          contain
          height="200"
        />

        <v-progress-circular
          v-if="loader"
          class="mt-5"
          :size="70"
          :width="7"
          color="blue"
          indeterminate
        ></v-progress-circular>

        <v-data-iterator
          v-if="!loader"
          :items="items"
          :items-per-page.sync="itemsPerPage"
          :page.sync="page"
          :search="search"
          :sort-by="sortBy.toLowerCase()"
          :sort-desc="sortDesc"
          hide-default-footer
        >
          <template v-slot:header>
            <v-toolbar dark color="blue darken-3" class="mb-1">
              <v-text-field
                v-model="search"
                clearable
                flat
                solo-inverted
                hide-details
                prepend-inner-icon="mdi-magnify"
                label="Search"
              ></v-text-field>
            </v-toolbar>
          </template>

          <template v-slot:default="props">
            <v-row>
              <v-col
                v-for="(item, index) in props.items"
                :key="index"
                cols="12"
                sm="6"
                md="4"
                lg="3"
              >
                <v-card max-width="400" class="mx-auto" v-if="true">
                  <v-img :src="dogImage[index]" height="300px" dark>
                    <v-row class="fill-height">
                      <v-card-title>
                        <v-btn icon @click="saveFavorite(item.breed)">
                          <v-icon
                            color="red"
                            v-if="favorites.includes(item.breed)"
                          >
                            mdi-heart
                          </v-icon>
                          <v-icon v-else>
                            mdi-heart-outline
                          </v-icon>
                        </v-btn>
                      </v-card-title>

                      <v-card-title class="white--text pl-0 pt-5">
                        <div class="display-1 text-center">
                          {{ item.breed }}
                        </div>
                      </v-card-title>
                    </v-row>
                  </v-img>

                  <v-list two-line>
                    <v-row>
                      <v-col cols="6" sm="4"
                        v-for="subBreed in item.subBreed"
                        :key="subBreed"
                      >
                        <v-tooltip bottom>
                          <template v-slot:activator="{ on, attrs }">
                            <v-col>
                              <img
                                :src="dogSubImage[subBreed]"
                                height="80"
                                width="80"
                                v-bind="attrs"
                                v-on="on"
                              />
                            </v-col>
                          </template>
                          <span>{{ subBreed }}</span>
                        </v-tooltip>
                      </v-col>
                    </v-row>
                  </v-list>
                </v-card>

                <v-card v-if="false">
                  <v-card-title class="subheading font-weight-bold">
                    {{ item.breed }}

                    <v-btn icon @click="saveFavorite(item.breed)">
                      <v-icon color="red" v-if="favorites.includes(item.breed)">
                        mdi-heart
                      </v-icon>
                      <v-icon v-else>
                        mdi-heart-outline
                      </v-icon>
                    </v-btn>

                    <v-col cols="4">
                      <img :src="dogImage[index]" height="80" width="100" />
                    </v-col>
                  </v-card-title>

                  <v-divider />

                  <v-list dense>
                    <v-list-item
                      v-for="subBreed in item.subBreed"
                      :key="subBreed"
                    >
                      <v-list-item-content class="align-end">
                        {{ subBreed }}
                      </v-list-item-content>

                      <v-list-item-content>
                        <v-col cols="4">
                          <img
                            :src="dogSubImage[subBreed]"
                            height="80"
                            width="100"
                          />
                        </v-col>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list>
                </v-card>
              </v-col>
            </v-row>
          </template>

          <template v-slot:footer>
            <v-row class="mt-2 responsive-row" align="center" justify="center">
              <span class="grey--text">Items per page</span>
              <v-menu offset-y>
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    dark
                    text
                    color="primary"
                    class="ml-2"
                    v-bind="attrs"
                    v-on="on"
                  >
                    {{ itemsPerPage }}
                    <v-icon>mdi-chevron-down</v-icon>
                  </v-btn>
                </template>
                <v-list>
                  <v-list-item
                    v-for="(number, index) in itemsPerPageArray"
                    :key="index"
                    @click="updateItemsPerPage(number)"
                  >
                    <v-list-item-title>{{ number }}</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>

              <v-spacer></v-spacer>

              <span
                class="mr-4
                grey--text"
              >
                Page {{ page }} of {{ numberOfPages }}
              </span>
              <v-btn
                fab
                dark
                color="blue darken-3"
                class="mr-1"
                @click="formerPage"
              >
                <v-icon>mdi-chevron-left</v-icon>
              </v-btn>
              <v-btn
                fab
                dark
                color="blue darken-3"
                class="ml-1"
                @click="nextPage"
              >
                <v-icon>mdi-chevron-right</v-icon>
              </v-btn>
            </v-row>
          </template>
        </v-data-iterator>

        <v-snackbar
          v-if="snackbarError"
          :timeout="3000"
          :value="true"
          absolute
          bottom
          color="error"
          right
        >
          Erro na requisição!
        </v-snackbar>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "ListPage",

  data() {
    return {
      itemsPerPageArray: [4, 8, 12],
      search: "",
      filter: {},
      sortDesc: false,
      page: 1,
      itemsPerPage: 8,
      sortBy: "name",
      items: [],
      snackbarError: false,
      favorites: [],
      loader: null,
      dogImage: [],
      dogSubImage: {},
    };
  },

  computed: {
    numberOfPages() {
      return Math.ceil(this.items.length / this.itemsPerPage);
    },

    cachorrosDaPaginaAtual() {
      const indexInicial = (this.page - 1) * this.itemsPerPage;
      const indexFinal = indexInicial + this.itemsPerPage;

      return this.items.slice(indexInicial, indexFinal);
    },

    subRacasDaPaginaAtual() {
      return this.cachorrosDaPaginaAtual;
    },
  },

  watch: {
    page() {
      this.listaDeImagensCachorrosPaginaAtual();
      this.retornaListaSubRacaCachorro();
    },
  },

  mounted() {
    this.init();
  },

  methods: {
    async init() {
      await this.getData();
      this.getFavorite();
      this.listaDeImagensCachorrosPaginaAtual();
      this.retornaListaSubRacaCachorro();
    },

    async getData() {
      this.loader = true;

      try {
        let response = await this.$http.get(
          "https://dog.ceo/api/breeds/list/all"
        );

        this.items = Object.keys(response.data.message).map((breed) => ({
          breed,
          subBreed: response.data.message[breed],
        }));
      } catch (erro) {
        this.snackbarError = true;
      }

      this.loader = false;
    },

    async getImageBreed(breed) {
      try {
        let imagem = await this.$http.get(
          `https://dog.ceo/api/breed/${breed}/images/random`
        );

        return imagem.data.message;
      } catch (erro) {
        this.snackbarError = true;
      }

      return null;
    },

    async getImageSubBreed(breed, subBreed) {
      try {
        let imagemSubDog = await this.$http.get(
          `https://dog.ceo/api/breed/${breed}/${subBreed}/images/random`
        );

        return imagemSubDog.data.message;
      } catch (erro) {
        this.snackbarError = true;
      }

      return null;
    },

    async listaDeImagensCachorrosPaginaAtual() {
      this.dogImage = [];
      for (const imagem of this.cachorrosDaPaginaAtual) {
        this.dogImage.push(await this.getImageBreed(imagem.breed));
      }
    },

    retornaListaSubRacaCachorro() {
      for (const listSubRacas of this.subRacasDaPaginaAtual) {
        if (listSubRacas.subBreed.length) {
          listSubRacas.subBreed.forEach((subRaca) => {
            this.listaDeImagensSubRacasCachorrosPaginaAtual(
              listSubRacas.breed,
              subRaca
            );
          });
        }
      }
    },

    async listaDeImagensSubRacasCachorrosPaginaAtual(breed, subBreed) {
      if (!this.dogSubImage[subBreed]) {
        const imgSubBreed = await this.getImageSubBreed(breed, subBreed);

        this.dogSubImage[subBreed] = imgSubBreed;
      }
    },

    getFavorite() {
      if (localStorage.getItem("favorite_breed")) {
        this.favorites = JSON.parse(localStorage.getItem("favorite_breed"));
      }
    },

    saveFavorite(raca) {
      if (this.favorites.includes(raca)) {
        this.favorites.splice(raca, 1);
      } else {
        this.favorites.push(raca);
      }

      localStorage.setItem("favorite_breed", JSON.stringify(this.favorites));
    },

    nextPage() {
      if (this.page + 1 <= this.numberOfPages) this.page += 1;
    },

    formerPage() {
      if (this.page - 1 >= 1) this.page -= 1;
    },

    updateItemsPerPage(number) {
      this.itemsPerPage = number;
    },
  },
};
</script>
<style scoped>
@media (max-width: 600px) {
  .responsive-row {
    width: 68%;
  }
}
</style>
