<style scoped>
.img-preview {
    width: 240px;
    height: 200px;
    object-fit: cover;
}

.img-gallery {
    width: 240px;
    height: 200px;
    object-fit: cover;
}
</style>

<template>
    <div class="container">

        <h2 class="fw-bold text-center text-md-start mt-4 mb-0">Image Gallery</h2>

        <hr class="mt-2 mb-5">

        <div class="row ">
            <div class="col-lg-4 col-md-6">
                <form autocomplete="off" class="text-lg-start">
                    <div class="form-group mb-3 text-center text-md-start">
                        <img class="img-fluid rounded-4 border border-1 border-secondary img-preview" id="img-preview" :src="form.previewUpload" alt="Preview">
                    </div>
                    <div class="form-group mb-3">
                        <div class="input-group">
                            <input type="file" name="avatar" id="avatar" class="form-control form-control-sm" :class="{ 'is-invalid': validation.avatar.length }" :disabled="isLoading" @change="handleInput($event)" ref="file" accept="image/*" aria-describedby="inputGroupFileAddon04" aria-label="Upload">
                            <button class="btn btn-outline-success btn-sm" type="button" id="inputGroupFileAddon04" @click="handleUpload()" :disabled="isLoading">
                                <span v-if="isLoading">Uploading...</span>
                                <span v-else>Upload</span>
                            </button>
                        </div>
                        <input type="hidden" :class="{ 'is-invalid': validation.avatar.length }">
                        <div v-if="validation.avatar" class="invalid-feedback mt-1">
                            <ul class="mb-0 ps-3">
                                <li v-for="(error, index) in validation.avatar">{{ error }}</li>
                            </ul>
                        </div>
                    </div>
                </form>
            </div>
        </div>

        <hr class="mt-2 mb-5">

        <div class="row text-center text-md-start">
            <div v-for="(gallery, index) in galleries" class="col-lg-3 col-md-4 col-6 mb-4" :key="gallery.thumb.name">
                <div>
                    <a :href="gallery.thumb.url" target="__blank" class="d-block h-100">
                        <img class="img-fluid img-thumbnail img-gallery" :src="gallery.thumb.url" :alt="gallery.thumb.name" style="object-fit: cover; width: 100%;">
                    </a>
                    <span>#{{ gallery.thumb.name }}</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue';
import axios from 'axios';

const mimeType = [
    'image/jpg',
    'image/jpeg',
    'image/bmp',
    'image/png',
    'image/gif'
];

const isLoading = ref(false);
let galleries = reactive(
    JSON.parse(localStorage.getItem('gallery')) || []
);

const form = reactive({
    avatar: "",
    previewUpload: "/img/vue+vite.png",
})

const validation = reactive({
    avatar: [],
})

function validateFile(file) {
    resetValidation('avatar', []);

    const { type = null, size = null } = file || {};
    const validate = [];

    if (!file) {
        validate.push('File tidak boleh kosong!');
    }

    if (!mimeType.includes(type)) {
        validate.push('Format file yang dibolehkan: jpg, jpeg, bmp, png, gif');
    }

    const fSize = Math.round((size / 1024));
    if (fSize >= 2048) {
        validate.push('Ukuran file maksimal 2Mb.');
    }

    if (validate.length) {
        validation.avatar = validate;
        return false;
    }

    return true;
}

function resetValidation(key, value) {
    validation[key] = value;
}

function handleInput(e) {
    try {
        const file = e.target.files[0];
        if (! validateFile(file)) {
            return;
        }
        
        form.avatar = file;
        form.previewUpload = URL.createObjectURL(file);
    } catch (error) {
        console.log(error);
    }
}

async function handleUpload() {
    if (! validateFile(form.avatar)) {
        alert('Input tidak valid.');
        return;
    }

    isLoading.value = true;
    const formData = new FormData();
    formData.append('image', form.avatar);

    const results = await axios.post(`https://api.imgbb.com/1/upload?key=75efdcc7b120f481b86f981be613ce5f`, formData, {
        headers: {
            'Content-Type': 'multipart/form-data'
        }
    })
    .then(({ data }) => data)
    .catch(({ response: { data } }) => data);

    isLoading.value = false;
    const { status, success, data, error = null, status_txt = 'Error!' } = results;
    if (error) {
        alert(`${status_txt} - ${error.message}`);
        return;
    }

    const currentGallery = localStorage.getItem('gallery');
    const galleryData = {
        image: data.image,
        thumb: data.thumb,
    };

    if (!currentGallery) {
        localStorage.setItem('gallery', JSON.stringify([galleryData]));
    } else {
        const newGallery = JSON.parse(currentGallery);
        newGallery.push(galleryData);

        localStorage.setItem('gallery', JSON.stringify(newGallery));
    }

    rendergallery();
    alert('Success upload!');
}

function rendergallery() {
    galleries = JSON.parse(localStorage.getItem('gallery')) || [];
}

</script>