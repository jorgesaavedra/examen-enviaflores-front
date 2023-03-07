<template>
  <main>
    <div class="home">
      <div class="home-info">
        <div class="row mb-4">
          <div class="col-12 d-flex justify-content-end">
            <button type="button" class="btn btn-primary btn-sm" @click="openAgregarModal()"><i class="bi bi-plus"></i> Agregar</button>
          </div>
        </div>
        <div class="row">
          <div class="col-4 mb-4" v-for="image in imagenes">
            <div class="card">
                <span v-if="image.is_favorite == 0" class="heart-icon" @click="actualizarFavorito(image)"><i class="bi bi-heart"></i></span>
                <span v-if="image.is_favorite == 1" class="heart-icon" @click="actualizarFavorito(image)"><i class="bi bi-heart-fill"></i></span>
                <div v-if="image.base64" class="card-img-top" :style="`background-image: url('${image.base64}');`"></div>
                <div v-else class="card-img-top" :style="`background-image: url('${image.base64}');`">{{ image.alt_text }}</div>
                <div class="card-body">
                  <h5 class="card-title">{{image.filename}}</h5>
                  <p class="card-text">descripcion</p>
                  <div class="d-flex justify-content-between">
                    <a href="#" class="btn btn-secondary" @click="openEditarModal(image)"><i class="bi bi-pencil"></i></a>
                    <a href="#" class="btn btn-danger" @click="eliminarImagen(image)"><i class="bi bi-trash"></i></a>
                  </div>
                </div>
            </div>
          </div>
        </div>

      </div>

    </div>

    <div class="modal fade modal-xl" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true" @hidden.bs.modal="">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">New message</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form>
              <div class="mb-3 row">
                <label for="file_name" class="col-sm-2 col-form-label">Nombre del archivo</label>
                <div class="col-sm-10">
                  <input type="text" class="form-control" :class="{ 'border-danger': v$.form.file_name.$error }" id="file_name" v-model="form.file_name">
                  <div v-if="v$.form.file_name.$error" class="text-danger">Campo Nombre del archivo requerido.</div>
                </div>
              </div>
              <div class="mb-3 row" v-if="!isEditar">
                <label for="file" class="col-sm-2 col-form-label">Archivo</label>
                <div class="col-sm-10">
                  <input type="file" class="form-control" :class="{ 'border-danger': v$.form.file.$error }" id="file" @change="handleImage" accept="image/*">
                  <div v-if="v$.form.file.$error" class="text-danger">Campo Archivo requerido.</div>
                </div>
              </div>
              <div class="mb-3 row">
                <label for="alt_text" class="col-sm-2 col-form-label">Texto alternativo</label>
                <div class="col-sm-10">
                  <input type="text" class="form-control" :class="{ 'border-danger': v$.form.alt_text.$error }" id="alt_text" v-model="form.alt_text">
                  <div v-if="v$.form.alt_text.$error" class="text-danger">Campo Texto alternativo requerido.</div>
                </div>
              </div>
              <div class="mb-3 row">
                <div class="col-12">
                  <div class="form-check">
                    <input class="form-check-input" type="checkbox" value="" id="is_favorite" v-model="form.is_favorite">
                    <label class="form-check-label" for="is_favorite">
                      Es Favorito?
                    </label>
                  </div>
                </div>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
            <button type="button" class="btn btn-primary" @click="editarImagen()" v-if="isEditar">Editar</button>
            <button type="button" class="btn btn-primary" @click="agregarImagen()" v-else>Agregar</button>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import axios from 'axios';
import { Modal } from "bootstrap";
import { useVuelidate } from '@vuelidate/core';
import { required, requiredIf } from '@vuelidate/validators';

export default {
  data() {
    return {
      v$: useVuelidate(),
      modal: null,
      imagenes: [],
      imagen: null,
      isEditar: false,
      fileRequired: false,
      form: {
        file_name: "",
        file: "",
        alt_text: "",
        is_favorite: false
      }
    }
  },
  validations () {
    return {
      form: {
        file_name: { required },
        file: {requiredIfFoo: requiredIf(this.fileRequired)},
        alt_text: { required }
      }
    }
  },
  mounted() {
    this.initPage();
    this.getImagenes();
  },
  methods: {
    initPage() {
      let _this = this;
      this.modal = new Modal(document.getElementById('exampleModal'), {});
      document.getElementById('exampleModal').addEventListener('hidden.bs.modal', function (event) {
        _this.onModalClose();
      });
    },
    getImagenes() {
      axios.get('http://localhost:8000/api/imagenes').then((response) => {
        console.log(response);
        this.imagenes = response.data;
      });
    },
    openAgregarModal(){
      this.isEditar = false;
      this.fileRequired = true;
      this.modal.show();
    },
    openEditarModal(image){
      this.isEditar = true;
      this.fileRequired = false;
      this.imagen = image;
      this.form.file_name = this.imagen.filename;
      this.form.alt_text = this.imagen.alt_text;
      this.form.is_favorite = this.imagen.is_favorite == 1 ? true : false;
      this.modal.show();
    },
    agregarImagen() {
      this.v$.$validate();
    
      if (this.v$.$error) return;
    
      axios.post(`http://localhost:8000/api/imagenes/add`, {filename: this.form.file_name, alt_text: this.form.alt_text, is_favorite: this.form.is_favorite, base64: this.form.file}).then((response) => {
        this.modal.hide();
        this.getImagenes();
      });
    
    },
    actualizarFavorito(imagen) {
      this.imagen = imagen;
      let favorito = this.imagen.is_favorite == 1 ? 0 : 1;
      this.imagen.is_favorite = favorito;
      this.actualizarImagen();
    },
    editarImagen() {
      this.v$.$validate();

      if (this.v$.$error) return;

      let favorito = this.form.is_favorite;
      this.imagen.is_favorite = favorito;
      this.imagen.filename = this.form.file_name;
      this.imagen.alt_text = this.form.alt_text;

      

      this.actualizarImagen();
    },
    actualizarImagen() {
      axios.post(`http://localhost:8000/api/imagenes/update/${this.imagen.id}`, {filename: this.imagen.filename, alt_text: this.imagen.alt_text, is_favorite: this.imagen.is_favorite, _method: 'patch'}).then((response) => {
        this.modal.hide();
      });
    
    },
    eliminarImagen(imagen) {
      this.imagen = imagen;
      axios.delete(`http://localhost:8000/api/imagenes/delete/${this.imagen.id}`).then((response) => {
        this.getImagenes();
      });
    
    },
    onModalClose(){
      this.file = "";
      this.form = {
        file_name: "",
        file: "",
        alt_text: "",
        is_favorite: false
      };
      this.v$.$reset();
    },
    handleImage(e) {
      const selectedImage = e.target.files[0];
      this.createBase64Image(selectedImage);
    },
    createBase64Image(fileObject) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.form.file = e.target.result;
      };
      reader.readAsDataURL(fileObject);
    }
  }
}    
</script>