<template>
  <div id="app">
    <div id="threeButtons">
      <button
        @click="updateCurrentNameList('list')"
        class="btn"
        :style="{
          'background-color': currentListName === 'list' ? 'blue' : 'white',
          color: currentListName === 'list' ? 'white' : 'black',
        }"
      >
        Show all
      </button>
      <button
        @click="updateCurrentNameList('developer-list')"
        class="btn"
        :style="{
          'background-color':
            currentListName === 'developer-list' ? 'blue' : 'white',
          color: currentListName === 'developer-list' ? 'white' : 'black',
        }"
      >
        Show developers
      </button>
      <button
        @click="updateCurrentNameList('over-30')"
        class="btn"
        :style="{
          'background-color': currentListName === 'over-30' ? 'blue' : 'white',
          color: currentListName === 'over-30' ? 'white' : 'black',
        }"
      >
        Show over 30 year old only
      </button>
    </div>

    <br />
    <div>
      <li
        v-for="developer in filterList(mainList, currentListName)"
        :key="developer._id"
      >
        {{ developer.firstName }} - {{ developer.lastName }} -
        {{ developer.job }} - {{ developer.age }}
        <button @click="removePerson(developer._id)">x</button>
        <button @click="editPerson(developer)">Edit</button>
      </li>
    </div>

    {{ formMode }}

    <br /><br />
    <form @submit.prevent="sendData">
      <label for="name">Enter first name: </label>
      <input
        type="text"
        :value="formFirstNameAdded"
        @input="updateFormField($event, 'formFirstNameAdded')"
        name="firstname"
        id="firstname"
        required
      /><br />
      <label for="name">Enter last name: </label>
      <input
        type="text"
        :value="formLastNameAdded"
        @input="updateFormField($event, 'formLastNameAdded')"
        name="lastname"
        id="lastname"
        required
      /><br />
      <label for="Age">Enter Age: </label>
      <input
        type="number"
        :value="formAgeAdded"
        @input="updateFormField($event, 'formAgeAdded')"
        name="age"
        id="age"
        required
      />
      <br />
      <label for="Age">Enter Job: </label>
      <input
        type="text"
        :value="formJobAdded"
        @input="updateFormField($event, 'formJobAdded')"
        name="job"
        id="job"
        required
      />
      <br />
      <br />
      <span v-if="loadingMessage"> {{ loadingMessage }}</span>
      <br /><br />

      <button type="submit">Send Data</button>
    </form>
    <div></div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      apiUrl: "https://front-end-test-back-end.up.railway.app/api/",
      mainList: [],
      greaterThan30YearsOld: [],
      showGreaterThan30: false,
      filterDeveloperArray: [],
      showDevelopersFiltered: false,
      formFirstNameAdded: "",
      formLastNameAdded: "",
      formAgeAdded: "",
      formJobAdded: "",
      loadingMessage: "",
      currentListName: "list",
      selectedDeveloper: null,
    };
  },

  // computed properties for 'formMode', if this.selectedDeveloper contains a truthy value
  // then show 'edit mode' in label, else show 'create mode' label
  computed: {
    formMode() {
      return this.selectedDeveloper ? "edit mode" : "create mode";
    },
  },

  // async create to make sure the funtions inside this method execute when the page loads
  async created() {
    await this.getDataFromAPI();
    this.filterGreaterThan30Years();
    this.showDevelopers();
  },

  // getDataFromAPI method pulls data API using sync/awaits
  // added loading message as well
  methods: {
    async getDataFromAPI() {
      this.loadingMessage = "Loading...";

      try {
        const response = await fetch(this.apiUrl);
        const data = await response.json();
        this.mainList = data;
        this.loadingMessage = "";
      } catch (error) {
        this.loadingMessage = "Cannot retreive data";
        console.error(error);
      }

      console.log(this.mainList);
    },

    // sendData Method is reponsible for creating and sending the object (finalObj)
    // finalObj takes key and values from the data (input v-bind)
    async sendData() {
      const finalObj = {
        firstName: this.formFirstNameAdded,
        lastName: this.formLastNameAdded,
        age: Number(this.formAgeAdded),
        job: this.formJobAdded,
      };

      // message changes based on form mode (in this case creating since we are sending data)
      if (this.formMode === "create mode") {
        this.loadingMessage = "Creating...";

        // Await and call fetch with POST method attaching the JSON body object
        try {
          const response = await fetch(this.apiUrl, {
            method: "POST",
            mode: "cors",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(finalObj),
          });
          console.log(response)
          if(response.status >= 400){
            console.warn('call return 404')
            this.loadingMessage = 'cannot post data - http 404'
          }
        } catch (error) {
          console.error(error);
          this.loadingMessage = "Unable to create person...";
        }
      } else {
        this.loadingMessage = "Updating...";

        // Await fetch with PUT option to update form
        try {
          await fetch(this.apiUrl + this.selectedDeveloper._id, {
            method: "PUT",
            mode: "cors",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(finalObj),
          });
        } catch (error) {
          console.log(error);
        }
      }

      // helpers method executed after the fetch call
      this.clearForm();
      this.loadingMessage = "";

      // refresh list to make sure updated values are shown
      this.refreshList();
    },

    // updateFormField - needs review
    updateFormField(event, fieldName) {
      this[fieldName] = event.target.value;
    },

    // editPerson allows current person values to be assigned to data property
    editPerson(developer) {
      this.selectedDeveloper = developer;
      this.formFirstNameAdded = developer.firstName;
      this.formLastNameAdded = developer.lastName;
      this.formAgeAdded = developer.age;
      this.formJobAdded = developer.job;

      // console.log("sending data");
    },

    // removePerson method, accepts personID as argument, then appends URL + id to delete
    // the choosen person from list

    async removePerson(personId) {
      this.loadingMessage = "Deleting..";

      try {
        await fetch('https://front-end-test-back-end.up.railway.app' + personId, {
          method: "DELETE",
          mode: "cors",
        });
        await this.refreshList();
        this.loadingMessage = "";
      } catch (error) {
        console.error(error);
      }
    },

    // Helpers functions
    async refreshList() {
      await this.getDataFromAPI();
      this.filterGreaterThan30Years();
      this.showDevelopers();
    },

    updateCurrentNameList(updateValue) {
      this.currentListName = updateValue;
      console.log(this.currentListName);
    },

    filterList(list, listName) {
      if (listName === "list") {
        // returns entire list of persons 'show all'
        return this.mainList;
      } else if (listName === "developer-list") {
        // returns entire filtered developer list 'show developers'
        return this.mainList.filter((item) => item.job === "Developer");
      } else if (listName === "over-30") {
        // returns entire filtered over 30 list 'show over 30 only'
        return this.mainList.filter((item) => {
          return item.age > 30;
        });
      }
    },

    clearForm() {
      this.formFirstNameAdded = "";
      this.formLastNameAdded = "";
      this.formAgeAdded = "";
      this.formJobAdded = "";
    },

    filterGreaterThan30Years() {
      this.greaterThan30YearsOld = this.mainList.filter((item) => {
        return item.age > 30;
      });
    },
    showDevelopers() {
      this.filterDeveloperArray = this.mainList.filter(
        (item) => item.job === "Developer"
      );
    },
    showOutput() {
      this.showGreaterThan30 = !this.showGreaterThan30;
    },
    showDevs() {
      this.showDevelopersFiltered = !this.showDevelopersFiltered;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.btn {
  border: 1px solid green;
  padding: 8px 16px;
  border-radius: 5px;
  font-family: "Segoe UI";
  font-weight: bold;
}
</style>
