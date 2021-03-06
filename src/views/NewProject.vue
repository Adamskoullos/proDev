<template>
  <div class="row">
    <form @submit.prevent="handleSubmit" class="col-12" :class="{light: light}">
      <h4 :class="{light: light}">Create new project</h4>
      <input type="text" placeholder="Project title" v-model="title" :class="{light: light}" required>
      <textarea placeholder="Description" v-model="description" :class="{light: light}" required></textarea>
      <div v-if="!image">
        <label :class="{light: light}" for="file-upload" class="file-upload-button">Add project image</label>
        <span>Required</span>
      </div>
      <label v-if="image" :class="{light: light}" for="file-upload" class="file-upload-button loaded">Image loaded</label>
      <input id="file-upload" type="file" @change="handleChange">
      <div class="error">{{ fileError }}</div>
      <button :class="{light: light}" v-if="!isPending">Add new project</button>
      <button :class="{light: light}" v-if="isPending" class="loading">Uploading...</button>
    </form>
  </div>
</template>

<script>
import { ref } from '@vue/reactivity'
import useStorage from '../composables/useStorage'
import useCollection from '../composables/useCollection'
import getUser from '../composables/getUser'
import { timestamp } from '../firebase/config'
import { useRouter } from 'vue-router'


export default {
  props: ['light'],
  setup(){
    const title = ref('')
    const description = ref('')
    const projectImage = ref(null)
    const fileTypes = ['image/png', 'image/jpeg']
    const fileError = ref(null)
    const isPending = ref(false)
    const image = ref(false)

    const { uploadImage, error, filePath, url } = useStorage()
    const { addDoc } = useCollection('projects')
    const { user } = getUser()
    const router = useRouter()

    const handleSubmit = async () => {
      if(!projectImage.value){
        fileError.value = 'Please select an image to load'
      }
      if(projectImage.value){
        isPending.value = true
        await uploadImage(projectImage.value)
        const res = await addDoc({
          title: title.value,
          description: description.value,
          userId: user.value.uid,
          userName: user.value.displayName,
          imageUrl: url.value,
          filePath: filePath.value,
          tasks: [],
          createdAt: timestamp()
        })
        if(!error.value){
          isPending.value = false
          router.push({ name: 'SingleProject', params: { id: res.id } })
        }
        projectImage.value = null
        title.value = ''
        description.value = ''
        fileError.value = null
        image.value = false
      } 
      
    }

    const handleChange = (e) => {
      const selected = e.target.files[0]
      if(selected && fileTypes.includes(selected.type)){
        image.value = true
        projectImage.value = selected
        fileError.value = null
      } else{
        image.value = false
        projectImage.value = null
        fileError.value = 'Please select an image file, jpeg or png'
      }
    }

    return { title, description, projectImage, fileError, handleSubmit, handleChange, error, isPending, image }
  }
}
</script>

<style scoped>
.row{
  flex:1;
  flex-direction: column;
  align-items: stretch;
  max-width: 95%;
}
form{
  margin-top: 10vh;
  width: 100%;
  max-width: 600px;
}
form.light{
  margin-top: 10vh;
  width: 100%;
  max-width: 600px;
}
h4.light{
  color: var(--primary)
}
input[type="file"]{
  display: none;
}
.file-upload-button{
  background: rgb(75, 75, 75);
  color:rgb(206, 206, 206);
  border-radius: 8px;
  border: 0;
  padding: 10px 12px;
  font-weight: 600;
  cursor: pointer;
  display: inline-block;
  margin: auto 5px;
  box-shadow: 1px 2px 6px rgba(50,50,50,0.5);
  border: 0px solid  var(--secondary);
  transition: all ease 0.2s;
}
.file-upload-button:hover{
  color: rgb(206, 206, 206);
  box-shadow: 1px 2px 6px rgba(50,50,50,0.3);
  transform: scale(0.94);
  transition: all ease 0.3s;
}
.loaded{
  color: rgb(45, 160, 0);
  box-shadow: 1px 2px 6px rgba(50,50,50,0.0);
}
.loaded:hover{
  cursor:default;
  transform: scale(1);
  color: rgb(45, 160, 0);
  box-shadow: 1px 2px 6px rgba(50,50,50,0.0);
}
button.light{
  background: white;
}
.file-upload-button.light{
  background: white;
  color: var(--primary);
}
.file-upload-button.light:hover{
  color: var(--primary);
}
</style>