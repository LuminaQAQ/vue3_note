1. setup(): 类似 data() 的东西
   
   ```js
   // setup(): 类似 data() 的东西
   
   // 1. 必须要写返回值, 变量可以在return那日定义, 也可先定义后返回
      setup() {
          const a = 0;
   
          return {
              a
          }
      }
   
   // 2. vue2 可以兼容 setup() 的变量, 但 vue3 不支持读取 vue2 的data 和 methods 
   // 就是不建议混用
   // 混用时, 以 vue3 为主
   data() {
      return {
          name: "lumina"
      }
   },
   setup() {
      console.log(this.name); // undefined
   }
   
   ```
   
   

2. ref(): 响应式数据
   
   ```js
    // 1. 定义数据
   let name = ref("lumina");
   let age = ref(18);
   const job = ref({
       type: "Ui",
       salary: "30k",
   });
   
   // 2. 修改数据(模板字符串自动会加value, 直接原值写就行)
   name.value = "Lumina";
   age.value = 19;
   job.value.type = "Java"
   
   ```

3. reactive
   
   ```js
    // 1. 对象类型定义
    const job = reactive(
      {
        type: "UI",
        salary: "30k",
      }
    )
    const hobby = reactive(["抽烟", "喝酒", "烫头"]);
   
    // 2. 对象类型值修改
    job.type = "Java"; // 比起ref, 少了一个.value的火锅城
    hobby[0] = "学习";
   ```

4. 因 setup() 改变了使用方式的 api 
   
   ```js
   setup(props, context) {
       console.log(props);  // 可以看到 props 参数
       console.log(context);  // 可以看到
   }
   
   // 1. 子组件与父组件通信
   emits: ['hello'] // 类似 props 声明
   context.emit("parentEvent", val)
   
   // 2.插槽
   <template v-slot:name>
       <span></spann>
   </template>
   ```

5. conputed(): 计算属性
   
   ```js
   // computed(() => { return a + b })
   const person = reactive({
     firstName: '张',
     lastName: '三',
   })
   
   // 1. 简写形式(无法修改计算属性的值)
   person.fullName = computed(() => {
     return person.firstName + '-' + person.lastName;
   })
   
   // 2. 完整形式
   person.fullName = computed({
     get() {
       return person.firstName + '-' + person.lastName;
     },
     set(val) {
       const arr = val.split('-');
       person.firstName = arr[0];
       person.lastName = arr[1];
     }
   })
   ```

6. watch(): 监听属性
   
   ```js
   // watch(var||arr||fn, callback, options)
   const sum = ref(0);
   const say = ref("你好啊");
   
   // 1. 监视1个响应式数据, 注意不要写成 sum.value
   watch(sum, (newVal, oldVal) => {
     console.log(newVal, oldVal);
   })
   
   // 2. 监视多个响应式数据
   watch([sum, say], (newVal, oldVal) => {
     const sumNewVal = newVal[0];
     const sumOldVal = oldVal[0];
   
     const sayNewVal = newVal[1];
     const sayOldVal = oldVal[1];
   
     console.log("sum: ", sumNewVal, sumOldVal);
     console.log("say: ", sayNewVal, sayOldVal);
   }, { immediate: true})
   
   // 3. 监视对象中的多个数据
   watch([() => person.name, () => person.age], (newVal, oldVal) => {
       // ...
   }, { immediate: true})
   
   // 注1: 至少在现在版本(3.4), 是没办法拿到被监听对象的 oldVal 的
   // 解决办法1: 拿出该值, 单独设置(如: name = ref(18))
   
   // 注2: 监听内置属性(如: person.name), 同样不行
   // 解决办法2: 将 person.name 使用函数返回来代替原来的写法
   watch(() => person.age, (newVal, oldVal) => {
     console.log(newVal, oldVal);
   })
   
   // 注3: 默认开启深度监视(deep), 现版本可手动关闭
   
   // 注4: deep(不进行配置), 默认情况下, 该情况都能监视到最底层
   watch(() => person, (newVal, oldVal) => {
     console.log(newVal, oldVal);
   })
   // 但不能直接深度监视 reactive对象内 的 某个属性, 需要手动开启 deep 
   watch(() => person.job, (newVal, oldVal) => {
    console.log(newVal, oldVal);
   })
   ```

