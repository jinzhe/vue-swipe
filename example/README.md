vue-swipe
=============

![](demo.gif)

基于 Vue.js 的左右滑动组件

## 安装

```sh
npm install --save @zee.kim/vue-swipe
```

## 使用
首先在项目的入口文件中引入, 调用 Vue.use 安装。

```javascript
import vueSwipe from '@zee.kim/vue-swipe'
Vue.use(vueSwipe)
```

```HTML
<div id="app">
    <swipe :autoplay="true" :width="width" :height="height" :items="items"></swipe>
</div>
```

```javascript
import swipe from "./swipe.vue";
import image1 from "./1.jpg";
import image2 from "./2.jpg";
export default {
  name: "app",
  components: {
    swipe
  },
  data() {
    return {
      width: window.innerWidth,
      height: 200,
      items: [image1, image2] //这里是图片数组
    };
  }
};
```