<template>
  <div>
    <div class="text-center">
      Worried about the post office delivering your ballot on time?<br />
      Pick your state:
      <select  v-model=selectedState @change="getCountyData()">
        <option
            v-for="(state, index) in states"
            :key="index"
            :value="index"
        >{{ state["State/Territory"] }}</option>
      </select>
      <hr />
    </div>
    <StateData
        v-if="selectedState"
        :state=states[selectedState]
        :counties="counties"
    />
  </div>
</template>

<script>
import StateData from "@/components/StateData";
import { AIRTABLE_API_KEY, AIRTABLE_BASE } from '@/config';
// import getData from "@/router/index"; // todo

export default {
  name: 'StateList',
  components: {StateData},
  props: {

  },
  data: function(){
    return {
      states: [],
      selectedState: null,
      selectedStateName: null,
      counties: [{},],
    }
  },
  computed: {
    stateName: function(){
      if ( this.selectedState ) return this.states[this.selectedState]['State/Territory'];
      return null;
    },
  },
  watch: {
    $route(to){
      this.selectedStateName = to.params.state;
    },
    stateName: function(){
      // Change route
      let state_route = this.$route.path.split("/")[1].replace("-", " ");
      if( this.$route.name !== "County" || this.stateName !== state_route) {
        let path = "/" + this.stateName.replace(/\s/g, "-");
        this.$router.push({path: path});
      }
    }
  },
  methods: {
    getCountyData() {
      const Airtable = require('airtable');
      const base = new Airtable({apiKey: AIRTABLE_API_KEY}).base(AIRTABLE_BASE);
      const county_data = [{}, ];
      base('County Local Election Information').select({
        filterByFormula: `State="${this.states[this.selectedState]['State/Territory']}"`,
        sort: [
          {field: 'County', direction: 'asc'}
        ]
      }).eachPage(function page(records, fetchNextPage) {
        records.forEach(function (record) {
          county_data.push(record.fields)
        });
        fetchNextPage();
      }, function done(err) {
        if (err) {
          console.error(err);
          return;
        }
      });
      this.counties = county_data;

    },
    checkRoute(){
      // Check for state route
      if( this.$route.params.state !== undefined) {
        this.selectedStateName = this.$route.params.state.replace("-", " ");
        let i=0;
        for(i; i<this.states.length; i++) {
          if( this.states[i]["State/Territory"] === this.selectedStateName ) break;
        }
        this.selectedState = i;
        this.getCountyData();
      }
    }
  },
  mounted: function() {
    const Airtable = require('airtable');
    const base = new Airtable({apiKey: AIRTABLE_API_KEY}).base(AIRTABLE_BASE);
    const state_data = [{},];
    const _this = this;
    base('State Absentee Voting Data').select({
      sort: [
        {field: 'State/Territory', direction:'asc'}
      ]
    }).eachPage(function page(records, fetchNextPage) {
      records.forEach(function(record) {
        state_data.push(record.fields);
      });
      fetchNextPage();
    }, function done(err){
      if(err){console.error(err); return;}
      _this.states = state_data;
      _this.checkRoute();
    });
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
