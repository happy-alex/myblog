---
title: suspense
date: 2018-03-30
categories: js
tags: react
---

Dan在今年JSConf的会议上提到React未来的两大特性：Time Slicing(时间分片) 和 Suspense(渲染暂停)。
本文是根据Dan的ppt整理翻译。

## 背景
影响页面用户体验的两大因素分别是CPU计算和I/O能力。前者影响DOM元素的创建和更新，后者影响数据的获取和代码的加载。
Time Slicing 和 Suspense 分别是React在这两方面的尝试。

## Time Slicing
Time Slicing 是React16+采用的页面异步刷新方案
通过采用时间分片的方式，确保像用户输入等高优先级的任务先执行，render等低优先级的任务后执行。

## Suspense

简单Demo:
```javascript
// fetchMovieDetails为业务请求方法，返回promise
const movieDetailsFetcher = createFetcher(fetchMovieDetails);
function MovieDetails(props) {
    const movie = movieDetailsFetcher.read(props.id);
    return (
        <div className="MovieDetails">
            <h1>{movie.header}</h1>
            <Movie {...movie} />
        </div>
    );
}
```

### 简单介绍
在渲染过程中可以发起异步请求，获取数据，具体的请求被createFetcher方法(由react实现)包裹。

react优先检查缓存，如果有缓存，直接命中缓存；如果没有缓存，抛出一个异常的promise

当promise完成后，即数据请求成功，react继续执行render函数，渲染出页面。实际上是重新执行一遍render函数。

从上面的流程可以看出来，suspense赋予了react随意暂停/恢复渲染的功能，从而将传统的异步请求数据的方式都改由同步来实现。

执行render -> 发现异步请求 -> 检查缓存  ->  无缓存  ->render暂停，等待请求  -> 请求回来，重新执行render 
                                   |         
                                    ->  有缓存  -> 直接渲染

### 相关特性：
1. 在数据准备好之前可以暂停任意state更新
2. 无需其他“管道”, 可给任意组件添加任意异步数据
3. 网络状况良好的环境下，render在整棵树准备好之后
4. 网络状况差的环境下，能精确控制加载状态
5. 这既是一个高级API 也是一个低级API

## 后记
React16.6已经支持部分suspense特性
```javascript
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // Displays <Spinner> until OtherComponent loads
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```
主要用来处理loading的相关场景，允许组件延迟渲染，中间等待状态可以通过fallback来指定渲染内容
react组件提供原生支持，避免了之前在处理相关场景时要写大量的条件判断代码，React.Suspense可以出现在组件树的任意位置，更方便控制loading的粒度

注意：React.lazy和React.suspense总是搭配在一起使用，且不支持ssr，如果需要支持ssr，可以考虑 @loadable/component 这个库代替