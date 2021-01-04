<template>
  <div id="app">
    <b-navbar>
      <b-container class="d-flex justify-content-start">
          <div class="col-4">
            <b-navbar-brand to="/"><img src = "./assets/tooth-pin-white.png" alt="map pin with white tooth" width="50" height="50"></b-navbar-brand>
          </div>
          <div class="col-4 text-center d-flex justify-content-center">
            <b-navbar-nav>
              <b-nav-item class="h1" to="/">DentiPicks</b-nav-item>
            </b-navbar-nav>
          </div>
      </b-container>
    </b-navbar>
    <router-view/>
    <footer class="mt-auto py-3">
      <b-container>
        <b-row align-h="end">
          <b-col cols="3 text-right">
            <p>
              <a target="_blank" href="https://git.chalmers.se/courses/dit355/2020/group-3">GitLab Repository</a>
            </p>
          </b-col>
        </b-row>
      </b-container>
    </footer>
  </div>
</template>
