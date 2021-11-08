<template>
  <div>
    <v-card>
      <v-card-title class="justify-center">Github Search Webpage </v-card-title>
      <v-card-text>
        <v-container>
          <v-row>
            <v-select
              v-model="categoryModel"
              :items="searchBy"
              label="Select search category"
              filled
              item-text="name"
              return-object
            ></v-select
          ></v-row>
          <v-row>
            <v-combobox
              label="Search keyword"
              :items="suggestions"
              v-model="keyword"
              :loading="isLoading"
              :search-input.sync="search"
              placeholder="Start typing to Search"
              :item-text="categoryModel.text"
              append-icon=" "
              :error-messages="errorMsg"
            /> </v-row
          ><v-row
            ><v-spacer /><v-btn
              @click="
                repopulateTable = true;
                getRepositories();
              "
              :disabled="isLoading"
              >Populate</v-btn
            >
          </v-row>
        </v-container>
      </v-card-text>
    </v-card>
    <div class="pa-3" v-if="tableItems.length > 0">
      Displaying results for <b>{{ searchWord }}</b>
      <v-row>
        <v-col>
          <v-data-table
            :headers="headers"
            :items="tableItems"
            style="max-width: 100vw"
          >
            <template v-slot:[`item.actions`]="{ item }">
              <v-dialog v-model="dialog" scrollable
                ><template v-slot:activator="{ on, attrs }">
                  <v-icon small v-bind="attrs" v-on="on"
                    >mdi-information</v-icon
                  >
                </template>
                <v-card
                  ><v-card-text
                    ><v-container>{{ item }}</v-container></v-card-text
                  >
                  <v-divider></v-divider>
                  <v-card-actions
                    ><v-spacer></v-spacer
                    ><v-btn color="green darken-1" text @click="dialog = false"
                      >Close</v-btn
                    ></v-card-actions
                  >
                </v-card></v-dialog
              >
            </template>
          </v-data-table>
        </v-col>
        <v-col cols="2" v-if="this.categoryModel.name == 'repositories'"
          ><v-card class="mx-auto" max-width="500">
            <v-list>
              <v-list-item-group v-model="sortModel" mandatory>
                <v-list-item
                  v-for="(item, i) in sortItems"
                  :key="i"
                  :disabled="isLoading"
                >
                  <v-list-item-content>
                    <v-list-item-title v-text="item.text"></v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list> </v-card
        ></v-col>
      </v-row>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import { debounce } from "debounce";
export default {
  name: "SearchRepository",
  data: () => ({
    sortModel: 0,
    sortItems: [
      { text: "Best match" },
      {
        text: "Most stars",
        sortOrder: { sort: "stars", order: "desc" },
      },
      {
        text: "Fewest stars",
        sortOrder: { sort: "stars", order: "asc" },
      },
      {
        text: "Most forks",
        sortOrder: { sort: "forks", order: "desc" },
      },
      {
        text: "Fewest forks",
        sortOrder: { sort: "forks", order: "asc" },
      },
      {
        text: "Recently updated",
        sortOrder: { sort: "updated", order: "desc" },
      },
    ],
    errorMsg: null,
    dialog: false,
    searchWord: null, //display search word before table
    suggestions: [],
    suggestionWord: null,
    isLoading: false,
    keyword: null,
    search: null,
    tableItems: [],
    categoryModel: {
      name: "repositories",
      text: "name",
      headers: [
        { text: "Name", value: "name" },
        {
          text: "Url",
          value: "html_url",
        },
      ],
    },
    searchBy: [
      {
        name: "repositories",
        text: "name",
        headers: [
          { text: "Name", value: "full_name" },
          {
            text: "Url",
            value: "html_url",
          },
          {
            text: "Description",
            value: "description",
          },
        ],
      },
      {
        name: "code",
        text: "name",
        headers: [
          { text: "Name", value: "name" },
          {
            text: "Path",
            value: "path",
          },
          {
            text: "Url",
            value: "url",
          },
        ],
      },
      {
        name: "commits",
        text: "commit.message",
        headers: [
          { text: "Author", value: "commit.author.name" },
          { text: "Message", value: "commit.message" },
          {
            text: "Url",
            value: "url",
          },
        ],
      },
      {
        name: "issues",
        text: "title",
        headers: [
          { text: "Title", value: "title" },
          {
            text: "Url",
            value: "url",
          },
        ],
      },
      {
        name: "topics",
        text: "name",
        headers: [
          { text: "Name", value: "name" },
          {
            text: "Description",
            value: "short_description",
          },
        ],
      },
      {
        name: "users",
        text: "login",
        headers: [
          {
            text: "Login",
            value: "login",
          },
          {
            text: "Url",
            value: "url",
          },
        ],
      },
    ],
    repopulateTable: false,
  }),
  watch: {
    search: debounce(function (val) {
      //don't repopulate table, just provide suggestions
      this.getRepositories();
    }, 500),
    categoryModel() {
      console.log("category");
      this.suggestions = []; //clear suggestion
      this.tableItems = []; //clear table
      this.search = null; //clear keyword selection
      this.suggestionWord = null;

      this.getRepositories();
    },
    sortModel() {
      //to repopulate table
      this.repopulateTable = true;
      this.getRepositories();
    },
  },
  computed: {
    headers() {
      let tmp = JSON.parse(JSON.stringify(this.categoryModel.headers));
      switch (this.sortModel) {
        case 1:
        case 2: {
          tmp.push({ text: "Stars", value: "stargazers_count" });
          break;
        }
        case 3:
        case 4: {
          tmp.push({ text: "Forks", value: "forks_count" });
          break;
        }
        case 5:
        case 6: {
          tmp.push({ text: "Updated at", value: "updated_at" });
          break;
        }
      }
      tmp.push({ text: "Actions", value: "actions", sortable: false });
      return tmp;
    },
  },
  methods: {
    getRepositories() {
      return new Promise((resolve, reject) => {
        this.isLoading = true;
        if (this.errorMsg != null) this.errorMsg = null;
        let queryParams = {
          category: this.categoryModel.name,
          filterBy: this.search,
        };

        //only for "repositories" atm
        if (this.sortItems[this.sortModel].sortOrder != undefined) {
          queryParams["sortOrder.sort"] =
            this.sortItems[this.sortModel].sortOrder.sort;
          queryParams["sortOrder.order"] =
            this.sortItems[this.sortModel].sortOrder.order;
        }
        if (
          (this.search != null &&
            this.search.length > 0 &&
            //the word before this.search is not fetched yet
            this.search != this.suggestionWord) ||
          this.repopulateTable
        ) {
          console.log("searching " + this.search);
          this.suggestionWord = this.search;
          return axios
            .get("http://localhost:1234/search", {
              params: queryParams,
            })
            .then((res) => {
              this.suggestions = res.data.items;
              if (this.repopulateTable) {
                this.searchWord = this.search;
                this.tableItems = res.data.items;
              }
              this.repopulateTable = false;
              resolve(res);
            })
            .catch((err) => {
              if (err.response != null && err.response.status == 503)
                this.errorMsg = err.response.data;
              else this.errorMsg = err.toString();
              reject(err);
            })
            .finally(() => {
              this.isLoading = false;
            });
        } else {
          if (this.repopulateTable) {
            this.searchWord = this.search;
            this.tableItems = this.suggestions;
          }
          this.repopulateTable = false;
          this.isLoading = false;
          resolve("nth to search");
        }
      });
    },
  },
};
</script>
