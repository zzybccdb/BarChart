# BarChart
- 這是一份由 d3 和 pixi.js 一起實作完成的 BarChart 圖表
- 資料給定的格式爲
```javascript
    let response = {
        'data':{
        'test1':{
            'bin_edges':[0,1,2,3,4,5],
            'hist':[1,2,3,2,1],
            'error':false,
        },
        'test2':{
            'bin_edges':[0,1,2,3,4,5],
            'hist':[1,2,3,2,1],
            'error':false,
        },
        'test3':{
            'bin_edges':[0,1,2,3,4,5],
            'hist':[1,2,3,2,1],
            'error':false,                  
        }
        },
        'columns':['test1','test2','test3']
    }
```

- 圖表的 layout 可以由自由修改，charNum 表示的是每個 row 需要呈現的圖表的個數
```javascript
    vm.layout = {
        margin:{
            l:20,
            r:20,
            b:20,
            t:20,
        },
        height: 100,
        chartNum:3
    }
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

- 結果 Demo
- ![](https://i.imgur.com/GbUhHwQ.png)