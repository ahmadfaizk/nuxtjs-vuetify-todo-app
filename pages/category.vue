<template>
  <div>
    <v-data-table :headers="headers" :items="items" :items-per-page="10" class="elevation-1">
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Kategori</v-toolbar-title>
          <v-divider class="mx-4" inset vertical />
          <v-spacer />
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                New Category
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
                        <v-text-field v-model="editedItem.name" :rules="nameRules" label="Nama Kategori" required />
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
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">
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
      headers: [
        { text: 'Nama Kategori', value: 'name' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      nameRules: [
        v => !!v || 'Nama harus diisi'
      ],
      items: [],
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      editedItem: {
        name: ''
      },
      defaultItem: {
        name: ''
      }
    }
  },
  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'Kategori Baru' : 'Edit Kategori'
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
  },
  methods: {
    editItem (item) {
      this.id = item._id
      this.editedIndex = this.items.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    deleteItem (item) {
      this.id = item._id
      this.editedIndex = this.items.indexOf(item)
      this.editedItem = Object.assign({}, item)
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
          this.updateCategory()
        } else {
          this.createCategory()
        }
        this.close()
      }
    },
    deleteItemConfirm () {
      this.deleteCategory()
    },
    async fetchData () {
      await axios.get('/api/category')
        .then((response) => {
          this.items = response.data.data
        })
    },
    async getCategory () {
      await axios.get('api/category/' + this.id).then((response) => {
        this.form = response.data.data
      })
    },
    async createCategory () {
      await axios.post('api/category', this.editedItem)
        .then((response) => {
          this.fetchData()
        })
    },
    async updateCategory () {
      await axios.put('api/category/' + this.id, this.editedItem)
        .then((response) => {
          this.fetchData()
        })
    },
    async deleteCategory () {
      await axios.delete('api/category/' + this.id)
        .then(() => {
          this.closeDelete()
          this.fetchData()
        })
    }
  }
}
</script>
