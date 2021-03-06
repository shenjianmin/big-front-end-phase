---
title: NuxtJS案例 - Realworld项目 - 首页
date: 2021-01-08 20:14:29
sidebar: 'auto'
tags:
 - Vue.js
 - Part3·Vue.js 框架源码与进阶
 - NuxtJS
categories:
 - 大前端
publish: true 
isShowComments: false
---


## 11.4 首页
### 公共文章列表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107161709319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

请求方法：`GET `

请求地址：`/api/articles`

查询参数：

- 按标签筛选
	* `?tag=AngularJS`
- 按作者筛选
	* `?author=jake`
- 按用户筛选
	* `?favorited=jake`
- 文章分页数（默认20）
	* `?limit=20`
- 文章偏移/跳跃数（默认0）
	* `?offset=0`

`api/article.js`
```js
import request from '@/utils/request'

// 获取公共文章列表
export const getArticles = params => {
  return request({
    method: 'GET',
    url: '/api/articles',
    params
  })
}
```

`pages/home/index.vue`
```js
import { getArticles } from '@/api/article'

export default {
  name: 'HomeIndex',
  async asyncData () {
    const { data } = await getArticles()
    console.log(data)
  }
}
```
![在 这里插入图片描述](https://img-blog.csdnimg.cn/20210107165413776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107165555802.png)

```js
export default {
  name: 'HomeIndex',
  async asyncData () {
    const { data } = await getArticles()
    return {
      articles: data.articles,
      articlesCount: data.articlesCount
    }
  }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107170044994.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

遍历数据：
```js
<div
  class="article-preview"
  v-for="article in articles"
  :key="article.slug"
>
  <div class="article-meta">
    <!-- <a href="profile.html"><img src="http://i.imgur.com/Qr71crq.jpg" /></a> -->
    <nuxt-link :to="{
      name: 'profile',
      params: {
        username: article.author.username
      }
    }">
      <img :src="article.author.image" />
    </nuxt-link>
    <div class="info">
      <!-- <a href="" class="author">Eric Simons</a> -->
      <nuxt-link class="author" :to="{
        name: 'profile',
        params: {
          username: article.author.username
        }
      }">
        {{ article.author.username }}
      </nuxt-link>
      <span class="date">{{ article.createdAt }}</span>
    </div>
    <button class="btn btn-outline-primary btn-sm pull-xs-right"
      :class="{
          active: article.favorited
        }">
      <i class="ion-heart"></i> {{ article.favoritesCount }}
    </button>
  </div>
  <!-- <a href="" class="preview-link"> -->
  <nuxt-link 
    class="preview-link" 
    :to="{
      name: 'article',
      params: {
        slug: article.slug
      }
    }">
    <h1>{{ article.title }}</h1>
    <p>{{ article.description }}</p>
    <span>Read more...</span>
  <!-- </a> -->
  </nuxt-link>  
</div>
```

### 列表分页-分页参数的使用
`pages/home/index.vue`
```js
export default {
  name: 'HomeIndex',
  async asyncData () {
    const page = 1
    const limit = 10
    const { data } = await getArticles({
      // 文章分页数（默认20）
      limit,
      // 文章偏移/跳跃数（默认0）
      offset: (page - 1) * limit
    })
    return {
      articles: data.articles,
      articlesCount: data.articlesCount
    }
  }
}
```
### 列表分页-页码处理
添加页码，页码改变获取对应页码的数据：`pages/home/index.vue`
```js
<!-- 分页列表 -->
<nav>
  <ul class="pagination">

    <li class="page-item active">

      <a class="page-link ng-binding" href="">1</a>

    </li>
  </ul>
</nav>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107221633509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

