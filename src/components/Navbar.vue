<template>
  <b-navbar toggleable="lg" type="light" variant="light">
    <b-container>
      <b-navbar-brand href="/index.html">
        <img src="icons/plane.gif" alt="Timline exporter" width="100px" />
        USF: Survey Location History <b-badge variant="primary">Dev 1.0.0</b-badge>
      </b-navbar-brand>

      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" is-nav>
        <!-- Right aligned nav items -->
        
      </b-collapse>
    </b-container>
  </b-navbar>
</template>

<script>
const PACKAGE_VERSION = require('../../package.json').version

export default {
  data() {
    return {
      packageVersion: PACKAGE_VERSION
    }
  }
}
</script>

<style scoped>
.navbar-icon {
  font-size: 14px;
}
</style>
