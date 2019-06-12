<!--
* 手写签名板组件
* by linpeiwei
* to 20190516
* -->
<template>
	<div class="main">	
		<div class="writingGo" @click="writingOpen">
			<img :src="writingUrl" />
		</div>
		<div v-show="writingShow" class="writingMian">
			<p>签名区域</p>
			<div class="writingbox">
				<div class="writingDiv">
					<canvas ref="canvas" id="canvas">Canvas画板</canvas>
				</div>
				<div class="writingli"><van-button type="primary" @click="clear">清除</van-button></div>
				<div class="writingli"><van-button type="info" @click="save">提交签名</van-button></div>
				<div class="writingli"><van-button type="danger" @click="writingClose">取消</van-button></div>
				<div style="clear: both;"></div>
			</div>
		</div>
	</div>
</template>

<script>

export default {
  data () {
    return {
      writingUrl: 'https://lbdj.oss-cn-beijing.aliyuncs.com/lbdj_app_h5/page/qianming.jpg',  //签字保存的图片url
	  imgFile: '',  //转换后的图片
	  writingShow: false,  //签名板
    }
  },
  mounted() {
      draw=new Draw('canvas');
      draw.init();
  },
  methods:{
	writingOpen: function(){
		draw.clear();
		this.writingShow = true;
	},
	writingClose:function(){
		this.writingShow = false;
	},
    clear:function(){
        draw.clear();
    },
    save:function(){
		var c=document.getElementById("canvas"); // 获取html的canvas对象，我这个id="canvas"
		if(isCanvasBlank(c)){
			this.$toast.fail("请签名");
			return;
		}else{
			var data=draw.save();
			this.writingUrl = data;
			this.imgFile = this.getBase64Img(data);
			this.writingShow = false;
			this.$emit('getWritingPad',this.writingUrl);
		}
 
    },
	//base64转图片
	getBase64Img: function(file){
		let fileName = 'writing.png';  //图片名称
		var arr = file.split(','), mime = arr[0].match(/:(.*?);/)[1],
			bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
		while(n--){
			u8arr[n] = bstr.charCodeAt(n);
		}
		return new File([u8arr], fileName, {type:mime});
	},
  }
}
//签名板
var draw;
var preHandler = function(e){e.preventDefault();}
class Draw {
    constructor(el) {
        this.el = el
        this.canvas = document.getElementById(this.el);  	//获取canvas
        this.cxt = this.canvas.getContext('2d');  			//canvas追加2d画图
        this.stage_info = canvas.getBoundingClientRect();   //canvas元素的边界框	
        this.path = {
            beginX: 0,
            beginY: 0,
            endX: 0,
            endY: 0
        }
		canvas.width = $('.writingDiv').width()
		canvas.height = $('.writingDiv').height()
    }
    init(btn) {
        var that = this; 

        this.canvas.addEventListener('touchstart', function(event) {
            document.addEventListener('touchstart', preHandler, false); 
            that.drawBegin(event)
        })
        this.canvas.addEventListener('touchend', function(event) { 
            document.addEventListener('touchend', preHandler, false); 
            that.drawEnd()
            
        })
        this.clear(btn)
    }
    drawBegin(e) {
        var that = this;
        window.getSelection() ? window.getSelection().removeAllRanges() : document.selection.empty()
        this.cxt.strokeStyle = "#000";    //设置线条的颜色
        this.cxt.beginPath();			  //绘制开始路径
		//通过moveTo和lineinfo，我们来绘制一条线
        this.cxt.moveTo(
            e.changedTouches[0].clientX - this.stage_info.left,
            e.changedTouches[0].clientY - this.stage_info.top
        )
        this.path.beginX = e.changedTouches[0].clientX - this.stage_info.left
        this.path.beginY = e.changedTouches[0].clientY - this.stage_info.top
        canvas.addEventListener("touchmove",function(){
            that.drawing(event)
        })
    }
    drawing(e) {
		//绘制已定义的路径
        this.cxt.lineTo(
            e.changedTouches[0].clientX - this.stage_info.left,
            e.changedTouches[0].clientY - this.stage_info.top
        )
		//计算、描绘划过的路径
        this.path.endX = e.changedTouches[0].clientX - this.stage_info.left
        this.path.endY = e.changedTouches[0].clientY - this.stage_info.top
        this.cxt.stroke()
    }
    drawEnd() {
        document.removeEventListener('touchstart', preHandler, false); 
        document.removeEventListener('touchend', preHandler, false);
        document.removeEventListener('touchmove', preHandler, false);
    }
	//清除
    clear(btn) {
        this.cxt.clearRect(0, 0, $('.writingDiv').width(), $('.writingDiv').height())
    }
	//保存图片
    save(btn){	
		return canvas.toDataURL("image/png");
    }
	
}
//验证canvas画布是否为空函数
	function isCanvasBlank(canvas) {
		var blank = document.createElement('canvas');//系统获取一个空canvas对象
		blank.width = canvas.width;
		blank.height = canvas.height;
		return canvas.toDataURL() == blank.toDataURL();//比较值相等则为空
	}
</script>


<style>		
	#canvas{
		background:rgba(0,0,0,0.1);
		cursor: default;
		border: 1px dashed #dfdfdf;
		display: block;
		margin: 0px;
		padding: 0;
		border-radius: 10px;
	}
	.writingMian{
		width: 100%;
		height: 100%;
		position: fixed;
		background:rgba(255,255,255,1);
		top: 0px;
		left: 0px;
	}
	.writingbox{
		padding: 20px;
		width: 710px;
		position: fixed;
		top: 0;
		left: 0;
	}
	.writingDiv{
		width: 710px;
		height: 400px;
	}
	.writingMian p{
		font-size: 32px;
		text-align: center;
		width: 100%;
		height: 400px;
		line-height: 410px;
		position: fixed;
		color: #dfc8c8;
	}
	.writingli{
		width: 33.33333%;
		float: left;
		text-align: center;
	}
	.writingGo{
		background-color: #ffffff;
		padding: 20px;
	}
	.writingGo img{
		width: 710px;
		border:1px solid #eeeeee;
	}
</style>
