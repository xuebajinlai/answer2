<template>
  <div id="app">
    <van-nav-bar title="入群试题" @click-left="onReturnMain" left-text="主页" 
      :right-text="butname" @click-right="onNextStep" left-arrow>
      <!-- <van-icon name="wap-home" slot=""/> -->
    </van-nav-bar>
    <van-steps :active="active" active-icon="success" active-color="#38f">
      <van-step>答题须知</van-step>
      <van-step>基本信息</van-step>
      <van-step>开始答题</van-step>
      <van-step>通过答题</van-step>
    </van-steps>

    <!-- step 0 -->
    <template v-if="active == 0">
      <h3 class="infoTitle">答题须知</h3>
      <p class="infoText">
        1. 本群致力于给大家提供一个良好的讨论学习，展示自己的氛围。群里经常有各种问答、互动以及讲座。
      </p>
      <p class="infoText">
        2. 本群禁止发送不良信息、脏话、宣群、打广告。不得刷屏。
      </p>
      <p class="infoText">
        3. 通过这个答题的小测试，你可以获得一个入群口令。输入口令即可入群。
      </p>
      <p class="infoText">
        4. 题目难度<strong style="color:red">与所选年级有关</strong>,请如实选择。答案不正确也没有关系，尽力答题即可。
      </p>
    </template>

    <!-- Step1 -->
    <template v-if="active == 1">
      <p class="infoText">基本信息</p>
      <van-cell-group>
        <van-field label="QQ号" v-model="basicUser" placeholder="请输入QQ号" clearable/>
      </van-cell-group>

      <van-field readonly clickable label="年级" :value="basicLevelExample[basicLevel]" placeholder="选择年级" @click="basicLevelShow = true" />
      <van-popup v-model="basicLevelShow" position="bottom">
        <van-picker
          show-toolbar
          :columns="basicLevelExample"
          @cancel="basicLevelShow = false"
          @confirm="onBasicLevelChange"
        />
      </van-popup>
    </template>

    <!-- Step2 -->
    <template v-if="active == 2">
      <template v-for="q in text">
        <Question v-bind:key="q.id" v-bind:type="q.type" 
                  v-bind:title="q.title" v-bind:options="q.options"
                  v-bind:answer="q.answer" v-model="q['result']"/>
      </template>
    </template>

    <!-- Step3 -->
    <template v-if="active == 3">
      <h3 class="infoTitle">答题结果</h3>
      <p class="infoText">你的得分是：{{Math.round(realScore)}} 分！</p>
      <p class="infoText" v-if="successScore > realScore">很遗憾没有通过测试（重新测试次数有限制）。</p>
      <p class="infoText" v-else-if="getCount()">重复提交次数过多。</p>
      <template v-else>
        <p class="infoText" >恭喜你，入群口令是：</p>
        <van-cell-group>
          <van-field v-model="flag" center readonly>
            <van-button id="clip" :data-clipboard-text="flag" slot="button" size="small" type="primary">复制</van-button>
          </van-field>
        </van-cell-group>
      </template>
    </template>
  </div>
</template>

<script>

import Question from './components/Question'
import Clipboard from 'clipboard';

export default {
  name: 'app',
  components: {
     Question
  },
  data() {
    return {
      active: 0,
      //butname: "确认",
      basicUser: "",
      basicLevelExample: ['小初中', '高中', '本科、研究生及以上', ],
      basicLevelShow: false,
      basicLevel: 0,
      text: [],
      successScore: 0,
      realScore: 0,
      flag:  "",
    };
  },
  computed: {
    butname: function () {
      if (this.active < 0 || this.active >= 4) {
        return ""
      }
      let dic = ["确认","下一步","提交",""]
      return dic[this.active]
    }
  },
  methods: {
    onReturnMain() {
      this.text = null;
      this.successScore = 100;
      this.flag = "";
      this.active = 0;
    },
    onNextStep() {
      if(this.active >= 4 || this.active < 0){
        // 非预期情况
        return
      }
      if(this.active == 1 ){
        // 载入试题
        let tmp = require("./assets/text" + this.basicLevel + ".json");
        this.text = tmp.text;
        this.successScore = tmp.ok;
        this.flag = tmp.flag;
      }
      if(this.active == 2) {
        // 检查结果
        let count = 0;
        for(let q of this.text) {
          if(q['result'] == true) {
            count += 1;
          }
        }
        this.realScore = count / this.text.length * 100;
        sessionStorage.setItem("count", parseInt(sessionStorage.getItem("count")) + 1);
      }
      this.active += 1;
    },
    onBasicLevelChange(value) {
      // 获取级别
      this.basicLevel = this.basicLevelExample.indexOf(value);
      this.basicLevelShow = false;
    },
    getCount() {
      // 是否超过尝试次数
      return parseInt(sessionStorage.getItem("count")) >= 4;
    }
  },
  mounted() {
    // 初始化部分
    const clipboard = new Clipboard('#clip');
    if(sessionStorage.getItem("count") == null) {
      sessionStorage.setItem("count","0");
    }
  }
}
</script>

<style>



.infoTitle {
  margin-left: 30px;
  margin-right: 20px;
}

.infoText {
  margin-left: 20px;
  margin-right: 20px;
}
</style>
