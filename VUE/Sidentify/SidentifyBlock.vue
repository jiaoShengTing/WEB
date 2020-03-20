<template >
  <div>
    <div class="slidingPictures" :style="'width: '+width+'px;'">
      <div class="sliding-pictures">
        <div class="operation">
          <span
            title="关闭验证码"
            style="right:5px;top:10px"
            @click="closeSilber"
            class="el-icon-circle-close"
          ></span>
          <span
            title="刷新验证码"
            style="right:40px;top:10px"
            @click="refresh"
            class="el-icon-refresh-left"
          ></span>
        </div>
        <div class="vimg">
          <canvas  :height="height" :width="width" ref="blockRef" id="sliderBlock"></canvas>
          <canvas   :height="height" :width="width" ref="codeRef" id="codeImg"></canvas>
        </div>
        <div class="slider">
          <div class="track" :class="{ pintuTrue: puzzle }">{{ tips }}</div>

          <div :class="'button '+ buttonClss" @mousedown.prevent="drag"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from "@/api";
export default {
  data() {
    return {
      tips: "拖动左边滑块完成上方拼图", //提示
      puzzle: false, //判断拼图是否证正确
      codeImage: "", //编码图片
      blockImage: "", //滑块图片
      buttonClss: "el-icon-check", //拖动滑块样式
      blockY: 0 //滑块Y轴的位置
    };
  },
  methods: {
    demo(event) {
      var X = event.offsetX;
      var Y = event.offsetY;
      console.log(X,Y)
    },
    canvasInit(tips) {
      api.user.getCaptchaImage().then(res => {
        this.codeImage = res.data.data.bgImage;
        this.blockImage = res.data.data.sliderImage;
        this.blockY = res.data.data.yposition;
        this.clearCanvas(tips, "el-icon-check", false);
        this.drawImage(15);
      });
    },
    drawImage(bx) {
      let _this = this;
      let mainDom = document.querySelector("#codeImg");
      let bg = mainDom.getContext("2d");
      let width = mainDom.width;
      let height = mainDom.height;
      let blockDom = document.querySelector("#sliderBlock");
      let block = blockDom.getContext("2d");
      //重新赋值，让canvas进行重新绘制
      blockDom.height = height;
      mainDom.height = height;
      //创建背景图片
      let img = document.createElement("img");
      img.style.objectFit = "scale-down";
      img.src = "data:image/png;base64," + this.codeImage;//后端传入base64
      img.onload = function() {
        bg.drawImage(img, 0, 0, width, height);// 
      };
      //创建滑块图
      let imgBlock = document.createElement("img");
      imgBlock.style.objectFit = "scale-down";
      imgBlock.src = "data:image/png;base64," + this.blockImage; //后端传入base64
      block.arc(width, height, 0, 0, 2 * Math.PI); //
      imgBlock.onload = function() {
        block.drawImage(
          imgBlock,
          0,
          _this.blockY,
          _this.blockWH,
          _this.blockWH
        );
      };
    },
    drag(e) {
      let dom = e.target; //dom元素
      let slider = document.querySelector("#sliderBlock"); //滑块dom
      const downCoordinate = { x: e.x, y: e.y };
      //x轴数据
      let x = 0;
      const move = moveEV => {
        x = moveEV.x - downCoordinate.x;
        if (x >= this.width - 20 || x <= 0) return false;
        dom.style.left = x + "px";
        slider.style.left = x + "px";
      };
      const up = e => {
        this.buttonClss = "el-icon-loading";
        this.tips = "正在验证";
        document.removeEventListener("mousemove", move);
        document.removeEventListener("mouseup", up);
        // 采用点击距离和最终离开距离计算滑动距离
        let movex = e.x - downCoordinate.x;
        setTimeout(() => {
          if (this.returnOffset(movex)) {
            dom.style.left = this.width - 20+"px";
            this.clearCanvas("验证成功", "el-icon-success", true);
            setTimeout(()=>{
                this.onSuccess();
            },1000)
          } else {
            this.canvasInit("验证失败，请重试","el-icon-error");
            this.onError();
            //this.clearCanvas("验证失败，请重试", "el-icon-error", false);
          }
        }, 1000);
        //允许正负误差1
      };
      document.addEventListener("mousemove", move);
      document.addEventListener("mouseup", up);
    },
    refresh() {
      this.canvasInit("拖动左边滑块完成上方拼图");
    },
    clearCanvas(tips, buttonClss, puzzle) {
      if (!puzzle) {
        let slider = document.querySelector("#sliderBlock"); //滑块dom
        let buttonDom = document.querySelector(".button"); //滑块dom
        buttonDom.style.left = "";
        setTimeout(() => {
          slider.style.left = "";
        }, 100);
      }
      this.tips = tips;
      this.puzzle = puzzle;
      this.buttonClss = buttonClss;
    }
  },
  mounted() {
    this.canvasInit("拖动左边滑块完成上方拼图");
  },
  watch: {
    visible(e) {
      if (e === true) {
        this.canvasInit( "拖动左边滑块完成上方拼图");
        this.puzzle = false;
      }
    }
  },
  props: {
    //宽
    width: {
      type: Number,
      default: 300
    },
    //高
    height: {
      type: Number,
      default: 150
    },
    // 滑块大小
    blockWH: {
      type: Number,
      default: 50
    },
    visible: {
      type: [Boolean],
      default: false
    },
    // 成功的回调
    onSuccess: {
      type: Function,
      default: () => {
        console.log("成功");
      }
    }, // 失败的回调
    onError: {
      type: Function,
      default: () => {
        console.log("失败");
      }
    },
    //点击关闭按钮
    closeSilber: {
      type: Function,
      default: () => {
        console.log("失败");
      }
    },
    returnOffset:{
      type: Function,
      default: () => {
        console.log("offset");
      }
    }
  }
};
</script>

<style scoped lang="scss">
.slidingPictures {
  border-radius: 0px;
}
/*该样式最终是以弹窗插入*/
.sliding-pictures {
  width: 100%;
  //   height: 260px;
  .vimg {
    width: 100%;
    // height: 170px;
    #codeImg,
    #sliderBlock {
      //padding: 7px 7px 0 7px;
    }
    #codeImg {
      //display: none;
    }
    #sliderBlock {
      position: absolute;
      z-index: 4000;
    }
  }
  .slider {
    width: 100%;
    height: 45px;
    border-bottom: #c7c9d0 1px solid;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    .track {
      margin-left: 7px;
      width: 100%;
      height: 28px;
      background: rgba(28, 136, 188, 0.5);
      border-radius: 25px;
      font-size: 10px;
      line-height: 30px;
      padding-right: 15px;
      padding-left: 70px;
    }
    .pintuTrue {
      background: #67c23a;
      color: #ffffff;
    }
    .pintuError {
      background: red;
    }
    .button {
      position: absolute;
      width: 28px;
      height: 28px;
      line-height: 28px;
      background: #ffffff;
      box-shadow: #b9bdc8 0 0 2px;

      left: 7px;
      text-align: center;
      font-size: 18px;
      color: #3e5d8b;
      &:hover {
        color: #2181bd;
      }
    }
  }
  .operation {
    //   style="position: absolute; right:5px;top:50px"
    width: 100%;
    // height: 40px;
    > span {
      position: absolute;
      color: #020305;
      display: inline-block;
      width: 40px;
      font-size: 25px;
      line-height: 40px;
      text-align: center;
      z-index: 5000;
      &:hover {
        background: #e2e8f5;
      }
    }
  }
}
</style>