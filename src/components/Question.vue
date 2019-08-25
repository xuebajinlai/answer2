<template>
  <div>
    <p class="questionText">{{title}} 
      <van-tag type="primary" v-if="type==0">问答题</van-tag>
      <van-tag type="success" v-if="type==1">单选题</van-tag>
    </p>

    <van-cell-group v-if="type==0">
      <van-field v-model="result" @input="updateValue(result)"/>
    </van-cell-group>

    <van-radio-group v-model="result" @input="updateValue(result)" v-if="type==1">
      <van-cell-group>
        <van-cell v-for="op of options" :key="op" :title="op" clickable @click="result = op">
          <van-radio slot="right-icon" :name="op" />
        </van-cell>
      </van-cell-group>
    </van-radio-group>
  </div>
</template>

<script>
export default {
    name: 'Question',
    data() {
      return {
        result: ""
      };
    },
    props: ['id','title','type','options','answer'],
    methods:{
      updateValue:function(value){
          if(value == this.answer) {
            this.$emit('input', true);
            return
          }
          this.$emit('input', false);
      }
  }
}
</script>

<style scoped>
.questionText {
  margin-right: 30px;
  margin-left: 30px;
}
</style>