<template>
  <div>
    <v-card>
      <v-card-title class="justify-center">Github Search Webpage </v-card-title>
      <v-card-text>
        <v-container>
          <v-row>
            <v-select
              v-model="searchByModel"
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
              v-model="model"
              :loading="isLoading"
              :search-input.sync="search"
              placeholder="Start typing to Search"
              :item-text="searchByModel.text"
              append-icon=" "
              :error-messages="errorMsg"
            /><v-btn class="mt-3 ml-3" @click="getRepositories()">Search</v-btn>
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
        <v-col cols="2"
          ><v-card class="mx-auto" max-width="500">
            <v-list>
              <v-list-item-group v-model="sortModel" mandatory>
                <v-list-item v-for="(item, i) in sortItems" :key="i">
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
    searchClicked: false,
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
      {
        text: "Least recently updated",
        sortOrder: { sort: "updated", order: "asc" },
      },
    ],
    errorMsg: null,
    dialog: false,
    searchWord: null,
    descriptionLimit: 256,
    suggestions: [],
    isLoading: false,
    model: null,
    search: null,
    tableItems: [],
    searchByModel: {
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
        text: "message",
        headers: [
          { text: "Author", value: JSON.stringify("commit.author") },
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
    keywordChanged: false,
    sortChanged: false,
  }),
  watch: {
    search: debounce(function (val) {
      if (val != null && val.length > 0 && !this.isLoading) {
        this.getRepositories();
      }
    }, 500),
    searchByModel() {
      this.suggestions = [];
      this.model = null;
    },
    model() {
      this.keywordChanged = true;
    },
    sortModel() {
      this.sortChanged = true;
      this.keywordChanged = true;
      this.getRepositories();
    },
  },
  computed: {
    headers() {
      let tmp = JSON.parse(JSON.stringify(this.searchByModel.headers));
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
        default:
          return tmp;
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
          category: this.searchByModel.name,
          filterBy: this.search,
        };

        if (this.sortItems[this.sortModel].sortOrder != undefined) {
          queryParams["sortOrder.sort"] =
            this.sortItems[this.sortModel].sortOrder.sort;
          queryParams["sortOrder.order"] =
            this.sortItems[this.sortModel].sortOrder.order;
        }
        if (this.searchWord != this.search || this.sortChanged) {
          console.log("searching " + this.search);
          return axios
            .get("http://localhost:1234/search", {
              params: queryParams,
            })
            .then((res) => {
              this.suggestions = res.data.items;
              if (this.keywordChanged) {
                //when user press "search" button or select dropdown list
                this.searchWord = this.search;
                this.tableItems = this.suggestions;
              }
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
              this.keywordChanged = false;
              this.sortChanged = false;
            });
        } else {
          //suggestions already populated the keyword
          this.searchWord = this.search;
          this.tableItems = this.suggestions;
          this.keywordChanged = false;
          this.isLoading = false;
          return resolve("already in item list");
        }
      });
    },
  },
};
</script>
