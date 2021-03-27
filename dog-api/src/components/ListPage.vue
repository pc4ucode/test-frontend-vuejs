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

        <v-data-iterator
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
                <v-card>
                  <v-card-title class="subheading font-weight-bold">
                    {{ item.raca }}
                    <span>
                      <v-icon>mdi-star-outline</v-icon>
                    </span>
                  </v-card-title>

                  <v-divider></v-divider>

                  <v-list dense>
                    <v-list-item>
                      <v-list-item-content
                        class="align-end"
                      >
                        {{ item.subRaca || '-' }}
                      </v-list-item-content>
                    </v-list-item>
                  </v-list>
                </v-card>
              </v-col>
            </v-row>
          </template>

          <template v-slot:footer>
            <v-row class="mt-2" align="center" justify="center">
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
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "HelloWorld",

  data() {
    return {
      itemsPerPageArray: [4, 8, 12],
      search: "",
      filter: {},
      sortDesc: false,
      page: 1,
      itemsPerPage: 8,
      sortBy: "name",
      items: []
    };
  },

  computed: {
    numberOfPages() {
      return Math.ceil(this.items.length / this.itemsPerPage);
    }
  },

  mounted() {
    this.getData();
  },

  methods: {
    getData() {             
      this.$http.get("https://dog.ceo/api/breeds/list/all").then(
        (response) => {                     
          let dogs = response.data.message

          for(const key in dogs) {              
              if(dogs[key].length)   {
                dogs[key].forEach(dog => {
                  this.items.push({raca: `${key}`, subRaca: `${dog}`})                  
                })
              } else {
                this.items.push({'raca': key})
              }              
          }                    
        },
        (response) => {
          console.log(response.err)
        }
      );
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
