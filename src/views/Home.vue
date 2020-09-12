<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    {{ plus10 }}
    <div @click="addReactive">添加</div>
    <div>{{ test2.join() }}</div>
    <div>{{ test3.bValue.join() }}</div>
    <div @click="changeObj">测试reactive</div>
  </div>
</template>

<script>
// @ is an alias to /src
import { reactive, getCurrentInstance, watch, ref, computed, onMounted, toRefs } from 'vue'

function haha () {
  console.log('haha');
}
export default {
  name: 'Home',
  setup () {
    const { ctx } = getCurrentInstance();
    console.log(ctx.$router.currentRoute.value.path);

    //点击按钮 ref ++
    const num = ref(1);
    const addRef = () => {
      num.value++;
    };

    //点击按钮 reactive ++
    const count = reactive({ value: 1 });
    const obj = reactive({
      test1: '李',
      test2: ['solace'],
      test3: {
        aValue: '哈哈',
        bValue: ['打印']
      }
    })

    const changeObj = () => {
      obj.test3.bValue.push('打印出来了吗')
      obj.test2.push('lee')
      haha()
    }
    const addReactive = () => {
      count.value++;
    };
    watch(count, (newVal, oldVal, clean) => {
      console.log(newVal.value + "," + oldVal.value);
      clean(() => console.log("Clean"));
    });
    //计算×10
    const plus10 = computed(() => count.value * 10);

    //获取state.status
    const status = computed(() => ctx.$store.state.status);
    const setStatus = () => {
      ctx.$store.commit("setStatus", status.value);
      console.log(ctx.$store.state.status);
    };
    onMounted(() => {console.log('onmounted');
    })
    return {
      num,
      count,
      addRef,
      addReactive,
      plus10,
      status,
      setStatus,
      changeObj,
      ...toRefs(obj)
    };
    
  },
  methods: {
    init () {

    }
  }
}
</script>
