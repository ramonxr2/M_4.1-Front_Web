<template>
  <v-data-table
    :headers="headers"
    :items="activos"
    :sort-by="[{ key: 'numSerie', order: 'asc' }]"
  >

    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Tabla de activos</v-toolbar-title>
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
                      v-model="editedItem.numSerie"
                      label="Numero Serie"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    md="4"
                    sm="6"
                  >
                    <v-text-field
                      v-model="editedItem.numInventario"
                      label="Numero Inventario"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    md="4"
                    sm="6"
                  >
                    <v-text-field
                      v-model="editedItem.descripcion"
                      label="Descripcion"
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
            <v-card-title class="text-h5">¿Desea eliminar este activo?</v-card-title>
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
        { title: 'Número de Serie', key: 'numSerie', align: 'start', sortable: true },
        { title: 'Número de Inventario', key: 'numInventario', sortable: true },
        { title: 'Descripción', key: 'descripcion', sortable: true },
        { title: 'Acciones', key: 'actions', sortable: false },
      ],
      activos: [],
      editedIndex: -1,
      editedItem: {
        numSerie: '',
        numInventario: '',
        descripcion: '',
      },
      defaultItem: {
        numSerie: '',
        numInventario: '',
        descripcion: '',
      },
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Nuevo activo' : 'Editar activo';
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
    async fetchActivos() {
      try {
        const response = await axios.get('http://localhost:3000/api/activos');
        this.activos = response.data;
      } catch (error) {
        console.error('Error al obtener los activos:', error);
      }
    },
    initialize() {
      this.fetchActivos();
    },
    editItem(item) {
      this.editedIndex = this.activos.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    deleteItem(item) {
      this.editedIndex = this.activos.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },
    deleteItemConfirm() {
      axios.delete(`http://localhost:3000/api/activos/${this.editedItem.id}`)
      .then(() => {
      this.activos.splice(this.editedIndex, 1);
      this.closeDelete();
      })
      .catch(error => {
      console.error('Error al eliminar activo en el backend:', error);
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
         // Actualizar activo existente en el backend
       axios.put(`http://localhost:3000/api/activos/${this.editedItem.id}`, this.editedItem)
      .then(() => {
        // Actualizar el activo en la lista local
        this.activos[this.editedIndex] = Object.assign({}, this.editedItem);
        this.close();
      })
      .catch(error => {
        console.error('Error al actualizar activo:', error);
      });

      } 
      else {
        // Crear un nuevo activo en el backend
       axios.post('http://localhost:3000/api/activos', this.editedItem)
      .then(response => {
        // Agregar el nuevo activo a la lista local
        this.activos.push(response.data);
        this.close();
      })
      .catch(error => {
        console.error('Error al crear nuevo activo:', error);
      });
      }
      this.close();
    },
  },
};
</script>