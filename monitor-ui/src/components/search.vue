<template>
  <div class="" style="display:inline-block">
   <ul class="search-ul">
      <li class="search-li">
        <Select
          style="width:300px;"
          v-model="endpoint"
          filterable
          remote
          ref="select"
          clearable
          :placeholder="$t('placeholder.input')"
          :remote-method="getEndpointList"
          >
          <Option v-for="(option, index) in endpointList" :value="option.option_value" :key="index">
            <Tag :color="endpointTag[option.option_type_name] || choiceColor(option.option_type_name, index)" class="tag-width">{{option.option_type_name}}</Tag>{{option.option_text}}</Option>
        </Select>
      </li>
      <li class="search-li" style="margin-left:20px">
          <Select v-model="timeTnterval" :disabled="disableTime" style="width:80px" @on-change="getChartsConfig">
            <Option v-for="item in dataPick" :value="item.value" :key="item.value">{{ item.label }}</Option>
          </Select>
      </li>
      <li class="search-li">
        <Select v-model="autoRefresh" :disabled="disableTime" style="width:100px" @on-change="getChartsConfig" :placeholder="$t('placeholder.refresh')">
          <Option v-for="item in autoRefreshConfig" :value="item.value" :key="item.value">{{ item.label }}</Option>
        </Select>
      </li>

      <li class="search-li" style="margin-left:20px">
        <DatePicker type="datetimerange" :value="dateRange" format="yyyy-MM-dd HH:mm:ss" placement="bottom-end" @on-change="datePick" :placeholder="$t('placeholder.datePicker')" style="width: 320px"></DatePicker>
      </li>
      <li class="search-li">
        <button type="button" class="btn btn-sm btn-confirm-f"
          @click="getChartsConfig()">
          <i class="fa fa-search" ></i>
          {{$t('button.search')}}
        </button>
      </li>
      <li class="search-li">
        <button type="button" v-if="isShow" @click="changeRoute" class="btn btn-sm btn-cancle-f btn-jump">{{$t('button.endpointManagement')}}</button>
      </li>
   </ul>
  </div>
</template>

<script>
import {dataPick, autoRefreshConfig, endpointTag, randomColor} from '@/assets/config/common-config'
export default {
  name: '',
  data() {
    return {
      endpoint: '',
      endpointObject: {},
      endpointList: [],
      endpointTag: endpointTag,
      randomColor: randomColor,
      cacheColor: {},
      ip: {},
      timeTnterval: -1800,
      dataPick: dataPick,
      dateRange: ['', ''],
      autoRefresh: 10,
      disableTime: false,
      autoRefreshConfig: autoRefreshConfig,
      params: {
        // time: this.timeTnterval,
        // group: 1,
        // endpoint: '192.168.0.16',
        // start: Date.parse(this.dateRange[0]),
        // end: Date.parse(this.dateRange[1])
      }
    }
  },
  computed: {
    isShow: function () {
      return !this.$root.$validate.isEmpty_reset(this.endpoint)
    }
  },
  watch: {
    endpoint: function (val) {
      if (val) {
        this.endpointObject = this.endpointList.find(ep => {
          return ep.option_value === val
        })
      } else {
        this.endpointObject = {}
      }
    },
    isShow: function () {
      this.clearEndpoint = []
      this.getEndpointList('.')
      this.$parent.showCharts = false 
      this.$parent.showRecursive = false
    }
  },
  mounted() {
    this.getEndpointList('.')
    if (!this.$root.$validate.isEmpty_reset(this.$route.params)) {
      this.endpoint = this.$route.params.option_value
      this.endpointObject = this.$route.params
    }
    if (this.$root.$validate.isEmpty_reset(this.$route.params) && !this.$root.$validate.isEmpty_reset(this.$route.query)) {
      this.endpoint = this.$route.query.endpoint
      this.$root.$store.commit('storeip', {
        id: '',
        option_value: this.$route.query.endpoint,
        type: this.$route.query.type
      })
    }
  },
  methods: {
    choiceColor (type,index) {
      let color = ''
      // eslint-disable-next-line no-prototype-builtins
      if (Object.keys(this.cacheColor).includes(type)) {
        color = this.cacheColor[type]
      } else {
        color = randomColor[index]
        this.cacheColor[type] = randomColor[index]
      }
      return color
    },
    getMainConfig () {
      if (this.$root.$validate.isEmpty_reset(this.endpointObject) && !this.$root.$validate.isEmpty_reset(this.$root.$store.state.ip)) {
        this.endpointObject = this.$root.$store.state.ip
        this.$root.$store.commit('storeip', {})
      }
      const type = this.endpointObject.type
      return new Promise(resolve => {
        let params = {
          type: type
        }
        this.$root.$httpRequestEntrance.httpRequestEntrance('GET', this.$root.apiCenter.mainConfig.api, params, (responseData) => {
          resolve(responseData)
        })
      })
    },
    datePick (data) {
      this.dateRange = data
      this.disableTime = false
      if (this.dateRange[0] && this.dateRange[1]) {
        if (this.dateRange[0] === this.dateRange[1]) {
          this.dateRange[1] = this.dateRange[1].replace('00:00:00', '23:59:59')
        }
        this.disableTime = true
        this.autoRefresh = 0
      }
      this.getChartsConfig()
    },
    getEndpointList(query) {
      let params = {
        search: query,
        page: 1,
        size: 1000
      }
      this.$root.$httpRequestEntrance.httpRequestEntrance('GET', this.$root.apiCenter.resourceSearch.api, params, (responseData) => {
        this.endpointList = responseData
      })
    },
    async getChartsConfig () {
      if (this.$root.$validate.isEmpty_reset(this.endpoint)) {
        return
      }
      if (this.$root.$validate.isEmpty_reset(this.endpointObject) && !this.$root.$validate.isEmpty_reset(this.$root.$store.state.ip)) {
        this.endpointObject = this.$root.$store.state.ip
        this.$root.$store.commit('storeip', {})
      }
      let params = {}
      if (this.endpointObject.type === 'sys' ) {
        params = {
          autoRefresh: this.autoRefresh,
          time: this.timeTnterval,
          endpoint: this.endpointObject.option_value,
          start: this.dateRange[0] ===''? '':Date.parse(this.dateRange[0])/1000,
          end: this.dateRange[1] ===''? '':Date.parse(this.dateRange[1])/1000,
          guid: this.endpointObject.option_value,
          sys: true
        }  
        this.$parent.manageCharts({}, params)
        return
      }
      const res = await this.getMainConfig()
      let url = res.panels.url
      let key = res.search.name
      params = {
        autoRefresh: this.autoRefresh,
        time: this.timeTnterval,
        endpoint: this.endpoint,
        start: this.dateRange[0] ===''? '':Date.parse(this.dateRange[0])/1000,
        end: this.dateRange[1] ===''? '':Date.parse(this.dateRange[1])/1000,
        sys: false
      }
      url = url.replace(`{${key}}`,params[key])
      this.$root.$httpRequestEntrance.httpRequestEntrance('GET',url, params, responseData => {
        this.$parent.manageCharts(responseData, params)
      },{isNeedloading: false})
    },
    changeRoute () {
      this.$router.push({name: 'endpointManagement', params: {search: this.endpointObject.option_value}})
    }
  },
  components: {
  }
}
</script>

<style scoped lang="less">
  .search-li {
    display: inline-block;
  }
  .search-ul>li:not(:first-child) {
    padding-left: 10px;
  }
  .tag-width {
    width: 80px;
    text-align: center;
  }
</style>
