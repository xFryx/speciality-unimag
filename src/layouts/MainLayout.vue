<template>
  <q-layout view="hHh lpR fFf">
    <!-- Header -->
    <q-header elevated>
      <q-toolbar>
        <q-btn flat dense round icon="menu" @click="drawer = !drawer" aria-label="Menu" />
        <q-toolbar-title>Especialización 2025</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <!-- Navigation Drawer -->
    <q-drawer v-model="drawer" show-if-above bordered>
      <q-list padding>
        <q-item clickable v-ripple to="/" exact>
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section> Traductor </q-item-section>
        </q-item>

        <q-item clickable v-ripple to="/images">
          <q-item-section avatar>
            <q-icon name="image" />
          </q-item-section>
          <q-item-section> Generador de imagenes </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <!-- Page Content -->
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const drawer = ref(false)
</script>
