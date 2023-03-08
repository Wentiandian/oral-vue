<template>
  <aside class="site-sidebar" :class="'site-sidebar--' + sidebarLayoutSkin">
    <div class="site-sidebar__inner">
      <el-menu
        :default-active="menuActiveName || 'home'"
        :collapse="sidebarFold"
        :collapseTransition="false"
        class="site-sidebar__menu">
        <el-menu-item index="home" @click="$router.push({ name: 'home' })">
          <icon-svg name="shouye" class="site-sidebar__menu-icon"></icon-svg>
          <span slot="title">首页</span>
        </el-menu-item>

        <el-submenu index="ghyy">
          <template slot="title">
            <icon-svg name="shoucang" class="site-sidebar__menu-icon"></icon-svg>
            <span>挂号预约</span>
          </template>
          <el-menu-item index="gh" @click="$router.push({ name: '挂号管理' })">
            <icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>
            <span slot="title">挂号管理</span>
          </el-menu-item>
          <el-menu-item index="yy" @click="$router.push({ name: '预约管理' })">
            <icon-svg name="editor" class="site-sidebar__menu-icon"></icon-svg>
            <span slot="title">预约管理</span>
          </el-menu-item>
        </el-submenu>

        <el-submenu index="ksgl">
          <template slot="title">
            <img src="~@/assets/img/logo/logo8.png" style="width: 40px;height: 30px">
            <span>科室管理</span>
          </template>
          <el-menu-item index="ksgl" @click="$router.push({ name: '科室管理' })">
            <img src="~@/assets/img/logo/logo4.png" style="width: 30px;height: 20px">
            <span slot="title">科室管理</span>
          </el-menu-item>
        </el-submenu>

        <el-submenu index="dzbl">
          <template slot="title">
            <img src="~@/assets/img/logo/logo7.png" style="width: 30px;height: 20px">
            <span>电子病历</span>
          </template>
          <el-menu-item index="blgl" @click="$router.push({ name: '病历管理' })">
            <img src="~@/assets/img/logo/logo6.png" style="width: 30px;height: 20px">
            <span slot="title">病历管理</span>
          </el-menu-item>
          <el-menu-item index="blwdgl" @click="$router.push({ name: '病历文档管理' })">
            <img src="~@/assets/img/logo/logo2.png" style="width: 30px;height: 20px">
            <span slot="title">病历文档管理</span>
          </el-menu-item>
        </el-submenu>

        <el-submenu index="ywgl">
          <template slot="title">
            <img src="~@/assets/img/logo/logo17.png" style="width: 30px;height: 20px">
            <span>药物管理</span>
          </template>
          <el-menu-item index="ypfl" @click="$router.push({ name: '药品分类' })">
            <img src="~@/assets/img/logo/logo16.png" style="width: 30px;height: 20px">
            <span slot="title">药品分类</span>
          </el-menu-item>
          <el-menu-item index="ypcrk" @click="$router.push({ name: '药品出入库' })">
            <img src="~@/assets/img/logo/logo15.png" style="width: 30px;height: 20px">
            <span slot="title">药品出入库</span>
          </el-menu-item>
          <el-menu-item index="ywd" @click="$router.push({ name: '药物单' })">
            <img src="~@/assets/img/logo/logo9.png" style="width: 30px;height: 20px">
            <span slot="title">药物单(处方)</span>
          </el-menu-item>
        </el-submenu>

        <el-submenu index="jfgl">
          <template slot="title">
            <img src="~@/assets/img/logo/logo1.png" style="width: 30px;height: 25px">
            <span>缴费管理</span>
          </template>
          <el-menu-item index="jfdd" @click="$router.push({ name: '缴费订单' })">
            <img src="~@/assets/img/logo/logo3.png" style="width: 30px;height: 20px">
            <span slot="title">缴费订单</span>
          </el-menu-item>
        </el-submenu>

        <el-submenu index="rygl">
          <template slot="title">
            <img src="~@/assets/img/logo/logo13.png" style="width: 30px;height: 25px">
            <span>人员管理</span>
          </template>
          <el-menu-item index="hzgl" @click="$router.push({ name: '患者管理' })">
            <img src="~@/assets/img/logo/logo11.png" style="width: 30px;height: 20px">
            <span slot="title">患者管理</span>
          </el-menu-item>
          <el-menu-item index="yhgl" @click="$router.push({ name: '医护管理' })">
            <img src="~@/assets/img/logo/logo14.png" style="width: 30px;height: 20px">
            <span slot="title">医护管理</span>
          </el-menu-item>
        </el-submenu>
        <sub-menu
          v-for="menu in menuList"
          :key="menu.menuId"
          :menu="menu"
          :dynamicMenuRoutes="dynamicMenuRoutes">
        </sub-menu>
      </el-menu>
    </div>
  </aside>
</template>

<script>
  import SubMenu from './main-sidebar-sub-menu'
  import { isURL } from '@/utils/validate'
  export default {
    data () {
      return {
        dynamicMenuRoutes: []
      }
    },
    components: {
      SubMenu
    },
    computed: {
      sidebarLayoutSkin: {
        get () { return this.$store.state.common.sidebarLayoutSkin }
      },
      sidebarFold: {
        get () { return this.$store.state.common.sidebarFold }
      },
      menuList: {
        get () { return this.$store.state.common.menuList },
        set (val) { this.$store.commit('common/updateMenuList', val) }
      },
      menuActiveName: {
        get () { return this.$store.state.common.menuActiveName },
        set (val) { this.$store.commit('common/updateMenuActiveName', val) }
      },
      mainTabs: {
        get () { return this.$store.state.common.mainTabs },
        set (val) { this.$store.commit('common/updateMainTabs', val) }
      },
      mainTabsActiveName: {
        get () { return this.$store.state.common.mainTabsActiveName },
        set (val) { this.$store.commit('common/updateMainTabsActiveName', val) }
      }
    },
    watch: {
      $route: 'routeHandle'
    },
    created () {
      this.menuList = JSON.parse(sessionStorage.getItem('menuList') || '[]')
      this.dynamicMenuRoutes = JSON.parse(sessionStorage.getItem('dynamicMenuRoutes') || '[]')
      this.routeHandle(this.$route)
    },
    methods: {
      // 路由操作
      routeHandle (route) {
        if (route.meta.isTab) {
          // tab选中, 不存在先添加
          var tab = this.mainTabs.filter(item => item.name === route.name)[0]
          if (!tab) {
            if (route.meta.isDynamic) {
              route = this.dynamicMenuRoutes.filter(item => item.name === route.name)[0]
              if (!route) {
                return console.error('未能找到可用标签页!')
              }
            }
            tab = {
              menuId: route.meta.menuId || route.name,
              name: route.name,
              title: route.meta.title,
              type: isURL(route.meta.iframeUrl) ? 'iframe' : 'module',
              iframeUrl: route.meta.iframeUrl || '',
              params: route.params,
              query: route.query
            }
            this.mainTabs = this.mainTabs.concat(tab)
          }
          this.menuActiveName = tab.menuId + ''
          this.mainTabsActiveName = tab.name
        }
      }
    }
  }
</script>

<style>
.site-sidebar{
  margin-top: 10px;
}
</style>
