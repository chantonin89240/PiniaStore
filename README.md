# PiniaStore avec nuxt 3
## Table des matières

* Introduction
* Mise en place
* Utilisation
------------------
## Introduction 

Pinia constitue un système de gestion d'état dédié à Vue.js, offrant une approche simple et performante pour gérer l'état global d'une application.

Sa conception vise la simplicité, la performance, et l'exploitation optimale des fonctionnalités modernes de JavaScript et de Vue.js.

Actuellement, Pinia Store est compatible avec Vue 2 et Vue 3. 

L'API reste identique pour les deux versions, exception faite des aspects liés à l'installation et au rendu côté serveur (SSR - Server-Side Rendering).

pour ce tutoriel, nous allons mettre en place Pinia sur une applications nuxt 3.

L’utilisation de Pinia avec Nuxt est plus facile car Nuxt s’occupe de beaucoup de choses en ce qui concerne le rendu côté serveur.

Par exemple, vous n’avez pas besoin de vous soucier de la sérialisation ni des attaques XSS.

Pinia prend en charge Nuxt Bridge et Nuxt 3.

------------------
## Mise en place

vous avez deux possibilités pour l'installation des paquets, soit avec yarn :
```
yarn add pinia @pinia/nuxt
```
sinon avec npm :
```
npm install pinia @pinia/nuxt
```
vous retrouver dans le fichier Package.json les dépendances envers Pinia.
```
"dependencies": {
    "@pinia/nuxt": "^0.5.1",
    "pinia": "^2.1.7"
  }
```
pour finir, il reste à ajouter dans le fichier nuxt.config.js l'appel de Pinia dans les modules.
```
modules: [
    '@pinia/nuxt',
  ]
```
------------------
## Utilisation

nous allons prendre exemple d'une application de jukebox digital pour la création d'un store et réaliser une recherche sur deezer en passant par une api.

tout d'abord, il faut ajouter un dossier store puis créer le fichier.

![image](https://github.com/chantonin89240/PiniaStore/assets/49941533/69bad30c-0aa8-4b30-8dae-7b1df8f50adc)

premièrement, nous devons initialiser la définition du store.
 ```
const runtimeConfig = useRuntimeConfig()
export const useCatalogStore = defineStore('catalogStore', {

}
```

ensuite, il faut gérer la partie état, il s'agit de la partie centrale du store.

Dans Pinia, l’état est défini comme une fonction qui renvoie l’état initial. Cela permet à Pinia de fonctionner à la fois côté serveur et côté client.

```
export const useCatalogStore = defineStore('catalogStore', {
  state: (): Provider => ({
        provider: null,
        statutAdd: 0
      }),
}
```

après cela, il faut s'occuper des getters ou accesseurs.

Les accesseurs sont l’équivalent des valeurs calculées pour l’état d’un store. 

Ils peuvent être définis à l’aide de la propriété.

il s'agit des propriétés qui seront utilisé dans notre page.

```
getters: {
    resultProvider: (state) => state.provider,
    resultStatut: (state) => state.statutAdd,
  },
```

la dernière étape de notre store sera les actions, Les actions sont l’équivalent des méthodes dans les composants.

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
dans ce code nous allons faire l'appel depuis une api de notre recherche deezer puis incrémenter l'état provider.

maintenant, il est temps de d'utiliser notre store dans la page des cataloges.

nous devons d'abord appeler et initialiser le store.
```
import { useCatalogStore } from '../../stores/CatalogStore';
const catalogStore = useCatalogStore();
```

ensuite doit être construit les propriétés.
```
const search = ref('')
const searchSong = ref('');
const songs = ref([]);
```

enfin, doit être créer la méthode qui va utilisé le store pour faire la recherche deezer.
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

la dernière étape est de mettre en place dans l'affiche les éléments de recherche tellement que le formulaire ainsi que les l'affichage du resultat.

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


