<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" color="#128c7e" dark :mini-variant="miniVariant" :clipped="clipped" fixed app>
      <v-list>
        <v-list-item v-for="(item, i) in items" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar color="#128c7e" dark :clipped-left="clipped" fixed app>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <img src="/whatsapp.svg" class="mr-1" height="30" />
      <!-- <v-toolbar-title v-text="title" /> -->
      <img src="/icono.png" class="mr-1" height="35" />
      <!-- Menú de tres puntos -->
      <v-menu offset-y right>
        <template #activator="{ on }">
          <v-btn icon v-on="on">
            <v-icon>mdi-dots-vertical</v-icon>
          </v-btn>
        </template>
        <v-list>
          <!-- <v-list-item>
            <v-list-item-title>Configuración</v-list-item-title>
          </v-list-item> -->
          <v-list-item>
            <v-btn icon>
              <v-icon>mdi-logout</v-icon>
            </v-btn>
            <v-list-item-title>
              <a href="https://advocatus-online.com/" class="link-styled">Cerrar sesión</a>
            </v-list-item-title>
          </v-list-item>

        </v-list>
      </v-menu>
      <!-- Fin del menú de tres puntos -->
    </v-app-bar>
    <v-main>
      <v-container fluid>
        <Nuxt />
      </v-container>
    </v-main>
    <v-navigation-drawer v-model="rightDrawer" :right="right" temporary fixed>
      <v-list>
        <v-list-item @click.native="right = !right">
          <v-list-item-action>
            <v-icon light>mdi-repeat</v-icon>
          </v-list-item-action>
          <v-list-item-title>Switch drawer (click me)</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-footer :absolute="!fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      clipped: true,
      drawer: true,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Lista de Chats',
          to: '/',
        },
        {
          icon: 'mdi-email',
          title: 'Envio de Campañas',
          to: '/send',
        },

      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Advocatus Online',
    }
  },
}
</script>

<style>
.pre-wrap {
  white-space: pre-wrap;
}
</style>

