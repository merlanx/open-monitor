<template>
  <div class=" ">
    <PageTable :pageConfig="pageConfig"></PageTable>
    <ModalComponent :modelConfig="modelConfig"></ModalComponent>
    <ModalComponent :modelConfig="authorizationModel">
      <div slot="authorization">  
        <div class="marginbottom params-each">
          <label class="col-md-2 label-name lable-name-select">{{$t('field.endpoint')}}:</label>
          <Select v-model="authorizationModel.addRow.user" multiple filterable style="width:338px">
              <Option v-for="item in authorizationModel.userList" :value="item.id" :key="item.name">
              {{item.display_name}}({{item.name}})</Option>
          </Select>
        </div>
      </div>
    </ModalComponent>
    <ModalDel :ModelDelConfig="ModelDelConfig"></ModalDel>
  </div>
</template>

<script>
let tableEle = [
  {title: 'tableKey.name', value: 'name', display: true},
  {title: 'tableKey.nickname', value: 'display_name', display: true},
  {title: 'tableKey.email', value: 'email', display: true},
  {title: 'tableKey.activeDate', value: 'created_string', display: true}
]
const btn = [
    {btn_name: 'button.edit', btn_func: 'editF'},
    {btn_name: 'button.authorization', btn_func: 'authorizationF'},
    {btn_name: 'button.remove', btn_func: 'deleteConfirm'}
  ]
export default {
  name: '',
  data() {
    return {
      ModelDelConfig: {
        deleteWarning: false,
        msg: '',
        callback: null
      },
      pageConfig: {
        CRUD: this.$root.apiCenter.setup.role.get,
        researchConfig: {
          input_conditions: [
            {value: 'search', type: 'input', placeholder: 'placeholder.input', style: ''}],
          btn_group: [
            {btn_name: 'button.search', btn_func: 'search', class: 'btn-confirm-f', btn_icon: 'fa fa-search'},
            {btn_name: 'button.add', btn_func: 'addRole', class: 'btn-cancle-f', btn_icon: 'fa fa-plus'}
          ],
          filters: {
            search: ''
          }
        },
        table: {
          tableData: [],
          tableEle: tableEle,
          // filterMoreBtn: 'filterMoreBtn',
          // primaryKey: 'guid',
          btn: btn,
          pagination: this.pagination,
          handleFloat:true,
        },
        pagination: {
          __orders: '',
          total: 0,
          page: 1,
          size: 10
        }
      },
      modelTip: {
        key: 'name',
        value: null
      },
      modelConfig: {
        modalId: 'add_role_Modal',
        modalTitle: 'button.add',
        isAdd: true,
        config: [
          {label: 'tableKey.name', value: 'name', placeholder: 'tips.required', v_validate: 'required:true', disabled: false, type: 'text'},
          {label: 'tableKey.nickname', value: 'display_name', placeholder: 'tips.required', v_validate: 'required:true', disabled: false, type: 'text'},
          {label: 'tableKey.email', value: 'email', placeholder: 'tips.required', v_validate: 'required:true|noEmail', disabled: false, type: 'text'}
        ],
        addRow: { // [通用]-保存用户新增、编辑时数据
          name: null,
          display_name: null,
          email: null
        }
      },
      authorizationModel: {
        modalId: 'authorization_model',
        modalTitle: 'button.authorization',
        isAdd: true,
        saveFunc: 'authorizationSave',
        config: [
          {name:'authorization',type:'slot'}
        ],
        addRow: {
          user: []
        },
        userList: []
      }, 
      id: null
    }
  },
  mounted () {
    this.initData(this.pageConfig.CRUD, this.pageConfig)
  },
  methods: {
    initData (url= this.pageConfig.CRUD, params) {
      this.$root.$tableUtil.initTable(this, 'GET', url, params)
    },
    addRole () {
      this.modelConfig.isAdd = true
      this.$root.JQ('#add_role_Modal').modal('show')
    },
    addPost () {
      let params = JSON.parse(JSON.stringify(this.modelConfig.addRow))
      params.operation = 'add'
      this.$root.$httpRequestEntrance.httpRequestEntrance('POST', this.$root.apiCenter.setup.role.update, params, () => {
        this.$Message.success(this.$t('tips.success'))
        this.$root.JQ('#add_role_Modal').modal('hide')
        this.initData(this.pageConfig.CRUD, this.pageConfig)
      })
    },
    editF (rowData) {
      this.modelConfig.isAdd = false
      this.modelTip.value = rowData[this.modelTip.key]
      this.id = rowData.id
      this.modelConfig.addRow = this.$root.$tableUtil.manageEditParams(this.modelConfig.addRow, rowData)
      this.$root.JQ('#add_role_Modal').modal('show')
    },
    editPost () {
      let params = JSON.parse(JSON.stringify(this.modelConfig.addRow))
      params.operation = 'update'
      params.role_id = this.id
      this.$root.$httpRequestEntrance.httpRequestEntrance('POST', this.$root.apiCenter.setup.role.update, params, () => {
        this.$Message.success(this.$t('tips.success'))
        this.$root.JQ('#add_role_Modal').modal('hide')
        this.initData(this.pageConfig.CRUD, this.pageConfig)
      })
    },
    deleteConfirm (rowData) {
      this.ModelDelConfig =  {
        deleteWarning: true,
        msg: rowData.name,
        callback: () => {
          this.delF(rowData)
        }
      }
    },
    delF (rowData) {
      let params = {role_id: rowData.id, operation: 'delete' }
      this.$root.$httpRequestEntrance.httpRequestEntrance('POST', this.$root.apiCenter.setup.role.update, params, () => {
        this.$Message.success(this.$t('tips.success'))
        this.initData(this.pageConfig.CRUD, this.pageConfig)
      })
    },
    authorizationF (rowData) {
      this.id = rowData.id
      this.$root.$httpRequestEntrance.httpRequestEntrance('GET', this.$root.apiCenter.setup.userManagement.get, "", (responseData) => {
        this.authorizationModel.userList = responseData.data
        this.existUser()
      })
    },
    existUser () {
      this.$root.$httpRequestEntrance.httpRequestEntrance('GET', this.$root.apiCenter.setup.userManagement.get, {role: this.id}, (responseData) => {
        responseData.data.forEach((item) => {
          this.authorizationModel.addRow.user.push(item.id)
        })
        this.$root.JQ('#authorization_model').modal('show')
      })
    },
    authorizationSave () {
      let params = {
        role_id: this.id,
        user_id: this.authorizationModel.addRow.user
      }
      this.$root.$httpRequestEntrance.httpRequestEntrance('POST', this.$root.apiCenter.setup.role.authorization, params, () => {
        this.$Message.success(this.$t('tips.success'))
        this.$root.JQ('#authorization_model').modal('hide')
      })
    } 
  },
  components: {},
}
</script>

<style scoped lang="less">
</style>
