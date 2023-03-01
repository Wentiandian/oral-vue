<template>
  <el-form>
    <h2>布局设置</h2>
    <el-form-item label="导航栏类型">
      <el-radio-group v-model="navbarLayoutType">
        <el-radio label="inverse" border>默认</el-radio>
        <el-radio label="default" border>反向</el-radio>
      </el-radio-group>
    </el-form-item>
    <el-form-item label="侧边栏皮肤">
      <el-radio-group v-model="sidebarLayoutSkin">
        <el-radio label="dark" border>主题</el-radio>
        <el-radio label="light" border>明亮</el-radio>
      </el-radio-group>
    </el-form-item>
  </el-form>
</template>

<script>
  export default {
    computed: {
      navbarLayoutType: {
        get () { return this.$store.state.common.navbarLayoutType },
        set (val) { this.$store.commit('common/updateNavbarLayoutType', val) }
      },
      sidebarLayoutSkin: {
        get () { return this.$store.state.common.sidebarLayoutSkin },
        set (val) { this.$store.commit('common/updateSidebarLayoutSkin', val) }
      }
    }
  }
</script>

