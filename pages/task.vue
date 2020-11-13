<template>
  <div>
    <v-data-table :headers="headers" :items="items" :items-per-page="10" class="elevation-1">
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Task</v-toolbar-title>
          <v-divider class="mx-4" inset vertical />
          <v-spacer />
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                New Task
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-form ref="form" v-model="valid">
                    <v-row>
                      <v-col cols="12">
                        <v-text-field v-model="editedItem.name" label="Nama Task" :rules="[v => !!v || 'Nama harus diisi']" required />
                      </v-col>
                      <v-col cols="12">
                        <v-dialog
                          ref="dialog"
                          v-model="modal"
                          :return-value.sync="date"
                          persistent
                          width="290px"
                        >
                          <template v-slot:activator="{ on, attrs }">
                            <v-text-field
                              v-model="editedItem.date"
                              label="Pilih tanggal"
                              prepend-icon="mdi-calendar"
                              readonly
                              v-bind="attrs"
                              :rules="[v => !!v || 'Tanggal harus diisi']"
                              required
                              v-on="on"
                            />
                          </template>
                          <v-date-picker
                            v-model="editedItem.date"
                            scrollable
                          >
                            <v-spacer />
                            <v-btn
                              text
                              color="primary"
                              @click="modal = false"
                            >
                              Cancel
                            </v-btn>
                            <v-btn
                              text
                              color="primary"
                              @click="$refs.dialog.save(date)"
                            >
                              OK
                            </v-btn>
                          </v-date-picker>
                        </v-dialog>
                      </v-col>
                      <v-col cols="12">
                        <v-select
                          v-model="editedItem.category"
                          :items="categories"
                          item-text="name"
                          item-value="_id"
                          label="Kategori"
                          :rules="[v => !!v || 'Kategori harus dipilih']"
                          required
                        />
                      </v-col>
                      <v-col cols="12">
                        <v-textarea v-model="editedItem.description" label="Deskripsi" rows="2" :rules="[v => !!v || 'Deskripsi harus diisi']" required />
                      </v-col>
                      <v-col cols="12">
                        <v-checkbox v-if="editedIndex > -1" v-model="editedItem.done" label="Selesai ?" />
                      </v-col>
                    </v-row>
                  </v-form>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>
                <v-btn color="blue darken-1" :disabled="!valid" text @click="save">
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title>
                Apakah anda yakin akan menghapus ini ?
              </v-card-title>
              <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" text @click="closeDelete">
                  Cancel
                </v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">
                  OK
                </v-btn>
                <v-spacer />
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">
          mdi-pencil
        </v-icon>
        <v-icon small @click="deleteItem(item)">
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:[`item.date`]="{ item }">
        {{ dateFormatter(item.date) }}
      </template>
      <template v-slot:[`item.done`]="{ item }">
        {{ item.done ? 'Selesai' : 'Belum Selesai' }}
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="fetchData">
          Reset
        </v-btn>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      id: null,
      valid: false,
      modal: false,
      date: new Date().toISOString().substr(0, 10),
      headers: [
        { text: 'Nama Task', value: 'name' },
        { text: 'Kategori', value: 'category.name' },
        { text: 'Tanggal', value: 'date' },
        { text: 'Status', value: 'done' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      items: [],
      categories: [],
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      editedItem: {
        name: '',
        description: '',
        date: '',
        category: '',
        done: false
      },
      defaultItem: {
        name: '',
        description: '',
        date: '',
        category: '',
        done: false
      }
    }
  },
  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    }
  },
  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    }
  },
  created () {
    this.fetchData()
    this.getCategories()
  },
  methods: {
    editItem (item) {
      this.id = item._id
      this.editedIndex = this.items.indexOf(item)
      item.category = item.category._id
      item.date = item.date.substr(0, 10)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    deleteItem (item) {
      this.id = item._id
      this.dialogDelete = true
    },
    close () {
      this.dialog = false
      this.$refs.form.reset()
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    save () {
      this.$refs.form.validate()
      if (this.valid) {
        if (this.editedIndex > -1) {
          this.updateTask()
        } else {
          this.createTask()
        }
        this.close()
      }
    },
    deleteItemConfirm () {
      this.deleteTask()
    },
    async fetchData () {
      await axios.get('/api/task')
        .then((response) => {
          this.items = response.data.data
        })
    },
    async getCategories () {
      await axios.get('api/category')
        .then((response) => {
          this.categories = response.data.data
        })
    },
    async getCategory () {
      await axios.get('api/task/' + this.id).then((response) => {
        this.form = response.data.data
      })
    },
    async createTask () {
      await axios.post('api/task', this.editedItem)
        .then((response) => {
          this.fetchData()
        })
    },
    async updateTask () {
      await axios.put('api/task/' + this.id, this.editedItem)
        .then((response) => {
          this.fetchData()
        })
    },
    async deleteTask () {
      await axios.delete('api/task/' + this.id)
        .then(() => {
          this.closeDelete()
          this.fetchData()
        })
    },
    dateFormatter (date) {
      return new Date(date).toLocaleDateString('id', {
        weekday: 'long',
        day: 'numeric',
        month: 'short',
        year: 'numeric'
      })
    }
  }
}
</script>
