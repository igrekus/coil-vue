<template>
    <q-layout view="lhr Lpr lfr">
        <q-header elevated>
            <q-toolbar>
                <q-toolbar-title>Coil gen</q-toolbar-title>
            </q-toolbar>
        </q-header>

        <q-page-container>
            <router-view/>
        </q-page-container>
    </q-layout>
</template>

<script>
export default {
  name: 'MainLayout',
  data () {
    return {
    }
  },
  methods: {
  }
}
</script>

<style>
</style>
