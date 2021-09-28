<template>
<div class="container-fluid">
	<div class="row">
				<div class="col-md-4">
          <b-form @submit.prevent="updateSortedItems" inline>
          <label class="mr-sm-2" >Сортировать по</label>

          <b-form-select v-model="sortform.selected_column" required
            id="inline-form-custom-select-pref"
            class="mb-2 mr-sm-2 mb-sm-0"
            :options="[
                {value: null, text: 'Выберите колонку'},
                {value: 'name', text: 'имени'},
                {value: 'count', text: 'количеству'},
                {value: 'distance', text: 'расстоянию'},
            ]"
            :value="sortform.selected_column"
          ></b-form-select>

            <div v-if="sortform.selected_column != null">
              <b-button  id="but" @click="updateSortedItems" type="submit">Отсортировать</b-button>
            </div>

            <div v-else>
              <b-button  disabled type="submit">Отсортировать</b-button>
            </div>
        </b-form>
			</div>

	</div>

  <div class="row">
     <div class="col-md-12">
       <b-form @submit.prevent="updateFilteredItems" inline>
         <label class="mr-sm-2" >Фильтр: </label>
       <b-form-select v-model="filterform.selected_column" required
            class="mb-2 mr-sm-2 mb-sm-0"
            :options="[
                {value: null, text: 'Выберите колонку'},
                {value: 'date', text: 'дата'},
                {value: 'name', text: 'имя'},
                {value: 'count', text: 'количество'},
                {value: 'distance', text: 'расстояние'},
            ]"
            :value="filterform.selected_column"
          ></b-form-select>

        <div v-if="filterform.selected_column === 'name'">
          <b-form-select v-model="filterform.selected_clause" required
            class="mb-2 mr-sm-2 mb-sm-0"
            :options="[
                {value: null, text: 'Выберите условие'},
                {value: 'equals', text: 'равно'},
                {value: 'contains', text: 'содержит'},
                {value: 'more', text: 'больше'},
                {value: 'less', text: 'меньше'},
            ]"
            :value="filterform.selected_clause"
          ></b-form-select>
        </div>

         <div v-else>
                  <b-form-select v-model="filterform.selected_clause" required
            class="mb-2 mr-sm-2 mb-sm-0"
            :options="[
                {value: null, text: 'Выберите условие'},
                {value: 'equals', text: 'равно'},
                {value: 'more', text: 'больше'},
                {value: 'less', text: 'меньше'},
            ]"
            :value="filterform.selected_clause"
          ></b-form-select>
         </div>

         <div  v-if="filterform.selected_column === 'count' || filterform.selected_column === 'distance'">
           <b-form-input required type="number" v-model="filterform.filter_value" placeholder="введите значение"></b-form-input>
         </div>

         <div v-else-if="filterform.selected_column === 'name'">
           <b-form-input required type="text" v-model="filterform.filter_value" placeholder="введите значение"></b-form-input>
         </div>

         <div v-else-if="filterform.selected_column === 'date'">
           <b-form-input required type="date" v-model="filterform.filter_value" placeholder="введите значение"></b-form-input>
         </div>
          <div v-if="filterform.selected_column != null && filterform.selected_clause != null && filterform.filter_value != null">
              <b-button  @click="updateFilteredItems" type="submit">Отфильтровать</b-button>
            </div>

            <div v-else>
              <b-button  disabled type="submit">Отфильтровать</b-button>
            </div>

      </b-form>

     </div>
  </div>

			<div class="row">
				<div class="col-md-8">
					<table class="table">
						<thead>
							<tr>
								<th>Дата</th>
								<th>Имя</th>
								<th>Количество</th>
								<th>Расстояние</th>
							</tr>
						</thead>
						<tbody>
            <tr v-for="item in displayedItems" >
          <td>{{ item.date }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.count }}</td>
          <td>{{ item.distance }}</td>
        </tr>


						</tbody>
					</table>
        </div>

				  <div class="col-md-4">
            <b-button  @click="getItems" type="submit" class="btn btn-warning">Сбросить настройки</b-button>
          </div>
      </div>

  <div class="row">
    <b-button-group class="mx-1">
          <b-button type="button"  v-if="page !== 1" @click="page--"> Назад </b-button>
    </b-button-group>

    <b-button-group class="mx-1">
          <b-button type="button"  v-for="pageNumber in pages.slice(page-1, page+5)" @click="page = pageNumber"> {{pageNumber}} </b-button>
    </b-button-group>

    <b-button-group  class="mx-1">
          <b-button type="button" @click="page++" v-if="page < pages.length" > Вперед </b-button>
    </b-button-group>
  </div>

</div>


</template>

<script>
import axios from 'axios'

const baseUrl = process.env.VUE_APP_API

export default {
  data() {
    return {
      items: [],
      filterform: {
        filter_value: null,
        selected_clause: null,
        selected_column: null,
        sendInfo: null,
        errors: [],
        submitStatus: null
      },
      sortform: {
        selected_column: null
      },
      page: 1,
      perPage: 5,
      pages: [],
    };
  },
computed: {
    displayedItems() {
      return this.paginate(this.items);
    }
  },
  watch: {
    items() {
      this.setPages();
    }
  },


  methods: {

    setPages() {
      this.page = 1;
      this.pages = [];
      let numberOfPages = Math.ceil(this.items.length / this.perPage);
      for (let index = 1; index <= numberOfPages; index++) {
        this.pages.push(index);
      }
    },
    paginate(items) {
      let page = this.page;
      let perPage = this.perPage;
      let from = (page * perPage) - perPage;
      let to = (page * perPage);
      return items.slice(from, to);
    },


  getItems() {
    const path = `${baseUrl}/items/`;
    axios
        .get(path)
        .then(res => (this.items = res.data))
        .catch((error) => {
          console.error(error);
        });

  },
  updateFilteredItems() {
    const path = `${baseUrl}/filter/`;
    const article = {
      sort_value: this.filterform.filter_value,
      selected_clause: this.filterform.selected_clause,
      selected_column: this.filterform.selected_column
    };

    axios
        .post(path, article)
        .then(res => (this.items = res.data))
        .catch((error) => {
          console.error(error);
        });
    this.reloadComponentForce()
  },

  updateSortedItems() {
    const path = `${baseUrl}/sort/`;
    const article = {selected_column: this.sortform.selected_column};
    axios
        .post(path, article)
        .then(res => (this.items = res.data))
        .catch((error) => {
          console.error(error);
        });
    this.reloadComponentForce()
  },

  reloadComponentForce() {
    this.$forceUpdate();

  },


},

  created() {
    this.getItems();

  }
};

</script>
