<template>
    <!-- <v-layout ref='layout'> -->
        <div style="width:100%;" ref='histogram'></div>
    <!-- </v-layout> -->
</template>
<script>
let vm = undefined
let PIXI = undefined
let d3 = undefined

// 需要注意一个问题，当 canvas 长度为 16500 时就会出现 resolution 错误的问题

export default {
    components: {
    },

    data: () => {
        return {
        };
    },
    created(){
        let vm = this
        vm.app = undefined
    },

    methods:{
        // 加载资料
        // response data 包含 hist（纵轴）， bin_edges（横轴）
        loadData(){
            // 輸入測試 資料
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
            vm.data = response.data
            vm.columns = response.columns
            vm.count = vm.columns.length
            vm.pixiInit()
            vm.d3Init(vm.data)
            vm.drawGraph()
        },
        // PIXI初始化设定,视窗绑定
        pixiInit(){
            // PIXI.settings.PRECISION_FRAGMENT= 'highp'
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
            vm.appWidth = vm.$refs.histogram.clientWidth
            // Math.ceil(number) 向上取整
            vm.appHeight = vm.layout.height * Math.ceil(vm.count/vm.layout.chartNum)
            // vm.root.$refs.histWrapper.style.height = vm.appHeight
            // 初始化绘图内容
            vm.app = new PIXI.Application({
                autoResize:true,
                backgroundColor: 0xffffff, //白色背景畫布
                antialias:false,
                transparent: false,
                resolution:1,
            })
            vm.app.renderer.roundPixels = true
            vm.app.renderer.view.style.display = 'block'
            vm.app.renderer.resize(vm.appWidth,vm.appHeight)
            // 將圖表加入 DOM tree
            vm.$refs.histogram.appendChild(vm.app.view)
            // 圖表整體外包裝
            vm.wrapper = new PIXI.Container()
            vm.wrapper.name = 'histogram_wrapper'
            vm.wrapper.x = 0
            vm.wrapper.y = 0 
            // 將包裝紙加入畫布
            vm.app.stage.addChild(vm.wrapper)
            // 主體容器 
            vm.ctn  = new PIXI.Container()
            vm.ctn.name = 'main_ctn'
            vm.wrapper.addChild(vm.ctn)

        },      
        // d3初始化设定,绑定资料和 scale
        d3Init(data){
            vm.dimensions = {}
            // 每行绘制 4 个
            let width = vm.appWidth / vm.layout.chartNum
            let height = vm.layout.height
            for(let d in data){
                let temp = {}
                if(!data[d].error){
                    temp.hist_data = data[d].hist 
                    temp.bin_edges_data = data[d].bin_edges
                    console.log(temp)
                    temp.hist_extent = d3.extent(temp.hist_data)
                    temp.bin_edges_extent = d3.extent(temp.bin_edges_data)
                    temp.hist_scale = d3.scaleLinear()
                                        .domain(temp.hist_extent)
                                        .range([vm.layout.margin.t,height-vm.layout.margin.b-vm.layout.margin.t])
                    console.log([vm.layout.margin.t,height-vm.layout.margin.b-vm.layout.margin.t])
                    temp.bin_edges_scale = d3.scaleLinear()
                                        .domain(temp.bin_edges_extent)
                                        .range([2*vm.layout.margin.l,width-vm.layout.margin.r-20])
                    temp.ctn = new PIXI.Container()
                    temp.ctn.name = d
                    temp.error = false
                    vm.ctn.addChild(temp.ctn)
                }
                else{
                    temp.error = true
                    temp.ctn = new PIXI.Container() 
                    temp.ctn.name = d
                }
                vm.dimensions[d] = temp
            }
        },
        // 開始繪製
        drawGraph(){
            vm.columns.forEach((label,i) => {
                let temp = vm.dimensions[label]
                let gap = vm.layout.chartNum
                // 坐标原点的起始坐标
                let orgx = vm.layout.margin.l*2
                let orgy = vm.layout.margin.t
                //  单张 histogram 的长宽
                let graphWidth = vm.appWidth/vm.layout.chartNum-vm.layout.margin.l
                let graphHeight = vm.layout.height-vm.layout.margin.t-vm.layout.margin.b
                // 每张图表的位移位置
                let x = vm.layout.margin.t + (0===i%gap?0:(i%gap)*vm.appWidth/vm.layout.chartNum)
                let y = vm.layout.margin.l + parseInt(i/gap) * vm.layout.height
                if(!temp.error){
                    let axis_ctn = vm.drawAxis(label,temp.ctn.name,orgx,orgy,graphWidth,graphHeight)
                    let hist_ctn = vm.drawHist(label,graphHeight+vm.layout.margin.t)
                    temp.ctn.addChild(axis_ctn)
                    temp.ctn.addChild(hist_ctn)
                } 
                else{
                    let text = vm.Text(temp.ctn.name+' DATA ERROR',15)
                    text.x = vm.appWidth/(vm.layout.chartNum*2)-text.width/2
                    text.y = vm.layout.height/2-text.height/2
                    temp.ctn.addChild(text)
                    vm.ctn.addChild(temp.ctn)
                }
                temp.ctn.x = x 
                temp.ctn.y = y
            });
        },
        // 绘制轴线
        drawAxis(label,name,orgx,orgy,graphWidth,graphHeight){
            let axis_ctn = new PIXI.Container()
            // Label name 
            let text = vm.Text(name,15)
            text.y = 0
            text.x = vm.appWidth/(vm.layout.chartNum*2)-text.width/2
            let yAxis = vm.drawSolidLine(orgx,orgy,orgx,graphHeight)
            let xAxis = vm.drawSolidLine(orgx,graphHeight,graphWidth,graphHeight)
            // 绘制刻度
            let xTicks = vm.drawxTicks(label,0,graphHeight,orgx)
            let yTicks = vm.drawyTicks(label,10,graphHeight+vm.layout.margin.t,orgx)
            
            axis_ctn.addChild(text)
            axis_ctn.addChild(yAxis)    
            axis_ctn.addChild(xAxis)
            axis_ctn.addChild(xTicks)
            axis_ctn.addChild(yTicks)
            return axis_ctn
        },
        // 繪製Hist
        drawHist(label,y){
            let hist_ctn = new PIXI.Container()
            let hist = vm.dimensions[label].hist_data
            let bin_edges = vm.dimensions[label].bin_edges_data
            let hist_scale = vm.dimensions[label].hist_scale
            let bin_scale = vm.dimensions[label].bin_edges_scale
            hist.forEach((h,i) => {
                let box = new PIXI.Graphics()
                let x1 = bin_scale(bin_edges[i])
                let y1 = y-hist_scale(h)
                let w = bin_scale(bin_edges[i+1])-x1
                let height = hist_scale(h)-vm.layout.margin.t
                box.lineStyle(2, 0x3273b2, 1);
                box.beginFill(0x98b9d8)
                box.drawRect(x1,y1,w,height);
                box.endFill()
                hist_ctn.addChild(box)
            })
            return hist_ctn
        },  
        // 绘制刻度
        // x 轴固定 4 个 ticks
        drawxTicks(label,x,y){
            let ticks_ctn = new PIXI.Container()
            let format1 = d3.format('.2s')
            let format2 = d3.format('.2f')
            let scale = vm.dimensions[label].bin_edges_scale
            let ticks = scale.ticks(5)
            ticks.forEach(tick =>{
                let text = undefined
                if(tick > 1)
                    text = vm.Text(format1(tick),10)
                else
                    text = vm.Text(format2(tick),10)
                text.y = y+10
                text.x = x + scale(tick) 
                let line = vm.drawSolidLine(text.x,y,text.x,y+5)
                text.x -= text.width/2
                ticks_ctn.addChild(text)
                ticks_ctn.addChild(line)
            })

            return ticks_ctn
        },
        // y 轴固定3个ticks
        drawyTicks(label,x,y,orgx){
            let ticks_ctn = new PIXI.Container()
            let format1 = d3.format('.2s')
            let format2 = d3.format('.2f')
            let scale = vm.dimensions[label].hist_scale
            let extent = vm.dimensions[label].hist_extent
            let range = extent[1] - extent[0]
            let gap = range / 4
            let ticks = [1,2,3].map(i => {
                return (i * gap)+extent[0]
            })

            ticks.forEach(tick =>{
                let text = undefined
                if(tick > 1)
                  text = vm.Text(format1(tick),10)
                else
                  text = vm.Text(format2(tick),10)
                text.y = y - scale(tick)
                text.x = x
                let line = vm.drawSolidLine(orgx,text.y,orgx-5,text.y)
                text.y -= text.height/2
                ticks_ctn.addChild(text)
                ticks_ctn.addChild(line)
            })

            return ticks_ctn
        },
        // 绘制實線
        drawSolidLine(x1,y1,x2,y2,color,name=undefined){
            // 绘制轴线
            let axis = new PIXI.Graphics()
            axis.name = name
            // 线宽 1 跟 2 其实是一样的,但是因为渲染的问题,奇数单位的线宽会变得比较模糊不清楚
            axis.lineStyle(2, color, 1)
            axis.moveTo(x1, y1)
            axis.lineTo(x2, y2)
            return axis
        },
        // 绘制文本
        Text(content='no text',fontSize=18,fill='0x000000'){
            let text = new PIXI.Text(content,{
                fontFamily:'Arial',
                // fontStyle:'bold',
                fontSize:fontSize,
                fill:fill,
                align:'center',
            })
            text.resolution = 1
            return text
        },
        clear(){
            if (vm.app !== undefined) {
                vm.app.destroy()
            }
        }
    },

    mounted(){  
        vm = this
        vm.load = true

        PIXI = vm.$PIXI
        d3 = vm.$d3
        vm.loadData()
        // 动态调整 app 长宽比例
        // window.addEventListener('resize', vm.handleResize)
    },
	beforeDestroy() {
		if (vm.app !== undefined) {
			vm.app.destroy()
		}
	}
}
</script>