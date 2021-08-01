<template>
  <b-navbar toggleable="lg" type="light" variant="light">
    <b-container>
      <b-navbar-brand href="/index.html">
        <img src="icons/icon.svg" alt="Timline exporter" width="25px" />
        USF: Survey Location History <b-badge variant="primary">Dev</b-badge>
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