```js
export default {
  name: 'HomeIndex',
  async asyncData ({ query }) {
    const page = Number.parseInt(query.page || 1)
    const limit = 10
    const { data } = await getArticles({
      // 文章分页数（默认20）
      limit,
      // 文章偏移/跳跃数（默认0）
      offset: (page - 1) * limit
    })
    return {
      articles: data.articles,
      articlesCount: data.articlesCount,
      page,
      limit
    }
  },
  computed: {
    // 总页码
    totalPage () {
      return Math.ceil(this.articlesCount / this.limit)
    }
  }
}
```
```js
<!-- 分页列表 -->
<nav>
  <ul class="pagination">

    <li 
      class="page-item"
      :class="{
        active: item === page
      }"
      v-for="item in totalPage"
      :key="item"  
    >

      <!-- 默认query的改变不会调用asyncData方法。
      如果要监听这个行为，可以设置应通过页面组建的watchQuery参数监听参数 -->
      <nuxt-link 
        class="page-link" 
        :to="{
          name: 'home',
          query: {
            page: item
          }
        }"
      >{{ item }}</nuxt-link>

    </li>
  </ul>
</nav>
```
```js
export default {
  ...
  watchQuery:['page']
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107223834814.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107223854228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 文章标签列表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107224221660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

请求方法：`GET `

请求地址：`/api/tags`

`ap/tag.js`
```js
import request from '@/utils/request'

// 获取文章标签列表
export const getTags = () => {
  return request({
    method: 'GET',
    url: '/api/tags',
  })
}
```

`pages/home/index.vue`
```js
...
import { getTags } from '@/api/tag'

export default {
  name: 'HomeIndex',
  async asyncData ({ query }) {
    ...
    const { data: tagData } = await getTags()
    return {
      ...
      tags: tagData.tags
    }
  },
  ...
}
```
```js
<div class="col-md-3">
  <div class="sidebar">
    <p>Popular Tags</p>

    <div class="tag-list">
      <a 
        href="" 
        class="tag-pill tag-default"
        v-for="item in tags"
        :key="item"
      >{{ item }}</a>
    </div>
  </div>
</div>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107225235340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 优化并行异步任务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107225358272.png)

对于以上两个异步操作：获取文章列表和获取标签列表。它们的执行顺序是先拿到文章列表，再去请求获取标签列表，是串行的方式。

对于多个异步任务，它们之间并没有依赖关系，因此我们可以将多个异步任务并发执行。通过并行来提高获取请求加载的速度。

`pages/home/index.vue`
```js
export default {
  name: 'HomeIndex',
  async asyncData ({ query }) {
    const page = Number.parseInt(query.page || 1)
    const limit = 10
    const [ articleRes, tagRes ] = await Promise.all([
      getArticles({
        limit, // 文章分页数（默认20）
        offset: (page - 1) * limit // 文章偏移/跳跃数（默认0）
      }),
      getTags()
    ])

    const { articles, articlesCount } = articleRes.data
    const { tags } = tagRes.data

    return {
      articles,
      articlesCount,
      tags,
      page,
      limit
    }
  },
  ...
}
```

优化前：4.37s - 7.42s 不等
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021010723052586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107230626470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

优化后：2.68s - 3.98s 不等
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107230852226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107230810299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

经过多次尝试后，发现优化后的代码整体比优化前要快一些（网络对速度有一定影响），甚至有时候不到两秒就可以完成。

### 标签列表链接和数据
`pages/home/index.vue`
```js
<div class="col-md-3">
  <div class="sidebar">
    <p>Popular Tags</p>

    <div class="tag-list">
      <nuxt-link 
        :to="{
          name: 'home',
          query: {
            tag: item
          }
        }" 
        class="tag-pill tag-default"
        v-for="item in tags"
        :key="item"
      >{{ item }}</nuxt-link>
    </div>
  </div>
</div>
```
```js
watchQuery:['page', 'tag'],
```
对应数据：
```js
getArticles({
  limit, // 文章分页数（默认20）
  offset: (page - 1) * limit, // 文章偏移/跳跃数（默认0）
  tag: query.tag // 按标签筛选
})
```
解决标签和分页共存问题：
```js
<!-- 分页列表 -->
<nav>
  <ul class="pagination">

    <li 
      class="page-item"
      :class="{
        active: item === page
      }"
      v-for="item in totalPage"
      :key="item"  
    >

      <!-- 默认query的改变不会调用asyncData方法。
      如果要监听这个行为，可以设置应通过页面组建的watchQuery参数监听参数 -->
      <nuxt-link 
        class="page-link" 
        :to="{
          name: 'home',
          query: {
            page: item,
            tag: $route.query.tag
          }
        }"
      >{{ item }}</nuxt-link>

    </li>
  </ul>
</nav>
<!-- /分页列表 -->
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107234104193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 导航栏-展示状态处理
补全三个导航栏，示例：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021010815330948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

`pages/home/index.vue`
```js
<div class="feed-toggle">
  <ul class="nav nav-pills outline-active">
    <li class="nav-item">
      <a class="nav-link disabled" href="">Your Feed</a>
    </li>
    <li class="nav-item">
      <a class="nav-link active" href="">Global Feed</a>
    </li>
    <li class="nav-item">
      <a class="nav-link active" href="">#标签</a>
    </li>
  </ul>
