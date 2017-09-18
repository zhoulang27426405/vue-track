<style>
  .img-wrap {
    float: left;
    width: 100%;
  }
  .demo-img {
    float: left;
    width: 48%;
    text-align: left;
    vertical-align: top;
  }
  .demo-img #img {
    width: 100%;
  }
  .rect {
    position: absolute;
    left: 1000px;
    top: 1000px;
    border: 2px solid #a64ceb;
  }
  .hide {
    display: none;
  }
  .form-wrap {
    padding: 20px;
    text-align: left;
    background: #fcfcfc;
  }
  .form-group {
    float: left;
    width: 48%;
    min-height: 60px;
    margin-bottom: 20px;
  }
  .form-wrap h3 {
    font-size: 40px;
    line-height: 1;
  }
  .form-wrap .form-row {
    display: inline-block;
    width: 100%;
    height: 60px;
    box-sizing: border-box;
    margin-bottom: 30px;
  }
  .form-row label {
    font-size: 28px;
    margin-right: 20px;
  }
  .form-row .btn {
    position: relative;
    display: inline-block;
    box-sizing: border-box;
    width: 100px;
    height: 60px;
    line-height: 60px;
    text-align: center;
    border: none;
    border-radius: 4px;
    font-size: 28px;
    color: #fff;
    background-color: #2EA9DF;
  }
  .form-row .btn input {
    position: absolute;
    top: 0;
    left: 0;
    font-size: 100%;
    height: 0;
    opacity: 0;
  }
  .form-row .tips {
    position: absolute;
    font-size: 30px;
  }
  .form-row .range span {
    display: inline-block;
  }
  input[type="range"] {
      -webkit-appearance: none;
      width: 80%;
      border-radius: 10px; /*这个属性设置使填充进度条时的图形为圆角*/
  }
  input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
  }
  input[type="range"]::-webkit-slider-runnable-track {
      height: 25px;
      border-radius: 10px; /*将轨道设为圆角的*/
      box-shadow: 0 1px 1px #def3f8, inset 0 .125em .125em #0d1112; /*轨道内置阴影效果*/
  }
  input[type="range"]:focus {
      outline: none;
  }
  input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      height: 45px;
      width: 45px;
      margin-top: -10px; /*使滑块超出轨道部分的偏移量相等*/
      background: #ffffff;
      border-radius: 50%; /*外观设置为圆形*/
      border: solid 0.125em rgba(205, 224, 230, 0.5); /*设置边框*/
      box-shadow: 0 .125em .125em #3b4547; /*添加底部阴影*/
  }
  .form-group button {
    position: relative;
    display: inline-block;
    box-sizing: border-box;
    padding: 10px 20px;
    text-align: center;
    border: none;
    border-radius: 4px;
    font-size: 28px;
    color: #fff;
    background-color: #e55;
  }
</style>
<template>
  <div class="tracking-wrap">
    <div class="form-wrap">
      <div class="form-group">
        <h3>选择图片</h3>
        <div class="form-row"><label>上传</label><div class="btn">上传<input value="选择相册" type="file" @change="uploadPic($event)" /></div></div>
        <div class="form-row"><label>拍摄</label><div class="btn">拍摄<input value="拍摄" type="file" capture="camera" accept="image/*" @change="uploadPic($event)" /></div></div>
      </div>
      <div class="form-group">
        <h3>参数设置</h3>
        <div class="form-row">
          <label>步长</label>
          <input type="range" min="1" max="2" step="0.1"  value="1.3" @change="setStepSize($event)" />
          <div class="tips">{{this.StepSize}}</div>
        </div>
        <div class="form-row">
          <label>比例</label>
          <input type="range" min="1" max="2" step="0.1"  value="1.3" @change="setScaleFactor($event)" />
          <div class="tips">{{this.ScaleFactor}}</div>
        </div>
      </div>
      <div class="form-group">
        <button @click="spot">识别</button>
      </div>
      <div class="form-group">
        <button @click="fix" v-show="isShowBtn">合成</button>
      </div>
    </div>

    <div class="img-wrap">
      <div class="demo-img">
        <div id="demo-container">
          <img id="img" src="../assets/static/faces.jpg" />
        </div>
      </div>
      <div class="demo-img">
        <canvas id="myCanvas"></canvas>
      </div>
      <div class="hide">
          <img src="../assets/static/male.png" id="maleBg">
          <img id="theFace">
      </div>
    </div>
  </div>
