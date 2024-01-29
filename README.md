# PiniaStore avec nuxt 3
## Table des matières

* [Introduction](#introduction)
* [Mise en place](#miseenplace)
* [Utilisation](#utilisation)

------------------
## <a id="introduction">Introduction</a>

Pinia constitue un système de gestion d'état dédié à Vue.js, offrant une approche simple et performante pour gérer l'état global d'une application.

Sa conception vise la simplicité, la performance, et l'exploitation optimale des fonctionnalités modernes de JavaScript et de Vue.js.

Actuellement, Pinia Store est compatible avec Vue 2 et Vue 3. 

L'API reste identique pour les deux versions, exception faite des aspects liés à l'installation et au rendu côté serveur (SSR - Server-Side Rendering).

pour ce tutoriel, nous allons mettre en place Pinia sur une applications nuxt 3.

L’utilisation de Pinia avec Nuxt est plus facile car Nuxt s’occupe de beaucoup de choses en ce qui concerne le rendu côté serveur.

Par exemple, vous n’avez pas besoin de vous soucier de la sérialisation ni des attaques XSS.

Pinia prend en charge Nuxt Bridge et Nuxt 3.

------------------
## <a id="miseenplace">Mise en place</a>

Vous avez deux possibilités pour l'installation des paquets : 
* soit avec Yarn :
```
yarn add pinia @pinia/nuxt
```
* Sinon avec npm :
```
npm install pinia @pinia/nuxt
```
Vous retrouverez dans le fichier package.json les dépendances envers Pinia.
```
"dependencies": {
    "@pinia/nuxt": "^0.5.1",
    "pinia": "^2.1.7"
  }
```
Pour finir, il reste à ajouter dans le fichier nuxt.config.js l'appel de Pinia dans les modules.
```
modules: [
    '@pinia/nuxt',
  ]
```
------------------
## <a id="utilisation">Utilisation</a>

Nous allons prendre l'exemple d'une application de jukebox digital pour créer un store et effectuer une recherche sur Deezer en utilisant une API.

Tout d'abord, il faut ajouter un dossier "store" puis créer le fichier.

![image](https://github.com/chantonin89240/PiniaStore/assets/49941533/69bad30c-0aa8-4b30-8dae-7b1df8f50adc)

Premièrement, nous devons initialiser la définition du store.
 ```
const runtimeConfig = useRuntimeConfig()
export const useCatalogStore = defineStore('catalogStore', {

}
```

Ensuite, il faut gérer la partie "état", qui constitue la partie centrale du store.

Dans Pinia, l'état est défini sous forme d'une fonction qui renvoie l'état initial. Cela permet à Pinia de fonctionner aussi bien côté serveur que côté client.

```
export const useCatalogStore = defineStore('catalogStore', {
  state: (): Provider => ({
        provider: null,
        statutAdd: 0
      }),
}
```

Après cela, il faut s'occuper des getters ou accesseurs.

Les accesseurs sont l'équivalent des valeurs calculées pour l'état d'un store.

Ils peuvent être définis à l'aide de la propriété.

Il s'agit des propriétés qui seront utilisées dans notre page.

```
getters: {
    resultProvider: (state) => state.provider,
    resultStatut: (state) => state.statutAdd,
  },
```

La dernière étape de notre store concerne les actions. Les actions sont l'équivalent des méthodes dans les composants.

```
actions: {
  async searchProvider(search: string) {
    const { data, pending, error, status } = await useFetch<Track[]>(runtimeConfig.public.apiBase + '/catalogs/search/' + search, {
      method: 'GET'
    })

    this.provider = data.value;
  },
}
```
Dans ce code, nous allons effectuer l'appel à notre recherche Deezer depuis une API, puis incrémenter l'état de provider.

Maintenant, il est temps d'utiliser notre store dans la page des catalogues.

Nous devons d'abord appeler et initialiser le store.
```
import { useCatalogStore } from '../../stores/CatalogStore';
const catalogStore = useCatalogStore();
```

Ensuite, il faut construire les propriétés.
```
const search = ref('')
const searchSong = ref('');
const songs = ref([]);
```

Enfin, il faut créer la méthode qui va utiliser le store pour effectuer la recherche Deezer.
```
const validateSearch = async () => {
  const searchData = {
    result: search.value
  }
  // appel de l'action du store catalog
  await catalogStore.searchProvider(searchData.result)
  // incrémentation du tableau avec les résultats
  songs.value = catalogStore.resultProvider
}
```

La dernière étape consiste à mettre en place l'affichage des éléments de recherche, y compris le formulaire ainsi que l'affichage des résultats.

```
<template>
  <div>
    <!-- partie recherche et provider  -->
    <div class="providerContainer">
      <span>
          <v-form ref="form" @submit.prevent="validateSearch" >
            <v-text-field v-model="search" class="inputSearch" :rules="searchSong" required label="Song, Artist or Album name"></v-text-field>
            <!-- <v-select v-model="select" label="Provider" :items="items" item-title="provider" item-value="id" required :rules="v => !!v || 'Select one provider !'" persistent-hint return-object single-line ></v-select> -->
            <div class="containButton">
              <v-btn color="success" type="submit" class="me-4">Validate</v-btn>
              <v-btn @click="resetFields" color="error">clear</v-btn>
            </div>
          </v-form>      
      </span>
    </div>

    <!-- partie affichage des song retourné -->
    <div class="">
        <v-container class="pa-1">
          <v-item-group v-model="selection" multiple>
            <v-row>
              <!-- liste des song retourner par le provider -->
              <v-col v-for="(item,song) in songs" :key="song" cols="12" md="4">
                <!-- card de mise en forme -->
                <v-card class="mx-auto">
                <v-item>
                  <!-- image de la pochette -->
                  <v-img :src="`${item.album.cover}`" height="150" class="text-right pa-2"></v-img>
                  <!-- div des informations -->
                  <div class="containInfor">
                    <p>Title : {{ item.title }}</p>
                    <p>Artist name : {{ item.artist.name }}</p>
                    <p>Album name : {{ item.album.title }}</p>
                  </div>
                </v-item>
                </v-card>
              </v-col>
            </v-row>
          </v-item-group>
        </v-container>
        </template>
    </div>
  </div>
</template>

```