</div>
```
业务逻辑：

- Your Feed：只有在登录状态下才被展示
- Global Feed：不论登录与否均被展示
- #标签：只有在点击某个标签才被展示，切换回Your Deed或Global Feed后不再展示

```js
import { mapState } from 'vuex'

export default {
  name: 'HomeIndex',
  async asyncData ({ query }) {
    ...
    const { tag } = query
    const [ articleRes, tagRes ] = await Promise.all([
      getArticles({
        ...
        tag // 按标签筛选
      }),
      ...
    ])

    ...

    return {
      ...
      tag
    }
  },
  computed: {
    ...mapState(['user']),
    ...
  }
}
```
```js
<li v-if="user" class="nav-item">
  <a class="nav-link disabled" href="">Your Feed</a>
</li>

...

<li v-if="tag" class="nav-item">
  <a class="nav-link active" href="">#{{ tag }}</a>
</li>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021010815514795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108155202655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108155236843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 导航栏-标签高亮及链接
导航链接优化：`pages/home/index.vue`
```js
async asyncData ({ query }) {
  ...

  return {
    ...
    tab: query.tab || 'global_feed'
  }
},
watchQuery:['page', 'tag', 'tab'],
...
```
```js
<div class="feed-toggle">
  <ul class="nav nav-pills outline-active">
  
    <li v-if="user" class="nav-item">
      <!-- exact 精确匹配 -->
      <nuxt-link 
        class="nav-link"
        :class="{
          active: tab === 'your_feed'
        }"
        exact
        :to="{
          name: 'home',
          query: {
            tab: 'your_feed'
          }
        }"
      >Your Feed</nuxt-link>
    </li>
    
    <li class="nav-item">
      <nuxt-link 
        class="nav-link"
        :class="{
          active: tab === 'global_feed'
        }"
        exact
        :to="{
          name: 'home',
          query: {
            tab: 'global_feed'
          }
        }"
      >Global Feed</nuxt-link>
    </li>
    
    <li v-if="tag" class="nav-item">
      <nuxt-link 
        class="nav-link"
        :class="{
          active: tab === 'tag'
        }"
        exact
        :to="{
          name: 'home',
          query: {
            tab: 'tag',
            tag: tag
          }
        }"
      >#{{ tag }}</nuxt-link>
    </li>
    
  </ul>
</div>
```
修改分页列表，添加`tab: tab`
```js
<nuxt-link 
  class="page-link" 
  :to="{
    name: 'home',
    query: {
      page: item,
      tag: $route.query.tag,
      tab: tab
    }
  }"
>{{ item }}</nuxt-link>
```
修改标签列表，添加`tab: 'tag'`
```js
<div class="col-md-3">
  <div class="sidebar">
    <p>Popular Tags</p>

    <div class="tag-list">
      <nuxt-link 
        :to="{
          name: 'home',
          query: {
            tab: 'tag',
            tag: item
          }
        }" 
        class="tag-pill tag-default"
        v-for="item in tags"
        :key="item"
      >{{ item }}</nuxt-link>
    </div>
  </div>
</div>
```

### 导航栏-用户关注的文章列表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108170533552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

请求方法：`GET `

请求地址：`/api/articles/feed`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108171603617.png)

身份验证标头：
```js
Authorization: Token jwt.token.here
```

重新登录获取一个新Token：（长时间不使用会过期）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108173609573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

