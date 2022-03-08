<template>
  <div>
    <v-card>
      <v-card-title class="justify-center">Github Search Webpage </v-card-title>
      <v-card-subtitle style="text-align: center"
        >A <u>single</u> autocomplete page written in VueJS based on
        <a href="https://developer.github.com/v3/search">Github Search API</a
        ><br />
        <p style="font-size: 10px">
          <i>
            Source code can be found in
            <a
              href="https://github.com/joey-peh/SEARCH_Frontend/tree/frontend_only"
              >https://github.com/joey-peh/SEARCH_Frontend/tree/frontend_only</a
            >
          </i>
        </p></v-card-subtitle
      >
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
          <!-- 
              @keydown.space="preventLeadingSpace" -->
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
              messages="Rate limit: 10 requests/min"
            />
            <v-btn
              class="ml-2 mt-3"
              @click="
                repopulateTable = true;
                getFromGithubAPI();
              "
              :disabled="isLoading || search == null"
              >Populate Results</v-btn
            ></v-row
          >
        </v-container>
      </v-card-text>
    </v-card>
    <div class="pa-3 ma-5" v-if="tableItems.length > 0">
      Displaying results for <b>{{ searchWord }}</b>
      <v-row>
        <v-col>
          <v-data-table
            :headers="headers"
            :items="tableItems"
            style="max-width: 100vw"
          >
            <template v-slot:[`item.actions`]="{ item }">
              <v-dialog
                v-model="information_dialog[item[categoryModel.id]]"
                scrollable
                :key="item.id"
                ><template v-slot:activator="{ on, attrs }">
                  <v-icon small v-bind="attrs" v-on="on"
                    >mdi-information</v-icon
                  >
                </template>
                <v-card>
                  <v-card-title class="text-h5 grey lighten-2">
                    {{ item[categoryModel.text] }}
                  </v-card-title>
                  <v-card-text
                    ><v-container>
                      <v-data-table
                        :headers="categoryModel.headers"
                        :items="[item]"
                        :disable-sort="true"
                        hide-default-footer
                        class="pb-5"
                      ></v-data-table>
                      <v-card>
                        <v-card-title>JSON Information</v-card-title>
                        <v-card-text>{{ item }}</v-card-text>
                      </v-card>
                    </v-container></v-card-text
                  >
                  <v-divider></v-divider>
                  <v-card-actions
                    ><v-spacer></v-spacer
                    ><v-btn
                      color="green darken-1"
                      text
                      @click.stop="
                        $set(information_dialog, item[categoryModel.id], false)
                      "
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
    information_dialog: {},
    searchWord: null, //display search word before table
    suggestions: [],
    suggestionWord: null,
    isLoading: false,
    keyword: null,
    search: null,
    tableItems: [],
    searchBy: [
      {
        name: "repositories",
        text: "name",
        id: "id",
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
      // {
      //   name: "code",
      //   text: "name",
      //   headers: [
      //     { text: "Name", value: "name" },
      //     {
      //       text: "Path",
      //       value: "path",
      //     },
      //     {
      //       text: "Url",
      //       value: "url",
      //     },
      //   ],
      // },
      {
        name: "commits",
        text: "commit.message",
        id: "url",
        headers: [
          { text: "Message", value: "commit.message" },
          { text: "Commiter", value: "commit.author.name" },
          {
            text: "Url",
            value: "url",
          },
        ],
      },
      {
        name: "issues",
        text: "title",
        id: "url",
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
        id: "name",
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
        id: "id",
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
    categoryModel: {
      name: "repositories",
      text: "name",
      id: "id",
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
    repopulateTable: false,
  }),
  watch: {
    $route: {
      immediate: true,
      handler(to, from) {
        document.title = "Git Search";
      },
    },
    search: debounce(function (val) {
      //don't repopulate table, just provide suggestions
      // this.getRepositories();
      if (val.trim() != "") this.getFromGithubAPI();
    }, 500),
    categoryModel() {
      this.suggestions = []; //clear suggestion
      this.tableItems = []; //clear table
      this.search = null; //clear keyword selection
      this.suggestionWord = null;
      this.sortModel = 0;

      // this.getRepositories();
      this.getFromGithubAPI();
    },
    sortModel() {
      //to repopulate table
      this.repopulateTable = true;
      // this.getRepositories();
      this.getFromGithubAPI();
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
      tmp.push({ text: "Information", value: "actions", sortable: false });
      return tmp;
    },
  },
  methods: {
    getFromGithubAPI() {
      this.errorMsg = null;
      const GITHUB_API_BASE_URL = "https://api.github.com/search/";
      let category = this.categoryModel.name;
      let query = "?q=" + this.search.trim();
      let sort = null;
      if (this.sortItems[this.sortModel].sortOrder != undefined) {
        let s = this.sortItems[this.sortModel].sortOrder.sort;
        let o = this.sortItems[this.sortModel].sortOrder.order;
        sort = "&sort=" + s + "&order=" + o;
      }

      let search_url = GITHUB_API_BASE_URL;
      if (category != null) search_url += category;
      if (query != null) search_url += query;
      if (sort != null) search_url += sort;

      return new Promise((resolve, reject) => {
        this.isLoading = true;
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
            .get(search_url)
            .then((res) => {
              if (res.data.items.length > 0) {
                this.suggestions = res.data.items;
              } else {
                this.errorMsg = "No matching items found for " + this.search;
              }
              if (this.repopulateTable) {
                this.searchWord = this.search;
                this.tableItems = res.data.items;
              }
              this.repopulateTable = false;
              resolve(res);
            })
            .catch((err) => {
              if (err.response.status == 403) {
                this.errorMsg = err.response.request.response;
              } else {
                this.errorMsg = err.toString();
              }
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

    getid(item) {
      return item.id ? item.id : item.url ? item.url : item.name;
    },
    // getRepositories() {
    //   //Spring Boot API
    //   return new Promise((resolve, reject) => {
    //     this.isLoading = true;
    //     if (this.errorMsg != null) this.errorMsg = null;
    //     let queryParams = {
    //       category: this.categoryModel.name, //code/commits/users..
    //       filterBy: this.search, //user=joey filterBy:joey
    //     };

    //     //only for "repositories" atm
    //     if (this.sortItems[this.sortModel].sortOrder != undefined) {
    //       queryParams["sortOrder.sort"] =
    //         this.sortItems[this.sortModel].sortOrder.sort;
    //       queryParams["sortOrder.order"] =
    //         this.sortItems[this.sortModel].sortOrder.order;
    //     }
    //     if (
    //       (this.search != null &&
    //         this.search.length > 0 &&
    //         //the word before this.search is not fetched yet
    //         this.search != this.suggestionWord) ||
    //       this.repopulateTable
    //     ) {
    //       console.log("searching " + this.search);
    //       this.suggestionWord = this.search;
    //       return axios
    //         .get("http://localhost:1234/search", {
    //           params: queryParams,
    //         })
    //         .then((res) => {
    //           this.suggestions = res.data.items;
    //           if (this.repopulateTable) {
    //             this.searchWord = this.search;
    //             this.tableItems = res.data.items;
    //           }
    //           this.repopulateTable = false;
    //           resolve(res);
    //         })
    //         .catch((err) => {
    //           if (err.response != null) this.errorMsg = err.response.data;
    //           else this.errorMsg = err.toString();
    //           reject(err);
    //         })
    //         .finally(() => {
    //           this.isLoading = false;
    //         });
    //     } else {
    //       if (this.repopulateTable) {
    //         this.searchWord = this.search;
    //         this.tableItems = this.suggestions;
    //       }
    //       this.repopulateTable = false;
    //       this.isLoading = false;
    //       resolve("nth to search");
    //     }
    //   });
    // },
  },
};
</script>
