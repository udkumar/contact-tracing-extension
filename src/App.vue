<template>
  <div>
    <global-options-modal />
    <navbar />
    <b-container>
      <data-selection @data-updated="dataUpdated" style="margin-top: 10px !important" />
      <b-row>
        <b-col cols="0" style="display: none !important">
          <b-tabs>
            <b-tab title="Timesheet" active>
              <timesheet-table :data="data" />
            </b-tab>
          </b-tabs>
        </b-col>
        <b-col cols="12">
          <user-input></user-input>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import Navbar from './components/Navbar'
import DataSelection from './components/DataSelection'
import DataTable from './components/DataTable'
import GlobalOptionsModal from './components/GlobalOptionsModal'
import TimesheetTable from './components/TimesheetTable'
import MileageTable from './components/MileageTable'
import UserInput from './components/UserInput'

export default {
  components: {
    Navbar,
    DataSelection,
    DataTable,
    GlobalOptionsModal,
    TimesheetTable,
    MileageTable,
    UserInput
  },
  data() {
    return {
      data: {
        name: '',
        items: []
      }
    }
  },
  methods: {
    dataUpdated(data) {
      this.data = data
    }
  }
}
</script>

<style lang="scss">
@media print {
  .nav-tabs li a:not(.active) {
    display: none !important;
  }
}

$body-color: #00203fff;
$theme-colors: (
  'primary': #3db2dcff
);
@import '../node_modules/bootstrap/scss/bootstrap';
@import '../node_modules/bootstrap-vue/src/index.scss';

.multiselect__tag {
  background-color: #3db2dcff !important;
}
.multiselect__option--highlight {
  background-color: #3db2dcff !important;
}
.multiselect__option--selected.multiselect__option--highlight {
  background-color: #3db2dcff !important;
}
</style>