`api/article.js`
```js
...

// 获取关注的用户文章列表
export const getFeedArticles = params => {
  return request({
    method: 'GET',
    url: '/api/articles/feed',
    // Authorization: Token jwt.token.here
    headers: {
      // 注意数据格式：Token空格数据Token
      // 先手动写死，自动处理后续介绍
      Authorization: `Token eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
        .eyJpZCI6MTMzMjY4LCJ1c2VybmFtZSI6InNoaWd1YW5naGFpIiwiZXhwIjoxNjE1MjgyMzcxfQ
        ._QB6VDwPXLFSa30IScJ1q5Q_1zk5FXB99OYAKU3qMW0`
    },
    params
  })
}
```
`pages/home/index.vue`
```js
async asyncData ({ query, store }) {
  const page = Number.parseInt(query.page || 1)
  const limit = 10
  const { tag } = query
  const tab = query.tab || 'global_feed'
  const loadArticles = store.state.user && tab === 'your_feed'
    ? getFeedArticles
    : getArticles

  const [ articleRes, tagRes ] = await Promise.all([
    loadArticles({
      limit, // 文章分页数（默认20）
      offset: (page - 1) * limit, // 文章偏移/跳跃数（默认0）
      tag // 按标签筛选
    }),
    getTags()
  ])

  const { articles, articlesCount } = articleRes.data
  const { tags } = tagRes.data

  return {
    articles,
    articlesCount,
    tags,
    page,
    limit,
    tag,
    tab
  }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108173444118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021010817343062.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 统一设置用户Token
接下来把手动写死的接口中数据Token的方式变成自动的，让它通过Cookie来拿到用户的Token。

我们可以使用[ axios 拦截器 ](https://github.com/axios/axios#interceptors)的方式来实现：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108174451179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
```js
// Add a request interceptor
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  }, function (error) {
    // Do something with request error
    return Promise.reject(error);
  });

// Add a response interceptor
axios.interceptors.response.use(function (response) {
    // Any status code that lie within the range of 2xx cause this function to trigger
    // Do something with response data
    return response;
  }, function (error) {
    // Any status codes that falls outside the range of 2xx cause this function to trigger
    // Do something with response error
    return Promise.reject(error);
  });  
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108171603617.png)

`utils/request.js`
```js
// 请求拦截器
// Add a request interceptor
// 任何请求都要经过请求拦截器
// 我们可以在请求拦截器中做一些公共的业务处理，例如统一设置 Token
// config 包括本次请求的请求地址、方法、参数等等
request.interceptors.request.use(function (config) {
  // 请求就会经过这里（正确）
  // Do something before request is sent
  // console.log('123')
  config.headers.Authorization = `Token 用户Token`

  // 返回 config 请求配置对象
  return config
}, function (error) {
  // 如果请求失败（此时请求还没有发出去）就会进入这里
  // Do something with request error
  return Promise.reject(error)
})
````

此时考虑如何获取 用户Token：

> Nuxt.js 允许您在运行Vue.js应用程序之前执行js插件。这在您需要使用自己的库或第三方模块时特别有用。

我们可以通过[ 插件 ](https://www.nuxtjs.cn/guide/plugins)获取到处理之后的store上下文对象：`plugins/request.js`
```js
import axios from 'axios'

// 创建请求对象
const request = axios.create({
  baseURL: 'https://conduit.productionready.io/'
})

// 通过插件机制获取到上下文对象（query、params、req、res、app、store...）
export default (context) => {
  console.log(context)
}
```

注册插件：`nuxt.config.js`
```js
module.exports = {
  ...
  
  // 注册插件
  plugins: [
    '~/plugins/request.js'
  ]
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108182048927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)


`plugins/request.js`
```js
/**
 * 基于 axios 封装的请求模块
 */

import axios from 'axios'

// 创建请求对象
export const request = axios.create({
  baseURL: 'https://conduit.productionready.io/'
})

// 通过插件机制获取到上下文对象（query、params、req、res、app、store...）
// 插件导出函数必须作为 default 成员
export default ({ store }) => {
  // 请求拦截器
  // Add a request interceptor
  // 任何请求都要经过请求拦截器
  // 我们可以在请求拦截器中做一些公共的业务处理，例如统一设置 Token
  // config 包括本次请求的请求地址、方法、参数等等
  request.interceptors.request.use(function (config) {
    // 请求就会经过这里（正确）
    // Do something before request is sent
    const { user } = store.state

    if (user && user.token){
      config.headers.Authorization = `Token ${user.token}`
    }

    // 返回 config 请求配置对象
    return config
  }, function (error) {
    // 如果请求失败（此时请求还没有发出去）就会进入这里
    // Do something with request error
    return Promise.reject(error)
  })
}
```

更改请求地址：`api/[article.js, tag.js, user.js]`
```js
import { request } from '@/plugins/request'
```

