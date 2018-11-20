<template>
  <div id="app">
    <Header :session="session" @login="popupLogin" @logout="logout" />
    <main>
      <section>
        <h2>Profile</h2>
        <p id="profileId-input-container">
          <label for="profile-input">Web ID:</label>
          <input v-model="profileId" id="profile-input">
          <button @click="loadProfile(profileId)">View</button>
        </p>
        <Profile :person="person"/>
      </section>
      <section>
        <h2>Friends</h2>
        <ul>
          <li v-for="friend in friends" :key="friend.id">
            <a @click="loadProfile(friend.id)">{{friend.name}}</a>
          </li>
        </ul>
      </section>
    </main>
  </div>
</template>

<script>
import auth from 'solid-auth-client'
import rdflib from 'rdflib'
import Header from './components/Header'
import Profile from './components/Profile'

// Set up a local data store and associated data fetcher
const FOAF = rdflib.Namespace('http://xmlns.com/foaf/0.1/')
const store = rdflib.graph()
const fetcher = new rdflib.Fetcher(store)

export default {
  name: 'app',
  components: {
    Header,
    Profile
  },
  data () {
    return {
      session: null,
      profileId: '',
      person: null,
      friends: []
    }
  },
  created () {
    auth.trackSession(session => {
      this.session = session
      if (session) {
        alert(`Logged in as ${this.session.webId}`)
        this.profileId = this.session.webId
      }
    })
  },
  methods: {
    async popupLogin () {
      let popupUri = 'https://solid.community/common/popup.html'
      try {
        await auth.popupLogin({ popupUri })
      } catch (err) {
        alert(err.message)
      }
    },
    async logout () {
      this.profileId = ''
      try {
        await auth.logout()
      } catch (err) {
        alert(err.message)
      }
    },
    async loadProfile (profileId) {
      // Load the person's data into the store
      this.person = null
      this.friends = []
      this.profileId = profileId
      try {
        await fetcher.load(profileId)
        this.person = this.getPerson(profileId)
        const friends = store.each(rdflib.sym(profileId), FOAF('knows')).slice(0,3)
        await Promise.all(friends.map(friend => fetcher.load(friend)))
        this.friends = friends.map(this.getPerson)
      } catch (err) {
        alert(err.message)
      }
    },
    getPerson (id, props = ['name']) {
      const symbol = store.sym(id)
      return props.reduce((acc, x) => {
        const prop = store.any(symbol, FOAF(x))
        if (prop) acc[x] = prop.value
        return acc
      }, { id })
    }
  }
}
</script>

<style>
  #profileId-input-container, main {
    display: flex;
    flex-flow: row wrap;
  }

  #profileId-input-container {
    align-items: center;
  }

  #profile-input {
    width: 90%;
    max-width: 400px;
  }

  main {
    padding: 16px;
  }

  main > section {
    flex-basis: 50%;
    min-width: 400px;
  }

  #profileId-input-container > * {
    margin: 8px 8px 8px 0;
  }
</style>
