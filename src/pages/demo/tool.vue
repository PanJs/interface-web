<template>
  <div class="content">
    <restful-tool :apiInfo="apiInfo"></restful-tool>
  </div>
</template>

<style lang="styl" rel="stylesheet/stylus" scoped type="text/css">
  .content
    padding 30px
</style>

<script type="text/ecmascript-6">
  import BaseComponent from 'src/extend/BaseComponent'
  import RestfulTool from 'src/components/RestfulTool/index.vue'
  export default {
    mixins: [ BaseComponent ],
    name: 'demo_tool',
    components: { RestfulTool },
    data: function () {
      return {
        apiInfo: {
          path: 'users/info',
          name: '接口demo',
          method: 'get',
          description: '获取用fdsafdsa户姓名',
          parameters: {
            path: null,
            query: null
          }
        }
      }
    },
    watch: {},
    mounted () {
    },
    methods: {}
  }
</script>
