<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-toolbar>
        <q-toolbar-title class="items-center row">
          <img class="img" alt="w3as logo" src="project_athena_icon.png" height="40" />
          <div class="q-px-sm">
            <span class="text-weight-thin">
              Project<span class="text-weight-medium"> Athena</span>
            </span>
          </div>
        </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>
