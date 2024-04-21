<template>
  <div class="dashboard-container">
    <div class="dashboard-text">Role: {{ name }}</div>
    <panel-group @handleSetLineChartData="handleSetLineChartData" />
  </div>
</template>

<script>
import PanelGroup from './components/PanelGroup'
import axios from '@/utils/axios';

export default {
  name: 'Dashboard',
  components: {
    PanelGroup
  },
  data(){
    return{
      name:"unknown"
    }
  },
  computed: {

  },
  methods: {
    handleSetLineChartData() {

    },
    getrole(){
      axios.get('/user/session')
        .then(response=>{
          console.log(response.data)
          this.name = response.data?.[0].slice(6)
          console.log("name = response.data")
        }).catch(error=>{
          this.$message.error("error")
          console.log(error)
        })
      console.log(name)
    }
  },
  created(){
    this.getrole()
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
