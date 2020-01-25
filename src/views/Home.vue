<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <label for="file">
      Upload
      <input type="file" @change="upload" id="file" style="display:none;" accept="image/*" capture="environment">
    </label>
    <p>Status: {{ message }}</p>
    <table>
        <thead>
          <tr>
            <th>Entities</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(label, index) in result" v-bind:key="label.id">
            <td>{{ index }} : {{ label.Name }} : {{ label.Confidence }}</td>
            <td v-if="label.Instances">
              <tr v-for="(instance, index) in label.Instances" v-bind:key="instance.id">
                <td>Instance{{ index }}: {{ instance.Confidence }}</td>
              </tr>
            </td>
          </tr>
        </tbody>
    </table>
  </div>
</template>

<script>
import { Storage } from 'aws-amplify';
import axios from 'axios'
export default {
  name: 'home',
  data () {
    return {
      message: "",
      filename: "",
      result: ""
    }
  },
  methods: {
    upload: async function (e) {
      var files = e.target.files
      var date = new Date().getTime()
      Storage.put(date + files[0].name, files[0])
      .then (result => {
        this.message = "uploaded:" + (this.filename = result['key'])
        this.analize()
      }).catch(err => this.message = err)
    },
    analize: async function () {
      const instance = axios.create({
        baseURL: 'https://kxnxujr7h0.execute-api.ap-northeast-1.amazonaws.com'
      })
      instance.post('/default/photoAnalize',{
        filename: this.filename
      },).then(response => {
        this.result = response.data.body.Labels
      }).catch(error => {
        this.result = error
      }).finally()
    }
  }
}
</script>
