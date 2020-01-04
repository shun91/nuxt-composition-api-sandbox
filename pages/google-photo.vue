<template>
  <v-layout column justify-center align-center>
    <h2>google photo</h2>
    <v-flex>
      <template v-if="me.name">
        こんにちは、{{ me.name }} <img :src="me.imageUrl" /> さん
        <v-btn @click="signout">Sign Out</v-btn>
        <v-btn @click="search">search</v-btn>
      </template>
      <template v-else>
        <v-btn @click="signIn">Sign In</v-btn>
      </template>
    </v-flex>
  </v-layout>
</template>

<script lang="ts">
import { createComponent, onMounted, reactive } from '@vue/composition-api'

const CLIENT_ID =
  '970395071555-024jl1sells0pji4pskt6rv3rq4lvkkp.apps.googleusercontent.com'

interface UseGapiAuth2Args {
  clientId: string
  scope: string
}

const useGapiAuth2 = ({ clientId, scope }: UseGapiAuth2Args) => {
  const me = reactive({ name: '', imageUrl: '' })

  const setMe = () => {
    const { isSignedIn, currentUser } = gapi.auth2.getAuthInstance()

    if (!isSignedIn.get()) {
      me.name = ''
      me.imageUrl = ''
      return
    }

    const profile = currentUser.get().getBasicProfile()
    me.name = profile.getName()
    me.imageUrl = profile.getImageUrl()
  }

  const signIn = () => {
    gapi.auth2.getAuthInstance().signIn({ scope })
  }

  const signout = () => {
    gapi.auth2.getAuthInstance().signOut()
  }

  onMounted(() => {
    gapi.load('client:auth2', () => {
      const { isSignedIn, currentUser } = gapi.auth2.init({
        client_id: clientId
      })
      isSignedIn.listen(setMe)
      currentUser.listen(setMe)
    })
  })

  return { me, signIn, signout }
}

const useGapiPhotos = () => {
  const API_VERSION = 'v1'
  const ALBUM_ID =
    'ANLDKGIz36CSFi_Q2aSEzpv1I6KZKeRL1I2xMQYnk3QJxtzTcuxx9kjZroO_z5icMzkBC52IpZmB'

  const search = async () => {
    const resp = await gapi.client.photoslibrary.mediaItems.search(
      {},
      { albumId: ALBUM_ID }
    )
    console.log('Response', resp)
  }

  onMounted(() => {
    gapi.load('client', () => {
      gapi.client.load(
        'https://content.googleapis.com/discovery/v1/apis/photoslibrary/v1/rest',
        API_VERSION
      )
    })
  })

  return { search }
}

export default createComponent({
  head: {
    script: [
      // npm package が存在しない
      // https://github.com/google/google-api-javascript-client/issues/432
      {
        type: 'text/javascript',
        src: 'https://apis.google.com/js/api.js',
        defer: true
      }
    ]
  },

  setup() {
    const { me, signIn, signout } = useGapiAuth2({
      clientId: CLIENT_ID,
      scope: 'https://www.googleapis.com/auth/photoslibrary.readonly'
    })

    const { search } = useGapiPhotos()

    return { me, signIn, signout, search }
  }
})
</script>
