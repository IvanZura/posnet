<template lang="pug">
  v-app
    v-navigation-drawer(dark v-model = "drawer" :mini-variant = "miniVariant" :clipped = "clipped" fixed app width = "220"
    style = 'background:rgba(0,151,167,0.1)')
      v-list
        v-list-tile(v-for = "(item, i) in items" :key = "i" :to = "item.to" router exact)
          v-list-tile-action
            v-icon(color = "white") {{ item.icon }}
          v-list-tile-content
            v-list-tile-title.white--text {{ item.title }}
            
    v-toolbar(:clipped-left = "clipped" fixed app flat dark :color = "$store.state.Colores.Header")
      v-icon(@click = "drawer = !drawer") fas fa-arrow-alt-circle-down
      v-btn(v-show = "drawer" icon @click.stop = "miniVariant = !miniVariant")
        v-icon {{ `chevron_${miniVariant ? 'right' : 'left'}` }}
      //v-btn(icon @click.stop = "clipped = !clipped")
        v-icon web
      //v-btn(icon @click.stop = "fixed = !fixed")
        v-icon remove
      v-toolbar-title Testing POSNET
        //v-img(src="https://www.coppel.com/wcsstore/AuroraStorefrontAssetStore/images/logo.svg" width="170" aspect-ratio="1.4" contain)
      //v-btn(icon @click.stop = "rightDrawer = !rightDrawer")
        v-icon menu
    v-content.fondo
      v-container
        nuxt
    v-navigation-drawer(v-model = "rightDrawer" :right = "right" temporary fixed)
      v-list
        v-list-tile(@click.native="right = !right")
          v-list-tile-action
            v-icon(light) compare_arrows
          v-list-tile-title Cambiar click aqui
    //v-footer(app dark :color = "$store.state.Colores.Footer")
      span &copy; 2019
    v-bottom-nav(app :value="showNav" fixed dark :color = "$store.state.Colores.Footer")
      v-btn(flat :color = "$store.state.Colores.Botones" to = '/posnet/venta')
        span Venta
        v-icon fas fa-credit-card
      v-btn(flat :color = "$store.state.Colores.Botones" to = '/posnet/devolucion')
        span Devolucion
        v-icon fas fa-undo
      v-btn(flat :color = "$store.state.Colores.Botones" to = '/posnet/anulacion')
        span Anulacion
        v-icon fas fa-ban
</template>

<script>
export default {
  data() {
    return {
      clipped: true,
      drawer: true,
      items: [
        {
          icon: 'fas fa-dollar-sign',
          title: 'POSNET',
          to: '/posnet/venta'
        }
      ],
      miniVariant: true,
      right: true,
      rightDrawer: false,
      title: 'Vuetify.js',
      showNav: true
    }
  }
}
</script>
<style scoped>
.fondo {
  background: url('../assets/fondos/fondo17.jpg') no-repeat center center fixed;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
}
</style>
