<template>
  <div class="content-wrapper page-with-layout-nav">


    <div class="flash-container flash-container-page">
    </div>


    <div class="container-fluid container-limited ">
      <div class="content">
        <div class="project-members-page prepend-top-default">
          <div class="panel panel-default">
            <div class="panel-heading">
              添加用户到项目
            </div>
            <div class="panel-body">
              <p class="light">
                下面列举的是可以访问项目的所有用户
              </p>
              <el-form ref="form" :model="form" label-width="80px">

                <el-form-item label="账号">
                  <el-autocomplete
                      v-model="form.name"
                      :fetch-suggestions="querySearchAsync"
                      custom-item="my-item-zh"
                      placeholder="请输入内容"
                      @select="handleSelect"
                      style="min-width: 300px;"
                  ></el-autocomplete>
                </el-form-item>
                <el-form-item label="权限">
                  <el-select v-model="value" placeholder="请选择">
                    <el-option
                        v-for="item in Metadata.projectPower"
                        :label="item.label"
                        :value="item.value">
                    </el-option>
                  </el-select>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" @click="onSubmit">添加用户到组</el-button>
                </el-form-item>
              </el-form>
            </div>
          </div>

          <div class="panel panel-default">
            <div class="panel-heading">
              <strong>{{info.projectName}}</strong>
              <span class="badge">{{pusers.length}}</span>
            </div>
            <ul class="content-list">
              <li class="group_member js-toggle-container" v-for="item in pusers">
                <div class="controls">
                  <el-dropdown @command="handleCommand">
                    <span class="el-dropdown-link">
                      {{item.role|projectRole}}<i class="el-icon-caret-bottom el-icon--right"></i>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item v-for="e in Metadata.projectPower" trigger="click"
                                        :command="e.value + ',' + item.userId">{{e.label}}
                      </el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
                  <a class="btn btn-remove" data-remote="true" rel="nofollow" @click="remove(item)">{{(item.userId ===
                    userInfo.userId)?'离开':'移除'}}</a>
                </div>
                <span class="list-item-name">
                  <img class="avatar s40" alt="" :src="item.photo">
                  <strong>
                    <router-link :to="{path:'user',query:{id:item.userId}}">{{item.name}}</router-link>
                  </strong>
                  <span class="label label-success" v-if="item.userId === userInfo.userId">当前用户</span>
                  <div class="cgray">{{item.email}}</div>
                </span>
              </li>
            </ul>
          </div>


          <div class="panel panel-default">
            <div class="panel-heading">
              <strong>{{info.groupName}}</strong>
              <span class="badge">{{gusers.length}}</span>
            </div>
            <ul class="content-list">
              <li class="group_member js-toggle-container" v-for="item in gusers">
                <span class="list-item-name">
                  <img class="avatar s40" alt="" :src="item.photo">
                  <strong>
                    <router-link :to="{path:'user',query:{id:item.userId}}">{{item.name}}</router-link>
                  </strong>
                  <span class="label label-success" v-if="item.userId === userInfo.userId">当前用户</span>
                  <div class="cgray">{{item.email}}</div>
                </span>
              </li>
            </ul>
          </div>

        </div>

      </div>
    </div>
  </div>
</template>

<style lang="styl" rel="stylesheet/stylus" scoped type="text/css">

</style>

<script type="text/ecmascript-6">
  import BasePage from 'src/extend/BasePage'
  import Server from 'src/extend/Server'
  import UserItem from 'src/components/User/item.vue'
  import {mapState} from 'vuex'
  import Vue from 'vue'
  Vue.component('my-item-zh', {
    functional: true,
    render: function (h, ctx) {
      return h('li', ctx.data, [ h(UserItem, {
        props: {
          item: ctx.props.item
        }
      }) ])
    },
    props: {
      item: { type: Object, required: true }
    }
  })

  export default{
    mixins: [ BasePage ],
    components: {},
    name: 'projects_members',
    data () {
      return {
        info: {},
        userQueryList: [],
        form: {
          name: '',
          projectId: '',
          description: ''
        },
        req: {
          projectId: '',
          role: '',
          userId: ''
        },
        value: '',
        pusers: [],
        gusers: []
      }
    },
    mounted: function () {
      this.req.projectId = this.$route.query.id
      this.loadData()
    },
    computed: mapState({
      Metadata: state => state.Metadata
    }),
    methods: {
      querySearchAsync (queryString, cb) {
        Server({
          url: 'users/search',
          method: 'get',
          params: { key: queryString }
        }).then((response) => {
          var results = this.pretreatmentList(response.data.data)
          cb(results)
        }).catch(() => {

        })
      },
      pretreatmentList (list) {
        var result = []
        list.forEach(function (e) {
          result.push({
            'value': e.name,
            'email': e.email,
            'name': e.name,
            'photo': e.photo,
            'id': e.id
          })
        })
        return result
      },
      loadData () {
        Server({
          url: 'project/projectuser',
          method: 'get',
          params: {
            count: 100,
            projectId: this.req.projectId,
            start: 0
          }
        }).then((response) => {
          this.pusers = response.data.data
        })
        Server({
          url: 'project/projectinfo',
          method: 'get',
          params: {
            id: this.req.projectId
          }
        }).then((response) => {
          this.info = response.data.data
          var me = this
          Server({
            url: 'project/groupuser',
            method: 'get',
            params: {
              count: 100,
              id: this.info.groupId,
              start: 0
            }
          }).then((response) => {
            me.gusers = response.data.data
          })
        })
      },
      handleSelect (item) {
        window.console.log(item)
        this.req.userId = item.id
      },
      handleCommand (command) {
        var role = command.split(',')[ 0 ]
        var userId = command.split(',')[ 1 ]
        Server({
          url: 'project/projectuser',
          method: 'put',
          data: {
            projectId: this.req.projectId,
            role: role,
            userId: userId
          }
        }).then((response) => {
          this.$message('修改成功')
          this.loadData()
        }).catch(() => {

        })
      },
      onSubmit: function () {
        this.req.role = this.value
        Server({
          url: 'project/projectuser',
          method: 'post',
          data: this.req
        }).then((response) => {
          this.$message('添加成功')
          this.loadData()
        }).catch(() => {

        })
      },
      remove (item) {
        Server({
          url: 'project/projectuser',
          method: 'delete',
          data: {
            projectId: this.req.projectId,
            userId: item.userId
          }
        }).then((response) => {
          this.$message('删除成功')
          if (item.userId === this.userInfo.userId) {
            this.$router.push({ path: 'dashboard_groups' })
          } else {
            this.loadData()
          }
        }).catch(() => {

        })
      }
    }
  }
</script>
