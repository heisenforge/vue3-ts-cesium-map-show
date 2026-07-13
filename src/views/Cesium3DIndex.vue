<!--
 * @Description: 
 * @Version: 1.668
 * @Autor: Hawk
 * @Date: 2021-06-17 15:13:58
 * @LastEditors: Hawk
 * @LastEditTime: 2021-11-10 15:13:18
-->
<template>
  <div class="Cesium3DIndex" id="cesiumContainer"></div>
  <ShowLngLat ref="ShowLngLatRef" />
  <ButtonTools ref="ButtonToolsRef" />
</template>
<script lang="ts">
/* eslint-disable no-debugger */
import { GController } from '@/utils/ctrlCesium/Controller'
import { getBaseMapConfig, getBaseMapImageryList } from '@/utils/getFormatData/BaseMap'
import ShowLngLat from '@/components/ShowLngLat.vue' // @ is an alias to /src
import ButtonTools from '@/components/ButtonTools.vue'
import Titleset from '@/utils/ctrlCesium/Titleset'
import Manager from '@/utils/ctrlCesium/effects/Manager'
import Primitive from '@/utils/ctrlCesium/model/Primitive'
import RoadNetwork from '@/utils/ctrlCesium/lines/RoadNetwork'
import { defineComponent, onBeforeMount, nextTick, ref } from 'vue'
declare const Cesium: any

export default defineComponent({
  name: 'Cesium3DIndex',
  components: { ShowLngLat, ButtonTools },
  setup() {
    let viewer = null
    const ShowLngLatRef = ref()
    const ButtonToolsRef = ref()

    const initMap = (BaseMapConfig:any, MapImageryList:any) => {
      viewer = GController.init(BaseMapConfig, MapImageryList)
      window.Gviewer = viewer // 全局控制台 调试viewer
      // 显示经纬度绑定事件
      ShowLngLatRef.value.initCesiumHandler(viewer)
      // 飞到配置的坐标 (防止与电厂对焦冲突导致闪烁)
      // ButtonToolsRef.value.flyTo()

      // 加载火电厂冷却塔模型
      const position = Cesium.Cartesian3.fromDegrees(113.9218, 22.5116, 0)
      const heading = Cesium.Math.toRadians(0)
      const hpr = new Cesium.HeadingPitchRoll(heading, 0, 0)
      const orientation = Cesium.Transforms.headingPitchRollQuaternion(position, hpr)
      
      const towerEntity = viewer.entities.add({
        name: '冷却塔模型',
        position: position,
        orientation: orientation,
        model: {
          uri: `${process.env.BASE_URL}thermal_power_plant__cooling_tower.glb`,
          minimumPixelSize: 128,
          maximumScale: 20000,
          scale: 1.0,
          heightReference: Cesium.HeightReference.CLAMP_TO_GROUND
        }
      })

      // 镜头直接飞向并对焦到这个冷却塔模型
      viewer.zoomTo(towerEntity)

      // 处理 白膜 (江苏电厂项目不需要深圳城市白膜)
      // const GTitleset = new Titleset(viewer)
      // GTitleset.init()

      // 模型List
      const GPrimitive = new Primitive(viewer)
      GPrimitive.init()

      // 处理 配置好的点效果列表
      const GManager = new Manager(viewer)
      GManager.init()

      // 公路效果 (江苏电厂项目不需要深圳公路网)
      // const GRoadNetwork = new RoadNetwork(viewer, 'road')
      // GRoadNetwork.init()

    }
    onBeforeMount(() => {
      nextTick(async () => {
        const BaseMapConfig:any = await getBaseMapConfig()
        const MapImageryList:any = await getBaseMapImageryList()
        initMap(BaseMapConfig, MapImageryList)
      })
    })
    return {
      ShowLngLatRef, ButtonToolsRef
    }
  },
})
</script>

<style lang="scss" scoped>
.Cesium3DIndex {
  height: 100%;
}
</style>