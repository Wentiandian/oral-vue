<template>
  <div class="mod-home">
    <div class="home-cards">
      <el-carousel :interval="4000" type="card" height="300px">
        <el-carousel-item v-for="item in cardList" :key="item.id" align="center">
          <h3><img :src="item.src" alt="" style="height:400px; width:700px"  /></h3>
        </el-carousel-item>
      </el-carousel>
    </div>
    <div class="home-view">
      <el-row>
        <el-col :span="12" v-for="item in viewList" :key="item.id">
          <el-card :body-style="{ padding: '0px', height: '650px' }">
            <div class="home-title">
              <h1>{{item.title}}</h1>
            </div>
            <img :src="item.src" class="image" style="height: 400px;width: 600px">
            <div style="padding: 14px;">
              <span class="text">{{item.text}}</span>
              <div class="bottom clearfix">
                <el-button type="primary" class="button" round @click="enterPage(item)">{{item.button}}</el-button>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        cardList: [
          {id: 1, src: require('@/assets/img/card/card1.jpg')},
          {id: 2, src: require('@/assets/img/card/card2.jpg')},
          {id: 3, src: require('@/assets/img/card/card3.jpg')},
          {id: 4, src: require('@/assets/img/card/card4.jpg')},
          {id: 5, src: require('@/assets/img/card/card5.jpg')},
          {id: 6, src: require('@/assets/img/card/card6.jpg')}
        ],
        viewList: [
          { id: 1,
            src: require('@/assets/img/view/registered.jpg'),
            title: '爱口腔，爱健康',
            text: '快速挂号，预约医生，马上就诊！',
            button: '挂号预约'
          },
          { id: 2,
            src: require('@/assets/img/view/department.jpg'),
            title: '专业，让您的口腔爱上美',
            text: '牙体牙髓科:治疗龋病、牙本质过敏症、慢性磨耗、楔状缺损、牙髓炎、根尖周炎等硬组织疾病;\n' +
              '牙周病科:治疗牙龈、牙槽骨、牙周膜等支持组织疾病;\n' +
              '口腔黏膜病科:治疗复发性口腔溃疡、扁平苔藓等口腔黏膜疾病;\n' +
              '口腔颌面外科:拔除牙齿或治疗口腔囊肿、外伤、肿瘤等疾病的科室;\n' +
              '口腔修复科针对牙体缺损或牙列缺失进行治疗的科室;\n' +
              '口腔正畸科:治疗牙列不齐、前突等错(牙合)畸形的科室；\n' +
              '14岁以下的患者儿童到牙病科就诊。',
            button: '查看科室'
          },
          { id: 3,
            src: require('@/assets/img/view/medical_record.jpg'),
            title: '口腔医生是您口腔健康的好朋友',
            text: '电子病历是新医疗体系的重要载体，信息学和医学科学相结合，是传统医学迈向现代医学的重要标志。',
            button: '电子病历'
          },
          { id: 4,
            src: require('@/assets/img/view/pharmaceuticals.jpg'),
            title: '请带走您的健康口腔，留下我们共同的笑容',
            text: '一尘不染，才能永保健康；一丝不敬，方能精品纷呈，产品质量好，全靠工作质量保。',
            button: '药物信息'
          },
          { id: 5,
            src: require('@/assets/img/view/bill.jpg'),
            title: '自信的微笑，更美丽的你',
            text: '我院基本医疗服务项目是根据 《广东省发展改革委关于督促落实收费目录清单制度的通知》 （粤发改价格函［2019］340号）要求，公示我院医疗服务项目收费价格，具体医疗服务项目收费清单详见附件。',
            button: '费用结算'
          },
          { id: 6,
            src: require('@/assets/img/view/personnel.jpg'),
            title: '一个微笑，闪耀出珍珠般的光芒',
            text: '人员信息安全保证',
            button: '人员信息'
          }
        ]
      }
    },
    methods: {
      enterPage (item) {
        switch (item.id) {
          case 1 :window.location.href = '/#/ghyy-gh'; break
          case 2 :window.location.href = '/#/ksgl-ksgl'; break
          case 3 :window.location.href = '/#/dzbl-blgl'; break
          case 4 :window.location.href = '/#/ywgl-ypfl'; break
          case 5 :window.location.href = '/#/jfgl-jfdd'; break
          case 6 :window.location.href = '/#/rygl-hzgl'; break
        }
      }
    }
  }
</script>

<style>
.mod-home {
  line-height: 1.5;
}
.home-title{
  text-align: center;
  font-size: 15px;
  color: #247ba8;
  font-family: 方正舒体;
}
.text{
  display: inline-block;
  height: 100px;
}
.el-carousel__item h3 {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 200px;
  margin: 0;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n+1) {
  background-color: #d3dce6;
}
.bottom {
  margin-top: 10px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both
}
</style>

