<script setup lang="ts">

import {search, windows} from "../../tools";
import {usePhenotypeOntologySelectorStore} from "../../pinia/Store.ts";
import {reactive, ref} from "vue";
import {Action, ElMessageBox} from "element-plus";
import router from "../../router";
import PhenotypeOntologyInteractions from "../../components/PhenotypeOntologyInteractions.vue";

const store = usePhenotypeOntologySelectorStore()
let isLoad = ref(false)
let data = reactive({
  databases: ["#none"],
  status: "OK"
})
let dbs = reactive({})

function graph(){
  return {
    "lines":data["lines"],
    "nodes":data["nodes"]
  }
}
function query(){
  isLoad.value = false
  search.phenotypeOntologySelector(store,data,function (){
    isLoad.value = true
    if (data.status == "Error"){
      ElMessageBox.alert('No results were queried.', 'Notice', {
        confirmButtonText: 'Return to the query page',
        callback: (action:Action) => {
          router.push("/home")
        },
      })
    }
  })
  search.description(store.current_select,descriptions)
}
search.getStatisticalData(dbs)
isLoad.value = true
const descriptions = reactive([])
</script>

<template>
  <el-row>
    <el-col :span="4">
      <el-select
          v-model="store.current_select"
          filterable
          placeholder="Please select a species"
          @change="query()"
      >
        <el-option
            v-for="item in dbs['databaseNames']"
            :key="item"
            :label="item"
            :value="item"
        />
      </el-select>

      <div v-if="store.current_select != '#none'">
        <el-collapse accordion>
          <el-collapse-item v-for="descr in descriptions" :title="descr['source']" :name="descr['source']">
            <div>
              Latin name：{{descr["latin"]}}<br>
              Chinese name：{{descr["chinesename"]}}<br>
              Link：<el-tag style="cursor: pointer" type="primary" @click="windows.goLink(descr['link'])">Redirect</el-tag><br>
              Description：{{descr["description"]}}
            </div>
          </el-collapse-item>
        </el-collapse>
      </div>

    </el-col>
    <el-col :span="1">
      <el-divider style="height: 100%" direction="vertical"/>
    </el-col>
    <el-col v-loading="!isLoad" :span="19">
      <div v-if="store.current_select == '#none'">
        Please select a species
      </div>
      <PhenotypeOntologyInteractions v-else-if="isLoad" :datas="graph()"></PhenotypeOntologyInteractions>
    </el-col>
  </el-row>
</template>

<style scoped>

</style>