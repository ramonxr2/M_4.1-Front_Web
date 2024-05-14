<template>
    <v-data-table
      :headers="headers"
      :items="usuarios"
      :sort-by="[{ key: 'nombre', order: 'asc' }]"
    >
  
      <template v-slot:top>
        <v-toolbar
          flat
        >
          <v-toolbar-title>Tabla de usuarios</v-toolbar-title>
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
                        v-model="editedItem.nombre"
                        label="Nombre"
                      ></v-text-field>
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
              <v-card-title class="text-h5">¿Desea eliminar este usuario?</v-card-title>
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
  import axios from 'axios';
  
  export default {
    data() {
      return {
        dialog: false,
        dialogDelete: false,
        headers: [
          { title: 'Nombre', key: 'nombre', align: 'start', sortable: true },
          { title: 'Password', key: 'password', sortable: true },
          { title: 'Acciones', key: 'actions', sortable: false },
        ],
        usuarios: [],
        editedIndex: -1,
        editedItem: {
          nombre: '',
          password: ''
        },
        defaultItem: {
            nombre: '',
          password: ''
        },
      };
    },
    computed: {
      formTitle() {
        return this.editedIndex === -1 ? 'Nuevo usuario' : 'Editar usuario';
      },
    },
    watch: {
      dialog(val) {
        val || this.close();
      },
      dialogDelete(val) {
        val || this.closeDelete();
      },
    },
    created() {
      this.initialize();
    },
    methods: {
      async fetchUsuarios() {
        try {
          const response = await axios.get('http://localhost:3000/api/usuarios');
          this.usuarios = response.data;
        } catch (error) {
          console.error('Error al obtener los usuarios:', error);
        }
      },
      initialize() {
        this.fetchUsuarios();
      },
      editItem(item) {
        this.editedIndex = this.usuarios.indexOf(item);
        this.editedItem = Object.assign({}, item);
        this.dialog = true;
      },
      deleteItem(item) {
        this.editedIndex = this.usuarios.indexOf(item);
        this.editedItem = Object.assign({}, item);
        this.dialogDelete = true;
      },
      deleteItemConfirm() {
        axios.delete(`http://localhost:3000/api/usuarios/${this.editedItem.id}`)
        .then(() => {
        this.usuarios.splice(this.editedIndex, 1);
        this.closeDelete();
        })
        .catch(error => {
        console.error('Error al eliminar usuario en el backend:', error);
      });
      },
      close() {
        this.dialog = false;
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem);
          this.editedIndex = -1;
        });
      },
      closeDelete() {
        this.dialogDelete = false;
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem);
          this.editedIndex = -1;
        });
      },
      save() {
        if (this.editedIndex > -1) {
           // Actualizar usuario en el backend
         axios.put(`http://localhost:3000/api/usuarios/${this.editedItem.id}`, this.editedItem)
        .then(() => {
          // Actualizar la usuario en la lista local
          this.usuarios[this.editedIndex] = Object.assign({}, this.editedItem);
          this.close();
        })
        .catch(error => {
          console.error('Error al actualizar usuario:', error);
        });
  
        } 
        else {
          // Crear un nuevo usuario en el backend
         axios.post('http://localhost:3000/api/usuarios', this.editedItem)
        .then(response => {
          // Agregar el nuevo usuario a la lista local
          this.usuarios.push(response.data);
          this.close();
        })
        .catch(error => {
          console.error('Error al crear nueva usuario:', error);
        });
        }
        this.close();
      },
    },
  };
  </script>