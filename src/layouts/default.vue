<template>
  <v-app>
    <v-layout>
      <v-navigation-drawer v-model="drawer" class="bg-deep-grey" theme="dark">
        <v-container class="text-center pa-4">
          <v-img
            ax-width="160"
            style="max-width: 100%; height: auto; filter: invert(1) brightness(2);"
            src="https://static.wixstatic.com/media/96f38e_00bc8de829464068a01361d80c000a67~mv2.png/v1/fill/w_152,h_32,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/Logo%20Negro.png"
            max-width="120"
            contain
          />
        </v-container>
        <v-list density="compact" item-props :items="items" nav />
        <template v-slot:append>
          <div class="pa-2">
            <v-btn block style="background-color: aliceblue; color: black;">
              Cerrar Sesión
            </v-btn>
          </div>
        </template>
      </v-navigation-drawer>

      <v-app-bar border="b" class="ps-4" flat>
        <v-btn v-if="$vuetify.display.mdAndUp" icon @click="drawer = !drawer">
          <v-icon>mdi-menu</v-icon>
        </v-btn>
        <v-app-bar-nav-icon v-if="$vuetify.display.smAndDown" @click="drawer = !drawer" />
        <v-app-bar-title>Aplicación Países</v-app-bar-title>
        <v-switch v-model="model" hide-details inset></v-switch>
        <template #append>
          <v-btn class="text-none me-2" height="48" icon slim>
            <v-avatar color="surface-light" image="https://cdn-icons-png.flaticon.com/512/219/219986.png" size="32" />
            <v-menu activator="parent">
              <v-list density="compact" nav>
                <v-list-item append-icon="mdi-cog-outline" link title="Configuraciones" to="/vista-configuraciones" />
                <v-list-item append-icon="mdi-logout" link title="Cerrar Sesión" />
              </v-list>
            </v-menu>
          </v-btn>
        </template>
      </v-app-bar>

      <v-main style="background-color:#90CAF9;">
        <div class="pa-4">
          <v-sheet border="dashed md" style="background-color:#90CAF9;" height="auto" rounded="lg" width="100%">
            <router-view />
          </v-sheet>
        </div>
      </v-main>
    </v-layout>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'

const drawer = ref(true)
const model = ref(true)
const items = ref([
  { title: 'Home', prependIcon: 'mdi-home', link: true, to: '/' },
  { title: 'Vista Uno', prependIcon: 'mdi-view-grid', link: true, to: '/vista-uno' },
  { title: 'Vista dos', prependIcon: 'mdi-archive-eye', link: true, to: '/vista-dos' },
  { title: 'Configuraciones', prependIcon: 'mdi-cog-outline', link: true, to: '/vista-configuraciones' },
])
</script>