`api/article.js`
```js
// 获取关注的用户文章列表
export const getFeedArticles = params => {
  return request({
    method: 'GET',
    url: '/api/articles/feed',
    // Authorization: Token jwt.token.here
    // headers: {
    //   // 注意数据格式：Token空格数据Token
    //   // 先手动写死，自动处理后续介绍
    //   Authorization: `Token eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
    //     .eyJpZCI6MTMzMjY4LCJ1c2VybmFtZSI6InNoaWd1YW5naGFpIiwiZXhwIjoxNjE1MjgyMzcxfQ
    //     ._QB6VDwPXLFSa30IScJ1q5Q_1zk5FXB99OYAKU3qMW0`
    // },
    params
  })
}
```
最后删除`utils/request.js`文件，此时仍然可以访问Your Feed
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108183126205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 文章发布时间格式化处理
预期效果：

**2021-01-08T09:43:43.952Z**  ->  **January 8, 2021**

[dayjs](https://github.com/iamkun/dayjs)：是一个轻量的处理时间和日期的 JavaScript 库

安装`Day.js`
```shell
npm install dayjs --save
```

通过插件来扩展Vue的资源，封装为全局过滤器来使用：`plugins/dayjs.js`
```js
import Vue from 'vue'
import dayjs from 'dayjs'

// {{ 表达式 | 过滤器 }}
Vue.filter('date', (value, format = 'YYYY-MM-DD HH:mm:ss') => {
  return dayjs(value).format(format)
})
```

注册插件：`nuxt.config.js`
```js
plugins: [
  ...
  '~/plugins/dayjs.js'
]
```

使用插件：`pages/home/index.vue`
```js
<span class="date">{{ article.createdAt | date }}</span>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108192559265.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

参考[ 文档 ](https://dayjs.gitee.io/docs/zh-CN/display/format)修改为我们预期的格式：
```js
<span class="date">{{ article.createdAt | date('MMMM DD, YYYY') }}</span>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108193126466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

### 文章点赞
最后我们来处理一下文章点赞的功能：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108193402275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)
添加点赞： 

请求方法：`POST `

请求地址：`/api/articles/:slug/favorite`

删除点赞：

请求方法：`DELETE `

请求地址：`/api/articles/:slug/favorite`

封装点赞功能：`api/article.js`
```js
// 添加点赞
export const addFavorite = slug => {
  return request({
    method: 'POST',
    url: `/api/articles/${slug}/favorite`
  })
}

// 取消点赞
export const deleteFavorite = slug => {
  return request({
    method: 'DELETE',
    url: `/api/articles/${slug}/favorite`
  })
}
```

`pages/home/index.vue`
```js
<button class="btn btn-outline-primary btn-sm pull-xs-right"
  :class="{
    active: article.favorited
  }"
  @click="onFavorite(article)">
  <i class="ion-heart"></i> {{ article.favoritesCount }}
</button>
```
```js
import { 
  ...
  addFavorite,
  deleteFavorite
} from '@/api/article'

export default {
  methods: {
    async onFavorite (article) {
      if (article.favorited) {
        // 取消点赞
        await deleteFavorite(article.slug)
        article.favorited = false
        article.favoritesCount -= 1
      } else {
        // 添加点赞
        await addFavorite(article.slug)
        article.favorited = true
        article.favoritesCount += 1
      }
    }
  }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108195215818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MTQ5MjU2,size_16,color_FFFFFF,t_70)

但是这里还有一个小问题：如果网络比较慢，用户频繁点击按钮，可能会导致期间来回处理导致出现错误。

因此我们应该在请求期间禁用此按钮：
```js
async asyncData ({ query, store }) {
  ...
  
  // 默认未禁用
  articles.forEach(article => article.favoriteDisabled = false)
},
...
methods: {
  async onFavorite (article) {
    article.favoriteDisabled = true
    if (article.favorited) {
      // 取消点赞
      await deleteFavorite(article.slug)
      article.favorited = false
      article.favoritesCount -= 1
    } else {
      // 添加点赞
      await addFavorite(article.slug)
      article.favorited = true
      article.favoritesCount += 1
    }
    article.favoriteDisabled = false
  }
}
```
```js
<button class="btn btn-outline-primary btn-sm pull-xs-right"
  :class="{
    active: article.favorited
  }"
  @click="onFavorite(article)"
  :disabled="article.favoriteDisabled">
  <i class="ion-heart"></i> {{ article.favoritesCount }}
</button>
```

至此，首页部分就以实现完毕