7. watchEffect()
   
   ```js
   // 可以监视该函数内所有使用过的参数, 未使用的不进行监测
   watchEffect(() => {
     const a = sum.value;
     const b = say.value;
   
     console.log(a, b);
   })
   ```

8. hook: 可以理解成mixin/代码复用
   
   ```js
   // 1. 封装一个函数, 保存在 src/hooks 中(自己创建);
   // 若要使用里面的某个数据, 记得返回出去
   export default function () {
      const point = reactive({
          x: 0,
          y: 0,
      })
   
      function pointChange(e) {
          point.x = e.pageX;
          point.y = e.pageY;
   
          console.log(point);
      }
   
      onMounted(() => {
          window.addEventListener('click', pointChange);
      });
   
      onBeforeUnmount(() => {
          window.removeEventListener('click', pointChange);
      })
   
      return point;
    }
   
   // 2. 文件命名规则, useXXX
   usePoint.js
   
   // 3. 在 setup 中要使用到的模块, 且需被封装的, 也要在该js内先进行引入
   import { onBeforeUnmount, onMounted, reactive } from 'vue'
   
   // 4. 引入模块时, 建议将名字也是以 useXXX 为标准
   import usePoint from '../hooks/usePoint'
   
   ```

9. toRef(): 将对象内属性, 转变成可动态变化的属性
   
   ```js
   // 1. 返回单个 toRef(objName || objParentPropName, 'props')
   const person = reactive({
     name: 'Lumina',
     job: {
       j1: {
         name: "ui"
       }
     }
   })
   
   const name = toRef(person, 'name');
   const jobName = toRef(person.job.j1, 'name')
   
   // 2. 返回 toRefs(objName || objParentPropName)
   // 可以利用解构赋值, 使得模板可以直接用对象中的名字来输入
   const person1 = toRefs(person, ['name']);
   
   return {
     name,
     jobName,
   
     person1,
   }
   ```

10. shallowRef()和shallowReactive()
    
    ```js
    // shallowRef(): 不监视和处理对象内的值的修改, 但是可以监视到替换该对象
    const a = shallowRef(0);
    
    // shallowReactive(): 处理对象的浅层属性(类似不开deep)
    const b = shallowReactive({x: 1});
    ```

11. readonly()和shallowreadonly()
    
    ```js
    // readonly(): 数据只读, 不可修改
    const a = readonly(0);
    
    // shallowreadonly(): 仅有对象的浅层属性只读, 深层仍可进行
    const b = shallowreadonly({x: 1});
    ```

12. toRaw()和markRaw()
    
    ```js
    // toRaw(): 将响应式对象(reactive())数据变为非响应式 
    const ref = reactive({a: 123});
    const a = toRaw(ref);
    
    // markRaw(): 标记该对象, 无论这个对象进行任何操作, 都不会变成响应式
    const b = {a: 123, b: 456};
    person.b = b;
    person.b.a = 0; // (不会生效)
    ```

13. customRef(track, trigger): 自定义响应数据逻辑
    
    ```js
    function myRef(val, delay) {
      let timer;
      const m = customRef((track, trigger) => {
        return {
          get() {
            track();
            return val;
          },
          set(newVal) {
            val = newVal;
            if(timer) clearTimeout(timer);
    
            timer = setTimeout(() => {
              trigger();
            }, delay)
          }
        }
      })
    
      return m;
    }
    ```

14. povide_inject: 祖与后代间通信
    
    ```js
    // 父组件
    provide('car', car);
    
    // 后代组件
    const car = inject(car);
    ```

15. vue2必须有根标签; vue3利用fragment, 每个"根标签"都是被一个Fragment虚拟元素中

16. teleport: 将指定元素块, 重新放入页面指定元素块下
    
    ```js
    // to 属性可以写选择器, 如: #id, .class
    <teleport to="body">
        <span>hello</span>
    </teleport>
    ```

17. 
