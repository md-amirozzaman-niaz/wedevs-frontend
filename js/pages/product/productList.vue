<template>
<v-app>
  <v-data-table
    :headers="headers"
    :items="products"
    sort-by="calories"
    class="elevation-1"
    mobile-breakpoint="0"
  >
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Product list</v-toolbar-title>
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
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              New product
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.title"
                      label="Title"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.description"
                      label="Description"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.price"
                      label="Price"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >

                    <v-file-input
                      v-model="editedItem.image"
                      label="Image"
                    ></v-file-input>
                  </v-col>
                 
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="secondary"
                @click="close"
              >
                Cancel
              </v-btn>
              <v-btn
                color="success"
                @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="secondary" @click="closeDelete">Cancel</v-btn>
              <v-btn color="error" @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.image="{ item }">
      <img :src='item.image ? item.image : "https://via.placeholder.com/60x30?text=No+Image"'/>
    </template>
    <template v-slot:item.actions="{ item }">
      <fa
        icon="edit" 
        @click="editItem(item)"
      >
        mdi-pencil
      </fa>
      <fa
        icon="trash-alt" 
        @click="deleteItem(item)"
      >
        mdi-delete
      </fa>
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
  <v-snackbar
      v-model="snackbar"
      :timeout="timeout"
    >
      {{ text }}

      <template v-slot:action="{ attrs }">
        <v-btn
          color="blue"
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
</v-app>
</template>


<script>
import axios from 'axios';
  export default {
      middleware: 'auth',

  metaInfo () {
    return { title: this.$t('product_list') }
  },
    data: () => ({
      dialog: false,
      dialogDelete: false,
      headers: [
        { text: 'ID', value: 'id' },
        {
          text: 'Product (name)',
          align: 'start',
          sortable: false,
          value: 'title',
        },
        { text: 'Description', value: 'description' },
        { text: 'Price', value: 'price' },
        { text: 'Image', value: 'image' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      products: [],
      editedIndex: -1,
      editedItem: {
        title: '',
        description: '',
        price: 0,
        image:[]
      },
      defaultItem: {
        title: '',
        description: '',
        price: 0,
        image:[]
      },
      snackbar:false,
      text: 'Deleted successfully.',
      timeout: 2000,
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
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
    },

    methods: {
      initialize () {
        axios.get('/api/product')
        .then((r)=>{
          this.products = r.data;
        })
        .catch(error => console.log(error.response.data.message)); 
      },

      editItem (item) {
        this.editedIndex = this.products.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.editedItem.image = []
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.products.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      deleteItemConfirm () {
        this.products.splice(this.editedIndex, 1)
        axios.delete('api/product/'+this.editedItem.id
        )
        .then(res=> {
          this.initialize()
          this.showSnackBar('Product deleted successfully')
          })
        .catch(err=>console.log(err.response.data.message))
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
        let data = this.prepareData();
        if (this.editedIndex > -1) {
            data.append("_method", 'PATCH');
            axios.post('api/product/'+this.editedItem.id,
            data
          )
          .then(res=> {
          this.initialize()
          this.showSnackBar('Product edited successfully')
          })
          .catch(err=>console.log(err.response.data.message))
        } else {
          axios.post('api/product',
          data
        )
        .then(res=> {
          this.initialize()
          this.showSnackBar('Product created successfully')
          })
        .catch(err=>console.log(err.response.data.message))
        }
        
        this.close()
      },

      prepareData(){
        let formData = new FormData();
        // this.editedItem.image = typeof(this.editedItem.image) == Object ?  this.editedItem.image : this.products[this.editedIndex].image;
        formData.append('title',this.editedItem.title);
        formData.append('description',this.editedItem.description);
        formData.append('price',this.editedItem.price);
        formData.append('image',this.editedItem.image);

        return formData;
      },
      showSnackBar(text){
        this.text = text;
        this.snackbar=true;
      }
    },
  }
</script>

<style>
img {
    width: auto;
    height: 40px;
}
</style>