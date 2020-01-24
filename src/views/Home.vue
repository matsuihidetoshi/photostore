<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <label for="file">
      Upload
      <input type="file" @change="upload" id="file" style="display:none;">
    </label>
    <p>Status: {{ message }}</p>
  </div>
</template>

<script>
import { Storage } from 'aws-amplify';
export default {
  name: 'home',
  data () {
    return {
      message: ""
    }
  },
  methods: {
    upload: async function (e) {
      var files = e.target.files
      Storage.put(files[0].name, files[0])
      .then (result => this.message = "uploaded:" + result['key'])
      .catch(err => this.message = err)
    }
  }
}
</script>
