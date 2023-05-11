# Welcome to [Slidev](https://github.com/slidevjs/slidev)!

To start the slide show:

- `npm install`
- `npm run dev`
- visit http://localhost:3030

Edit the [slides.md](./slides.md) to see the changes.

Learn more about Slidev on [documentations](https://sli.dev/).

# 大纲

* 从一个问题开始
* 一步一步引入函数式编程思想
* 问题的答案是：变成可靠输入，而不是副作用
* 官方答案：useSyncExternalStore
* 引入主角swr
* 引入配角ahooks的useRequst
* 几个现成的例子
* 函数式编程的好处
* 总结
* one more thing

# 部署


## 修改基础路径

目标地址：https://jianghong.site/presentation/swr，首先因为slidev用的是history路由，会修改路由地址。如：部署在https://jianghong.site/presentation/swr，则slidev在运行后会修改路由为：https://jianghong.site/1，这样刷新后就离开了slidev的范围。

slidev用的是vite，支持用base参数修改[公共基础路径](https://cn.vitejs.dev/config/shared-options.html#base)。

则需要在根目录新建`vite.config.js`，写入：

```js
export default {
    base: '/presentation/swr/',
}
```

即可，slidev修改路由时会基于`/presentation/swr/`来修改：https://jianghong.site/presentation/swr/1

## 上传cos

运行`npm run build`后，把`dist`目录下的内容上传到cos的桶中。路径也存在`/presentation/swr`：https://presentation-1258538316.cos.ap-guangzhou.myqcloud.com/presentation/swr/index.html

这时候直接访问也能使用。但是路径会被修改为：https://presentation-1258538316.cos.ap-guangzhou.myqcloud.com/presentation/swr/1，刷新后会404。不能直接使用。

## nginx

nginx需要做两个转发：把`/presentation/swr/*`转发到`https://presentation-1258538316.cos.ap-guangzhou.myqcloud.com/presentation/swr/index.html`。还要把`/presiontation/swr/assets/*`转发到`https://presentation-1258538316.cos.ap-guangzhou.myqcloud.com/presentation/swr/assets/*`;

经过多次尝试，最终配置如下：

```shell
location ~ ^/presentation/swr/assets(/.*)?$ {
    rewrite ^/presentation/swr/assets(/.*)?$ /presentation/swr/assets/$1 break;
    proxy_pass https://presentation-1258538316.cos.ap-guangzhou.myqcloud.com;
}

location ~ ^/presentation/swr(/.*)?$ {
    rewrite ^/presentation/swr(/.*)?$ /presentation/swr/index.html break;
    proxy_pass https://presentation-1258538316.cos.ap-guangzhou.myqcloud.com;
}

```
`(.*)`通配所有长度的任意字符，`^/presentation/swr(/.*)?$`都转发到`index.html`。

`$1`代表正则里括号匹配到的内容。

    rewrite ^/presentation/swr/assets(/.*)?$ /presentation/swr/assets/$1 break;

会把assets/后面转发到同名的新地址。
/presentation/swr/assets/index.js => /presentation/swr/assets/index.js
/presentation/swr/assets/index.css => /presentation/swr/assets/index.css

转发assets的配置应该要写在转发到index.html的前面。否则转发到index.html的配置会覆盖转发assets的配置，导致assets里的静态内容都转发到index.html下。

