<template>
  <div class="content">
    <div class="container-fluid">
      <div class="row">
        <div class="col-12">
          <card>
              <div class="row">
                <div class="col-md-4">
                  <fg-input type="text"
                            label="Unit Id"
                            placeholder="Unit ID"
                            v-model="unit.unitId">
                  </fg-input>
                </div>
                <div class="col-md-14">
                  <fg-input type="number"
                            label="Number of records"
                            placeholder="5"
                            v-model="unit.numberOfRecords">
                  </fg-input>
                </div>
              </div>
            <div class="text-center">
              <button type="submit" class="btn btn-info btn-fill float-right" @click.prevent="updateTable">
                Try Out
              </button>
              <br></br>
            </div>
            <template slot="header">
              <h4 class="card-title">Unit Data Log</h4>
              <p class="card-category">Here is a subtitle for this table</p>
            </template>
            <div class="table-responsive">
              <l-table class="table-hover table-striped"
                       :columns="table1.columns"
                       :data="table1.data">
              </l-table>
            </div>
          </card>
        </div>

      </div>
    </div>
  </div>
</template>
<script>
  import LTable from 'src/components/UIComponents/Table.vue'
  import Card from 'src/components/UIComponents/Cards/Card.vue'
  import {elasticSearchBaseUrl} from '../../../globalConstants.js'
  const tableColumns = ['Id', 'Timestamp']
  const tableData = [{
  }]
  export default {
    components: {
      LTable,
      Card
    },
    created: function() {
        this.fetchData()
        this.timer = setInterval(this.fetchData, 30000)
    },
    methods: {
      timeConverter(UNIX_timestamp){
        var a = new Date(UNIX_timestamp * 1000);
        var months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
        var year = a.getFullYear();
        var month = months[a.getMonth()];
        var date = a.getDate();
        var hour = a.getHours();
        var min = a.getMinutes();
        var sec = a.getSeconds();
        var time = date + ' ' + month + ' ' + year + ' ' + hour + ':' + min + ':' + sec ;
        return time;
      },
      fetchData () {
        console.log("[TableList] Fetching data ")
        var option = {
            "query": {
              "match_all": {}
            },
            "stored_fields" : [],
            "sort": [
               { "timestamp": "desc" }
            ]
        }
        option.size = this.unit.numberOfRecords
        this.$http.post(elasticSearchBaseUrl, option).then(response => {
          var dataList = []
          for (var i in response.data.hits.hits) {
            var data = {}
            data.id = response.data.hits.hits[i]._id
            data.timestamp = this.timeConverter(response.data.hits.hits[i].sort[0]/1000)
            dataList.push(data) 
          }
          this.table1.data = dataList
          //this.unit.unitId = dataList[0]._id
          console.log(this.unit.unitId)
        }, response => {
          //error callback
        });
      },
      updateTable () {
        alert('Your data: ' + JSON.stringify(this.unit))
      },
      cancelAutoUpdate: function () {
          console.log("[TableList] Cancel auto update")
          clearInterval(this.timer)
      }
    },
    data () {
      return {
        timer: '',
        table1: {
          columns: [...tableColumns],
          data: [...tableData]
        },
        table2: {
          columns: [...tableColumns],
          data: [...tableData]
        },
        unit: {
          unitId: '',
          numberOfRecords: 5
        }
      }
    },
    beforeDestroy() {
      this.cancelAutoUpdate()
    }
  }
</script>
<style>
</style>
