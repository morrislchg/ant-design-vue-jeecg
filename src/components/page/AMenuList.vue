<template>
  <div>
    <a-menu  :mode="mode" style="width: 100%;">
      <a-menu-item v-for="(item,index) in menu" :key="item.path" @click="select">
        <span><a-icon :type="item.meta.icon" /><span><a>{{item.meta.title}}</a></span></span>
      </a-menu-item>
    </a-menu>
  </div>
</template>

<script>
    import { SET_MENUSSECOND } from "@/store/mutation-types"
    export default {
        name: "AMenuList",
        props: {
            menu: {
                type: Array,
                required: true
            },
            theme: {
                type: String,
                required: false,
                default: 'dark'
            },
            mode: {
                type: String,
                required: false,
                default: 'inline'
            },
        },
        data(){
            return{
            }
        },
        methods:{
            select(e){
                const { mode, theme, menu } = this
                menu.map(item=>{
                    if(e.key===item.path){
                        localStorage.removeItem("selectMenu")
                        localStorage.removeItem("selectMenuKey")
                        console.log(item)
                        localStorage.setItem("selectMenu",JSON.stringify(item))
                        localStorage.setItem("selectMenuKey",e.key)
                        this.$emit('menuSecond',item,e.key)
                    }
                })
            }
        },
    }
</script>

<style scoped>
 /deep/ .ant-menu{
   background-color:#1890FF;
 }
</style>