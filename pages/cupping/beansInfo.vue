<template>
  <div class="container">
    <h2 class="title my-10">最後に、カッピングした豆の情報を教えてください</h2>
    <div class="beans_infomations mb-10">
      <form class="w-full max-w-sm my-10">
        <div class="flex items-center border-b border-b-2 border-teal-500 py-2 mb-5">
          <input v-model="country" class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none" type="text" placeholder="生産国">
        </div>
        <div class="flex items-center border-b border-b-2 border-teal-500 py-2 mb-5">
          <input v-model="farmer" class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none" type="text" placeholder="農園・生産者">
        </div>
        <div class="flex items-center border-b border-b-2 border-teal-500 py-2 mb-5">
          <input v-model="process" class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none" type="text" placeholder="プロセス">
        </div>
        <div class="flex items-center border-b border-b-2 border-teal-500 py-2 mb-5">
          <input v-model="elevation" class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none" type="number" placeholder="標高">
        </div>
        <div class="flex items-center border-b border-b-2 border-teal-500 py-2 mb-5">
          <input v-model="variety" class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none" type="text" placeholder="品種">
        </div>
      </form>
    </div>
    <button @click="dispathBeansInfo" class="shadow focus:shadow-outline focus:outline-none bg-blue-300 hover:bg-blue-500 text-gray-800 font-bold py-2 px-10 rounded-l start_button mt-5 end_button">カッピングを終了する</button>
    <p class="info my-10">6 / 6</p>
  </div>
</template>

<script>
import firebase from '~/plugins/firebase'

export default {
  data() {
    return {
      uid: '',
      cupped: '',
      country: '',
      farmer: '',
      elevation: '',
      process: '',
      variety: '',
    }
  },
  mounted() {
    this.uid = firebase.auth().currentUser.uid

    /**
     * ユーザーネーム取得.
     */
    const usersRef = firebase.firestore().collection('users').doc(this.uid)
    usersRef.get().then(doc => {
      if (doc) {
        this.cupped = doc.data().name
      }
    })
  },
  methods: {
    dispathBeansInfo() {
      this.$store.commit('cuppingResult/setUid', this.uid)
      this.$store.commit('cuppingResult/setCupped', this.cupped)
      this.$store.commit('cuppingResult/setCountry', this.country)
      this.$store.commit('cuppingResult/setFarmer', this.farmer)
      this.$store.commit('cuppingResult/setElevation', Number(this.elevation))
      this.$store.commit('cuppingResult/setProcess', this.process)
      this.$store.commit('cuppingResult/setVariety', this.variety)

      const db = firebase.firestore()
      const coffeeRef = db.collection('coffee')
      coffeeRef.add(this.$store.state.cuppingResult).then(res => {
        const resultData = {
          result_id: res.id,
          uid: this.uid
        }
        const cuppingResultsRef
          = db.collection('users').doc(this.uid).collection('cupping_results').doc(res.id)
        cuppingResultsRef.set(resultData) // カッピングリザルトをusersコレクションのサブコレクションとして保存

        this.$router.push(`/coffee/${res.id}`)
      }).catch(err => {
        console.log(err.message)
      })
    }
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
  width: 30rem;
}

.info {
  font-size: 180%;
}

.end_button {
  width: 30rem;
  height: 4rem;
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
