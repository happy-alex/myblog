---
title: koa2中间件机制
date: 2017-12-10
categories: node
tags: node
---

koa2中间件的核心实现：数组 + Promise + 递归

```javascript
// koa1 使用generator
function compose(middleware) {
    return function(context, next) {
        // last called middleware #
        let index = -1;
        return dispatch(0);
        function dispatch(i) {
            if (i <= index)
                return Promise.reject(new Error("next() called multiple times"));
            index = i;

            let fn = middleware[i];

            // 此处的next为undefined
            if (i === middleware.length) fn = next;

            // 如果当前middleware为空,直接返回，不再处理后续middleware
            if (!fn) return Promise.resolve();

            // Promise包裹中间件，递归调用
            try {
                return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
            } catch (err) {
                return Promise.reject(err);
            }
        }
    };
}

const m1 = async (ctx, next) => {
    console.log(1);
    await next();
    console.log(2);
}

const m2 = async (ctx, next) => {
    console.log(3);
    await next();
    console.log(4);
}

const middleware = [m1, m2];

const fnMiddleware= compose(middleware);

// koa的context对象
const ctx = {
    app: {},
    req: {},
    res: {}
};

fnMiddleware(ctx).then(() => {
    console.log('中间件处理结束')
    // const res = ctx.res;
    // res.end(ctx.body);
}).catch(err => {
    console.error(err);
});
```


备注： 上面可以看出，中间件是保存在数组中递归执行的，如果中间件数量过多，可能会带来性能影响。