</template>
<script>
  import tracking from '../../static/tracking-min.js'
  import face from '../../static/face-min.js'
  import alloyimage from '../../static/alloyimage-1.1.js'
  export default {
    data() {
      return {
        img: null, // 原始图片
        prop: 1, // 图片缩放比例
        tracker: null, // tracker实例
        StepSize: 1.3, // track参数
        ScaleFactor: 1.4, // track参数
        isShowBtn1: false,
        isShowBtn: false
      }
    },
    mounted() {
      this.init()
    },
    methods: {
      // 初始化
      init() {
        // 原始图片
        this.img =  document.getElementById('img')
        // 实例化
        this.tracker = new window.tracking.ObjectTracker(['face'])
        // 监听
        this.tracker.on('track', this.spotCallBack)
      },
      // 识别
      spot() {
        // 调用
        if (!this.img.src) {
          return;
        }
        this.tracker.setStepSize(this.StepSize);
        this.tracker.setScaleFactor(this.ScaleFactor);
        if (this.img.naturalWidth > this.img.width) {
            this.prop = this.img.naturalWidth / this.img.width;
        } else {
            this.prop = 1;
        }
        window.tracking.track(this.img, this.tracker);
      },
      // 面部识别回调函数
      spotCallBack(event) {
        if (!event.data.length) {
          alert('没有识别');
        } else {
          event.data.forEach((rect) => {
            // 面部数据提取
            this.plotRectangle(rect.x, rect.y, rect.width, rect.height);
            this.plotFace(this.img, rect.x * this.prop, rect.y * this.prop, rect.width * this.prop, rect.height * this.prop, 0, 0, rect.width * this.prop, rect.height * this.prop);
          })
          this.isShowBtn = true;
        }
      },
      // 绘制面部聚焦矩形
      plotRectangle(x, y, w, h) {
        let rect = document.createElement('div');
        document.getElementById('demo-container').append(rect);
        rect.classList.add('rect');
        rect.style.width = w + 'px';
        rect.style.height = h + 'px';
        rect.style.left = (this.img.offsetLeft + x) + 'px';
        rect.style.top = (this.img.offsetTop + y) + 'px';
      },
      // 绘制面部
      plotFace(img, sx, sy, sw, sh, x, y, w, h) {
        var c = document.getElementById("myCanvas");
        c.width = w;
        c.height = h;
        var cxt = c.getContext("2d");
        cxt.drawImage(img, sx, sy, sw, sh, x, y, w, h);
      },
      // 图片处理及模板合成
      fix() {
        let faceImg = document.getElementById('theFace');
        let faceC = document.getElementById('myCanvas');
        let maleBg = document.getElementById('maleBg');
        faceImg.src = faceC.toDataURL("image/jpeg", 1.0);
        faceImg.loadOnce(function() {
            AlloyImage(this).act("灰度处理").add(
                AlloyImage(this.width, this.height, "#808080")
                .act("高斯模糊", 4)
                .act("色相/饱和度调节", 22, 45, 0, true),
                "叠加"
            ).replace(this);
            setTimeout(function() {
                faceC.width = maleBg.width;
                faceC.height = maleBg.height;
                let ctx = faceC.getContext("2d");
                ctx.drawImage(faceImg, 0, 0, faceImg.width, faceImg.height, 95, 100, 186, 186);
                ctx.drawImage(maleBg, 0, 0);
            }, 400)
        });
      },
      // 上传图片
      uploadPic(e) {
        let files = e.target.files;
        if (!files[0]) {
          return;
        }
        let reader = new FileReader();
        reader.readAsDataURL(files[0]);
        reader.onload = () => {
          this.img.src = reader.result;
        }
      },
      // track参数配置
      setStepSize(e) {
        this.StepSize = e.target.value;
      },
      setScaleFactor(e) {
        this.ScaleFactor = e.target.value;
      }
    }
  }
</script>