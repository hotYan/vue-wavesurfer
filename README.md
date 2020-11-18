# vue-wavesurfer

## 官网
wavesurfer.js 官网[点这里](https://wavesurfer-js.org/)

<br/>

## 学习小结
在Vue中使用音频可视化插件wavesurfer.js[点这里](https://juejin.im/post/6862201364308492302)

<br/>

## 使用
将代码拷贝到可运行的vue项目中，将组件添加到路由文件：
```js
// 例如
{
  path: '/audio',
  component: Layout,
  redirect: '/audio/index',
  children: [
    {
      path: 'index',
      component: () => import('@/components/ZyAudio/index.vue'),
      name: 'AudioIndex',
    }
  ]     
},
```