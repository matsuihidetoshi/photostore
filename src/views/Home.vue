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
import { Storage } from 'aws-amplify'//画像アップロードに必要なライブラリ
import axios from 'axios'//API叩くため
export default {
  name: 'home',
  data () {
    return {
      message: "",//ただの表示用
      filename: "",//アップロードしたファイル名
      result: ""//Lambdaから帰ってきたRekognitionの判定結果入れる
    }
  },
  methods: {
    upload: async function (e) {//ファイルアップロードメソッド
      var files = e.target.files//ファイルオブジェクトを格納
      var date = new Date().getTime()//タイムスタンプ取得
      Storage.put(date + files[0].name, files[0])//タファイル名にイムスタンプくっ付けてファイルアップロード
      .then (result => {
        this.message = "uploaded:" + (this.filename = result['key'])//アップロード結果を表示用変数に格納
        this.analize()//Rekognition解析のLambdaのAPIを呼ぶ
      }).catch(err => this.message = err)//エラー処理
    },
    analize: async function () {//Rekognition解析のLambdaのAPIを呼ぶメソッド
      const instance = axios.create({//axiosインスタンス生成
        baseURL: 'https://kxnxujr7h0.execute-api.ap-northeast-1.amazonaws.com'//APIのベースURL設定
      })
      instance.post('/default/photoAnalize',{//APIのパスを設定
        filename: this.filename//S3にアップロードしたファイル名
      },).then(response => {
        this.result = response.data.body.Labels//解析結果をresultに格納
      }).catch(error => {
        this.result = error//エラー処理
      }).finally()//どのみちなんかやりたい場合はここに書く
    }
  }
}
</script>
