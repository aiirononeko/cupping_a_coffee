<template>
  <div class="container">
    <div>
      <h2 class="title mt-10">アカウントを作成しましょう</h2>
    </div>
    
    <form class="w-full max-w-sm my-8">
      <div class="md:flex md:items-center mb-6">
        <div class="md:w-1/3">
          <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4" for="name">
            Name
          </label>
        </div>
        <div class="md:w-2/3">
          <input v-model="$v.name.$model" placeholder="Taro" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="name" type="text">
        </div>
      </div>
      <div class="md:flex md:items-center mb-6">
        <div class="md:w-1/3">
          <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4" for="email">
            Email
          </label>
        </div>
        <div class="md:w-2/3">
          <input v-model="$v.email.$model" placeholder="taro@example.com" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="email" type="text">
        </div>
      </div>
      <div class="md:flex md:items-center mb-6">
        <div class="md:w-1/3">
          <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4" for="inline-username">
            Password
          </label>
        </div>
        <div class="md:w-2/3">
          <input v-model="$v.password.$model" class="bg-gray-200 appearance-none border-2 border-gray-200 rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-purple-500" id="inline-username" type="password" placeholder="******************">
        </div>
      </div>
      <div class="my-8">
        <div>
          <ul v-if="errors.length !== 0" class="mb-8">
            <li v-for="(err, i) in errors" :key="i" class="text-red-600">{{ err }}</li>
          </ul>
          <button @click="signUpWithEmail" class="shadow bg-purple-500 hover:bg-purple-400 focus:shadow-outline focus:outline-none text-white font-bold py-2 px-4 rounded" type="button">
            アカウントを作成する
          </button>
        </div>
      </div>
    </form>

    <p>------------------------------------------</p>

    <button @click="signUpWithTwitter" class="mt-10 mb-5 shadow focus:shadow-outline focus:outline-none bg-blue-400 hover:bg-blue-500 text-gray-100 font-bold py-2 px-10">Twitterアカウントで登録する</button>
    <button @click="signUpWithFacebook" class="mb-10 shadow focus:shadow-outline focus:outline-none bg-indigo-400 hover:bg-indigo-500 text-gray-100 font-bold py-2 px-8">Facebookアカウントで登録する</button>

    <nuxt-link to="/">トップページに戻る</nuxt-link>
  </div>
</template>

<script>
import firebase from '~/plugins/firebase'
import { twProvider, fbProvider, emCredential, twCredential, fbCredential} from '~/plugins/firebase'
import { required, email, maxLength } from 'vuelidate/lib/validators'

export default {
  data() {
    return {
      name: '',
      email: '',
      password: '',
      errors: [],
      anonymous: false,
      uid: '',
    }
  },
  validations: {
    name: {
      required,
      maxLength: maxLength(30)
    },
    email: {
      required,
      email
    },
    password: {
      required
    }
  },
  beforeMount() {
    firebase.auth().onAuthStateChanged(user => {
      this.anonymous = user.isAnonymous
      this.uid = user.uid

      if (user && user.isAnonymous) {
        const db = firebase.firestore()
        const usersRef = db.collection('users').doc(user.uid)
        usersRef.get().then(doc => {
          this.name = doc.data().name
        })
      }
      if (user && !user.isAnonymous) {
        this.$router.push(`/users/${user.uid}`)
      }
    })
  },
  methods: {
    async signUpWithEmail() {
      this.errors = []
      !this.$v.name.required && this.errors.push('名前を入力してください')
      !this.$v.name.maxLength && this.errors.push('名前は30文字以内で入力してください')
      !this.$v.email.required && this.errors.push('メールアドレスを入力してください')
      !this.$v.email.email && this.errors.push('メールアドレスの形式で入力してください')
      !this.$v.password.required && this.errors.push('パスワードを入力してください')

      this.$v.$touch() // バリデーションチェック
      if (this.$v.$invalid) return

      if (!this.anonymous) {
        await firebase.auth().createUserWithEmailAndPassword(this.email, this.password).then(async result => {
          const userInfo = {
            uid: result.user.uid,
            name: this.name
          }
          const db = firebase.firestore()
          const usersRef = db.collection('users').doc(result.user.uid)
          await usersRef.set(userInfo).then(res => {
            this.$router.push(`/users/${result.user.uid}`)
          }).catch(err => {
            console.log(err.message)
          })
        }).catch(err => {
          console.log(err.message)
        })
      } else { // 匿名ユーザーを永久アカウントにアップデートする処理
        const credential = emCredential.credential(this.email, this.password)
        await firebase.auth().currentUser.linkWithCredential(credential).then(async result => {
          const userInfo = {
            uid: result.user.uid,
            name: this.name
          }
          const db = firebase.firestore()
          const usersRef = db.collection('users').doc(result.user.uid)
          await usersRef.set(userInfo).then(res => {
            this.$router.push(`/users/${result.user.uid}`)
          }).catch(err => {
            console.log(err.message)
          })
        }).catch(err => {
          console.log(err.message)
        })
      }
    },
    signUpWithTwitter() {
      firebase.auth().signInWithPopup(twProvider).then(result => {
        const user = result.user

        const db = firebase.firestore()
        const usersRef = db.collection('users').doc(result.user.uid)
        usersRef.set({
          uid: result.user.uid,
          name: result.user.displayName,
        }).then(res => {
          this.$router.push(`/users/${result.user.uid}`)
        }).catch(err => {
          console.log(err.message)
        })
      }).catch(err => {
        console.log(err.message)
      })
    },
    signUpWithFacebook() {
      firebase.auth().signInWithPopup(fbProvider).then(result => {
        const user = result.user

        const db = firebase.firestore()
        const usersRef = db.collection('users').doc(result.user.uid)
        usersRef.set({
          uid: result.user.uid,
          name: result.user.displayName,
        }).then(res => {
          this.$router.push(`/users/${result.user.uid}`)
        }).catch(err => {
          console.log(err.message)
        })
      }).catch(err => {
        console.log(err.message)
      })
    },
  }
}
</script>

<style scoped>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 40px;
  color: #35495e;
  letter-spacing: 1px;
}

form {
  margin-top: 80px;
}

a {
  margin-top: 10px;
}

a:hover {
  color: #3490dc;
}

/** 
 * タブレット用ブレークポイント
 */
@media screen and (max-width: 1179px) {
  .title {
    font-size: 40px;
  }

  .user_form_items {
    width: 30%;
  }
}

/**
 * スマホ用ブレークポイント
 */
@media screen and (max-width: 767px) {
  .title {
    font-size: 25px;
  }

  .user_form_items {
    width: 60%;
  }
}
</style>

