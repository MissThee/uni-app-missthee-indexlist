# uni-app 索引列表

### 使用
```
<template>
  <missthee-indexlist :listData="listData" @select-item='selectHandler'></missthee-indexlist>
</template>

<script>
  export default {
    data() {
      return {
        listData: {
          "A": [{
            id: 1,  //城市id，作为v-for的key使用
            name: "A市",  //名称，作为显示字段
          },],
          "B": [{
            id: 2,
            name: "B市",
          },
          {
            id: 3,
            name: "B1市",
          },],
          "C": [{
            id: 4,
            name: "C市",
          },],
          "D": [{
            id: 5,
            name: "D市",
          },],
          "E": [{
            id: 6,
            name: "E市",
          },],
          "F": [{
            id: 7,
            name: "F市",
          },],
          "G": [{
            id: 8,
            name: "G市",
          },],
          "H": [{
            id: 9,
            name: "淮北市",
          },],
          "I": [{
            id: 10,
            name: "淮北市",
          },],
          "J": [{
            id: 11,
            name: "J市",
          },],
          "K": [{
            id: 12,
            name: "K市",
          },],
          "L": [{
            id: 13,
            name: "L市",
          },],
          "M": [{
            id: 14,
            name: "M市",
          },],
          "N": [{
            id: 15,
            name: "N市",
          },],
          "O": [{
            id: 16,
            name: "O市",
          },],
          "P": [{
            id: 17,
            name: "P市",
          },],
          "Q": [{
            id: 18,
            name: "Q市",
          },],
          "R": [{
            id: 19,
            name: "R市",
          },],
          "S": [{
            id: 20,
            name: "S市",
          },],
          "T": [{
            id: 21,
            name: "T市",
          },],
          "U": [{
            id: 22,
            name: "U市",
          },],
          "V": [{
            id: 23,
            name: "V市",
          },],
          "W": [{
            id: 24,
            name: "W市",
          },],
          "X": [{
            id: 25,
            name: "X市",
          },],
          "Y": [{
            id: 26,
            name: "Y市",
          },],
          "Z": [{
            id: 27,
            name: "Z市",
          },],
          "#": [{
            id: 28,
            name: "123市",
          },{
            id: 29,
            name: "456市",
          },],
        }
      }
    },
    methods: {
      selectHandler(e) {
        console.log('选中城市', e) // 返回选中对象 {id: 1, name: "A市"}
      }
    }
  }
</script>
````
|属性|默认值|说明|
|:---:|:---:|:---:|
|data|{}|绑定列表数据|
|placeholder|输入关键字查询|顶部查询框placeholder|
|useIndex|true|启用右侧索引|

|事件|说明|
|:---:|:---:|
|select-item|点击列表内容触发，返回选中的对象|

