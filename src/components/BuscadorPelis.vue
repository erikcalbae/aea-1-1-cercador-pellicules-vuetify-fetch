<script setup>
import { ref } from 'vue'
import {
  buscarPelicules,
  obtenirDetallPelicula
} from '@/services/communicationManager'

const textCerca = ref('')
const pelicules = ref([])
const peliculaDetall = ref(null)

const dialeg = ref(false)
const haCercat = ref(false)
const carregant = ref(false)
const error = ref('')

async function cercar() {
  if (!textCerca.value.trim()) {
    error.value = "Escriu el títol d'una pel·lícula"
    return
  }

  haCercat.value = true
  carregant.value = true
  error.value = ''
  pelicules.value = []

  try {
    const dades = await buscarPelicules(textCerca.value.trim())

    if (dades.Response === 'True') {
      pelicules.value = dades.Search
    } else {
      pelicules.value = []
    }
  } catch (e) {
    error.value = "S'ha produït un error en cercar les pel·lícules"
  } finally {
    carregant.value = false
  }
}

async function veureMesInfo(imdbID) {
  carregant.value = true
  error.value = ''

  try {
    const dades = await obtenirDetallPelicula(imdbID)

    if (dades.Response === 'True') {
      peliculaDetall.value = dades
      dialeg.value = true
    } else {
      error.value = "No s'ha pogut obtenir la informació de la pel·lícula"
    }
  } catch (e) {
    error.value = "S'ha produït un error en obtenir els detalls de la pel·lícula"
  } finally {
    carregant.value = false
  }
}

function netejar() {
  textCerca.value = ''
  pelicules.value = []
  peliculaDetall.value = null
  dialeg.value = false
  haCercat.value = false
  carregant.value = false
  error.value = ''
}
</script>

<template>
  <v-app>
    <v-app-bar color="primary">
      <v-app-bar-title>
        Buscador de pel·lícules
      </v-app-bar-title>
    </v-app-bar>

    <v-main>
      <v-container fluid class="pa-6">
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="textCerca"
              label="Títol de la pel·lícula"
              placeholder="Exemple: Batman"
              variant="outlined"
              clearable
              @keyup.enter="cercar"
            />

            <div class="d-flex ga-2">
              <v-btn
                color="primary"
                :loading="carregant"
                @click="cercar"
              >
                Cercar
              </v-btn>

              <v-btn
                variant="outlined"
                @click="netejar"
              >
                Netejar
              </v-btn>
            </div>
          </v-col>
        </v-row>

        <v-alert
          v-if="error"
          type="error"
          class="mb-6"
        >
          {{ error }}
        </v-alert>

        <v-alert
          v-if="!haCercat && !error"
          type="info"
          variant="tonal"
          class="mb-6"
        >
          Escriu el títol d'una pel·lícula i prem «Cercar».
        </v-alert>

        <div
          v-if="carregant"
          class="text-center pa-8"
        >
          <v-progress-circular
            indeterminate
            color="primary"
          />

          <p class="mt-4">
            Cercant pel·lícules...
          </p>
        </div>

        <v-alert
          v-if="haCercat && !carregant && !error && pelicules.length === 0"
          type="warning"
          variant="tonal"
          class="mb-6"
        >
          No s'han trobat pel·lícules.
        </v-alert>

        <v-row v-if="!carregant && pelicules.length > 0">
          <v-col
            v-for="pelicula in pelicules"
            :key="pelicula.imdbID"
            cols="12"
            sm="6"
            md="4"
          >
            <v-card height="100%">
              <v-img
                :src="
                  pelicula.Poster !== 'N/A'
                    ? pelicula.Poster
                    : 'https://placehold.co/300x450?text=Sense+poster'
                "
                height="420"
                cover
              />

              <v-card-title>
                {{ pelicula.Title }}
              </v-card-title>

              <v-card-subtitle>
                {{ pelicula.Year }} · {{ pelicula.Type }}
              </v-card-subtitle>

              <v-card-actions>
                <v-btn
                  color="primary"
                  @click="veureMesInfo(pelicula.imdbID)"
                >
                  MÉS INFORMACIÓ
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-dialog
      v-model="dialeg"
      max-width="800"
    >
      <v-card v-if="peliculaDetall">
        <v-card-title>
          {{ peliculaDetall.Title }}
        </v-card-title>

        <v-card-text>
          <p><strong>Director:</strong> {{ peliculaDetall.Director }}</p>
          <p><strong>Actors:</strong> {{ peliculaDetall.Actors }}</p>
          <p><strong>Argument:</strong> {{ peliculaDetall.Plot }}</p>
          <p><strong>Durada:</strong> {{ peliculaDetall.Runtime }}</p>
          <p><strong>Gènere:</strong> {{ peliculaDetall.Genre }}</p>
          <p><strong>Valoració IMDb:</strong> {{ peliculaDetall.imdbRating }}</p>
        </v-card-text>

        <v-card-actions>
          <v-spacer />

          <v-btn @click="dialeg = false">
            Tancar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>