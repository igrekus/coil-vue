<template>
    <q-layout view="lhr Lpr lfr">
        <q-header elevated>
            <q-toolbar>
                <q-toolbar-title>Coil gen</q-toolbar-title>
                <q-tabs v-model="tab" shrink stretch>
                    <q-route-tab
                            to="/cnc_edit"
                            label="cnc"
                            exact
                    />
                    <q-route-tab
                            to="/"
                            label="svg"
                            exact
                    />
                </q-tabs>
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
      tab: 'cnc'
    }
  },
  methods: {
  }
}
</script>

<style>
</style>
