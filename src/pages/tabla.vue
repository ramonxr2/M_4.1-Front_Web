<template>
  <v-data-table
    :headers="headers"
    :items="desserts"
    :sort-by="[{ key: 'calories', order: 'asc' }]"
  >
  
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Mi CRUD 2.2</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ props }">
            <v-btn
              class="mb-2"
              color="primary"
              dark
              v-bind="props"
            >
              Nuevo Item
            </v-btn>
          </template>
          
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    md="4"
                    sm="6"
                  >
                    <v-text-field
                      v-model="editedItem.name"
                      label="Autor"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    md="4"
                    sm="6"
                  >
                    <v-text-field
                      v-model="editedItem.titulo"
                      label="Titulo"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    md="4"
                    sm="6"
                  >
                    <v-text-field
                      v-model="editedItem.post"
                      label="Post"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    md="4"
                    sm="6"
                  >
                </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="close"
              >
                Cancel
              </v-btn>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
            <template v-slot:item.actions="{ item }">
  <v-icon
    class="me-2"
    size="small"
    @click="editItem(item)"
  >
    mdi-pencil
  </v-icon>
  <v-icon
    size="small"
    @click="deleteItem(item)"
  >
    mdi-delete
  </v-icon>
</template>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue-darken-1" variant="text" @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        class="me-2"
        size="small"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        size="small"
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="initialize"
      >
        Reset
      </v-btn>
    </template>
  </v-data-table>
</template>

<script>
  export default {
    data: () => ({
      dialog: false,
      dialogDelete: false,
      headers: [
  {
    title: 'Autor',
    align: 'start',
    sortable: false,
    key: 'name',
  },
  { title: 'Titulo', key: 'titulo' },  
  { title: 'Post', key: 'post' },     
  { title: 'Actions', key: 'actions', sortable: false },
],
      desserts: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        titulo: '',
        post: '',
      },
      defaultItem: { 
        name: '',
        titulo: '',
        post: '',
      },
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Nuevo item' : 'Editar Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    created () {
      this.initialize()
      this.fetchAuthors()
      this.fetchTitles();
      this.fetchPosts();
    },

    methods: {
      async fetchAuthors() {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        const data = await response.json();
        const authors = data.map(user => user.name);

        // Actualizar los nombres de autor en tu aplicación
        this.desserts.forEach((dessert, index) => {
          dessert.name = authors[index] || 'Unknown Author';
        });
      } catch (error) {
        console.error('Error al obtener los autores:', error);
      }
    },
    async fetchTitles() {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts');
        const data = await response.json();
        const titles = data.map(post => post.title);

        // Actualizar los títulos en tu aplicación
        this.desserts.forEach((dessert, index) => {
          dessert.titulo = titles[index] || 'Unknown Title';
        });
      } catch (error) {
        console.error('Error al obtener los títulos:', error);
      }
    },
    async fetchPosts() {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts');
        const data = await response.json();
        const posts = data.map(post => post.body);

        // Actualizar la información de los posts en tu aplicación
        this.desserts.forEach((dessert, index) => {
          dessert.post = posts[index] || 'No Content';
        });
      } catch (error) {
        console.error('Error al obtener la información de los posts:', error);
      }
    },
  
      initialize () {
        this.desserts = [
          {
            name: 'Frozen Yogurt',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Ice cream sandwich',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Eclair',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Cupcake',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Gingerbread',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Jelly bean',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Lollipop',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Honeycomb',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'Donut',
            titulo: 'A',
            post: 'A',
          },
          {
            name: 'KitKat',
            titulo: 'A',
            post: 'A',
          },
        ]
      },

      editItem (item) {
        this.editedIndex = this.desserts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.desserts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      deleteItemConfirm () {
        this.desserts.splice(this.editedIndex, 1)
        this.closeDelete()
      },

      close () {
        this.dialog = false
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
        if (this.editedIndex > -1) {
          Object.assign(this.desserts[this.editedIndex], this.editedItem)
        } else {
          this.desserts.push(this.editedItem)
        }
        this.close()
      },
      
    },
    
  }
</script>