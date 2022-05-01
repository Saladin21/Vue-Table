<template>
  <div class="table-responsive">
    <table class="table table-bordered">
      <thead>
        <tr>
          <template  v-for="c in columns" :key="c">
          <th
            v-if="!c.hidden"
            @click="sort(c)"
            class="text-blue clickable"
          >
            {{ c.name}}
          </th>
          </template>
          <th v-if="this.edit || this.delete" class="text-blue">Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="e in this.entries.slice(
            (this.page - 1) * this.entry_per_page,
            this.page * this.entry_per_page
          )"
          :key="e.id"
        >
            <template v-for="c in columns" :key="c">
              <template v-if="!c.hidden">
                <td v-if="c.clickable" @click="this.$emit('detail-entry', e[columns.Id.data])" class="clickable table-click">
                    {{ e[c.data] }}
                </td>
                <td v-else >
                    {{ e[c.data]}}
                </td>
              </template>
            </template>
          <template v-if="this.edit || this.delete"> 
          <td style="text-align: center">
            <button v-if="this.edit" @click="this.$emit('edit-entry', e[columns.Id.data])" class="btn btn-blue"> <i class="icon ion-android-create" style="font-size: 20px"></i> Edit </button>
            <button v-if="this.delete" @click="this.$emit('delete-entry', e[columns.Id.data])" class="btn-red"> <i class="icon ion-ios-trash-outline" style="font-size: 23px"></i> </button>
          </td>
          </template>
        </tr>
      </tbody>
    </table>
    <nav aria-label="Page navigation">
      <ul class="pagination justify-content-end clickable">
        <li v-if="this.page > 1" class="page-item">
          <div class="page-link" @click="prevPage()">Previous</div>
        </li>
        <li v-else class="page-item disabled">
          <div class="page-link">Previous</div>
        </li>
        <template v-for="p in this.pagination" :key="p">
          <li v-if="this.page == p" class="page-item disabled">
            <div class="page-link">{{ p }}</div>
          </li>
          <li v-else class="page-item">
            <div class="page-link" @click="goPage(p)">{{ p }}</div>
          </li>
        </template>
        <li v-if="this.page < this.page_num" class="page-item">
          <div class="page-link" @click="nextPage()">Next</div>
        </li>
        <li v-else class="page-item disabled">
          <div class="page-link">Next</div>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
export default {
  name: "Table",
  props: {
    data: Array,
    //map nama kolom ke data
    columns: Object,
    filter: String,
    edit: {
      type: Boolean,
      default: true
    },
    delete: {
      type: Boolean,
      default: true
    },
  },
  data() {
    return {
      pagination: [],
      sorted: {
        column: "",
        asc: true,
      },
      entry_per_page: 5,
      page: 1,
      page_num: 0,
    };
  },
  computed: {
    entries: function () {
      let entries;
      if (this.filter != "") {
        entries = this.data.filter(this.filterEntry);
      } else {
        entries = this.data;
      }

      this.changes(entries);
      
      return entries;
    },
  },
  methods: {
    changes(entries) {
      this.page_num = Math.ceil(entries.length / this.entry_per_page);
      this.pagination = [];
      for (let i = 1; i <= 3; i++) {
        if (i > this.page_num) {
          break;
        }
        this.pagination.push(i);
      }
    },
    sort(column) {
      if (column === this.sorted.column) {
        this.entries.reverse();
        this.sorted.asc = !this.sorted.asc;
      } else {
        var col = this.columns[column];
        this.entries.sort(function (a, b) {
          let x = a[col];
          let y = b[col];
          if (x < y) {
            return -1;
          }
          if (x > y) {
            return 1;
          }
          return 0;
        });
        this.sorted.column = column;
      }
      this.goPage(1);
    },
    nextPage() {
      if (this.page < this.page_num) {
        this.page += 1;
        this.updatePagination();
      }
    },
    prevPage() {
      if (this.page > 1) {
        this.page -= 1;
        this.updatePagination();
      }
    },
    goPage(i) {
      this.page = i;
      this.updatePagination();
    },
    updatePagination() {
      this.pagination = [];
      for (let i = this.page - 2; i <= this.page + 2; i++) {
        if (i <= this.page_num && i > 0) {
          this.pagination.push(i);
        }
      }
    },
    /* eslint-disable no-unused-vars */
    filterEntry(entry) {
      for (const [key, value] of Object.entries(this.columns)) {
        if (entry[value.data].toLowerCase().includes(this.filter)) {
          return true;
        }
      }
      return false;
    },
    /* eslint-enable no-unused-vars */
  },
};
</script>

<style scoped>
li {
  padding: 0;
}

.table-bordered td,
.table-bordered th,
.table-bordered thead {
  border: 1px solid #6992b4 !important;
}

.btn-blue {
  background-color: #6992b4;
  color: #f1f7fc;
  border-radius: 15px;
  padding: 5px;
  margin-right: 5px;
  vertical-align:middle
}

.btn-red {
  background-color: #f4476b;
  color: #f1f7fc;
  border-radius: 8px;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align:middle
}

th{
    text-align: center;
}
</style>
