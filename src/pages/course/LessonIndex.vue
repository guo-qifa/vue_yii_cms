<template>
  <div>
    <div class="row h44">
      <div class="col-xs-6">
        <p>教程［<span style="color: red">{{cname}}</span>］的课程</p>
      </div>
      <div class="col-xs-6 flex-end-center">
        <button-box type="add" :to="'/lesson/edit?cid='+this.$route.query.cid+'&cname='+this.$route.query.cname" title="添加课程"></button-box>
        <button-box type="edit" @click="editItem"></button-box>
        <button-box type="delete" @click="deleteItem"></button-box>
      </div>
    </div>

    <table-box ref="tableBox" :url="table.url" :columns="table.columns" :key-name="table.keyName" :parse-data="table.parseData" :data="table.data" :multi-select="table.multiSelect">

      <tr slot="row"  slot-scope="props">
        <td class="tc">{{props.index}}</td>
        <td><router-link :to="'/lesson/edit?id='+props.item.id" class="title"><span v-if="props.item.pid>0">&nbsp;&nbsp;&nbsp;&nbsp;|----</span>{{props.item.name}}</router-link></td>
        <td class="tc">{{props.item.viewnum}}</td>
        <td>{{props.item.sort}}</td>
      </tr>


    </table-box>


  </div>
</template>

<script>


  import TableBox from '../../components/mod/TableBox'
  import ButtonBox from  '../../components/mod/ButtonBox'

  export default {
    name: 'CategoryIndex',
    components: {TableBox,ButtonBox},

    data:function () {
      return {
        cname:'',
        cid:'',
        table:{
          url:this.$api.lesson_index+'?cid='+this.$route.query.cid,
          multiSelect:false,
          keyName:'id',
          columns:[
            {
              title:"名称",
              field:"name",
              width:''
            },
            {
              title:"浏览量",
              field:"viewnum",
              width:'80'
            },
            {
              title:"排序值",
              field:"value",
              width:'100'
            },

          ],
          parseData:function (data) {
            data.forEach(function(row){

            })
          }
        },

      }
    },
    mounted:function () {
      let breadcrumb=[
        {
          link: '#',
          title:'系统设置'
        },
        {
          link:'/course/index',
          title:'教程管理'
        },
        {
          link:'',
          title:'课程管理'
        },

      ]
      this.$store.commit('SET_BREADCRUMB',breadcrumb)
      this.cname=this.$route.query.cname
      this.cid=this.$route.query.cid

      console.log('mounted')
    },
    methods:{
      editItem:function () {
        let table=this.$refs.tableBox
        let id=table.getSelectedRowId()[0]
        if(!id){
          this.$toast({
            title: '请选择一行数据',
          })
          return
        }
        this.$router.push({path:'/lesson/edit',query:{
            id:id,
            cid:this.cid,
            cname:this.cname
          }})

      },
      deleteItem:function () {

        let table=this.$refs.tableBox
        let id=table.getSelectedRowId()[0]
        if(!id){
          this.$toast({
            title: '请选择一行数据'
          })
          return
        }
        this.$confirm({
          content:"删除操作不能恢复，您确定要删除吗",

          success: ()=> {
            let post_data={
              id: id,
              status:9,
              operation:'delete'
            }
            this.$network.post(this.$api.lesson_save,post_data).then((res)=>{
              table.refresh()
            })
          },
        })


      },



    },
    activated:function () {
      //此部分是为了处理缓存，如果缓存页面和当前页面cid不一致，则重新请求数据
      let new_url=this.$api.lesson_index+'?cid='+this.$route.query.cid
      if(this.table.url!=new_url){
        this.table.url=new_url
        this.$refs.tableBox.urlIn=new_url
        this.$refs.tableBox.reload()

      }
    },
    beforeRouteEnter:function (to,from,next) {
      //从编辑页面返回到列表，应该刷新一下当前页面
      if(from.path=='/lesson/edit'){
        next(vm=>{
          vm.$refs.tableBox.refresh()
        })
      }
      next()
    }


  }
</script>

<style lang="less" scoped>

</style>
