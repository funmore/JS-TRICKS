# 什么的异步文件

## 定义：通过异步方式加载的文件

最常见的配置1 在router.js里

```text
  { path: '/login', component: () => import('@/views/login/index'), hidden: true }

```

使用component : \(\) =&gt; import\('@/somepath'\)将会引入一个codesplie 属性的实践，并在最终打包文件中引入一个被拆分的文件

