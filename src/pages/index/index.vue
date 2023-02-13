<template>
  <div class="body">
    <div class="content">
    <div class="coin" @click="beLuckey()" :style="styleobj">
        <div class="coin-front"></div>
        <div class="coin-back"></div>
        <div class="coin-edge">
          <div class="coin-edge-face" v-for="i in edge_faces" :key="i" :style="styles[i]"></div>
        </div>
      </div>
    <div class="desk" :class="{ clicked: clicked, 'coin-landed': coin_landed }">
      <div class="desk-mark"></div>
      <div class="desk-mark2"></div>
      <span class="tip">点击硬币开始</span>
      <span class="result">{{ result.name }}</span>
    </div>
  </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      styleobj: {        
        "--coin-y-multiplier": 0,
        "--coin-scale-multiplier": 0,
        "--coin-rotation-multiplier": 0,        
      },
      result: {
        name: "",
        val: 0
      },
      styles:[],
      clicked: false,
      coin_landed: false,
      
      coin_diameter : 312,
      coin_thickness : 40,
      edge_faces : 100,

      maxMoveLoopCount:90,
      sideRotationCount:0,
      maxFlipAngle:0,
      moveLoopCount:0,
      angle:0,
    }
  },
  onShareAppMessage(from, target, webViewUrl) {
    return {
      title: "抛硬币决定吧",
      path: "/pages/index/index",
      content: "抛硬币决定吧",
      desc: "遇事不决抛硬币，没事点点也解压"
    }
  },
  onLoad() {
    for (let i = 1; i <= this.edge_faces; i++) {
      let v1 = this.coin_diameter / 2 - (3.14 * this.coin_diameter / this.edge_faces) / 2
      let v2 = this.coin_diameter / 2 - this.coin_thickness / 2
      let v3 = 360 / this.edge_faces * i + 90
      let v4 = this.coin_diameter / 2
      this.styles[i] = {
        height: (3.14 * this.coin_diameter / this.edge_faces) + "rpx",
        width: this.coin_thickness + "rpx",
        transform:"translateY(" + String(v1) + "rpx) translateX(" + v2 + "rpx)  rotateY(90deg) rotateX(" + v3 + "deg ) translateZ(" + v4 + "rpx)"
      }
    }
  },
  methods: {
    beLuckey($e){
      let coin = uni.createSelectorQuery().select(".coin")
      
      // 如果处于点击状态直接返回
      if (this.clicked) return
      this.clicked = true
      // 开始反转硬币
      setTimeout(() => {
        this.sideRotationCount = (Math.floor(Math.random() * 5 + 4)) * 180
        this.maxFlipAngle = (Math.floor(Math.random() * 4) + 3) * Math.PI
        flipCoin()
      }, 50)

      // 反转硬币
      const flipCoin = () => {
        console.log(this.sideRotationCount)

        this.moveLoopCount = 0
        let intervalId = setInterval(() => {
          flipCoinLoop(intervalId)
        }, 16)
      }
      // 反转硬币的循环
      const flipCoinLoop = (intervalId) => {
        this.moveLoopCount++
        let percentageCompleted = this.moveLoopCount / this.maxMoveLoopCount
        this.angle = -this.maxFlipAngle * Math.pow((percentageCompleted - 1), 2) + this.maxFlipAngle

        // Calculate the scale and position of the coin moving through the air
        this.styleobj["--coin-y-multiplier"] = -40 * Math.pow(percentageCompleted * 2 - 1, 4) + 40        
        this.styleobj["--coin-scale-multiplier"] = ((percentageCompleted * 2) <= 1 ? (percentageCompleted * 2) : (1 - percentageCompleted * 2 + 1)) * 0.5
        this.styleobj["--coin-rotation-multiplier"] = percentageCompleted * this.sideRotationCount

        if (percentageCompleted > 0.85) {
          this.coin_landed = true
        }

        console.log(this.styleobj["--coin-rotation-multiplier"])

        // Repeat animation loop
        if (this.moveLoopCount >= this.maxMoveLoopCount) {
          clearInterval(intervalId)
          if (this.sideRotationCount/180%2 == 1) {
              this.result.name = "星星"
              this.result.val = 2
          } else  {
            this.result.name =  "笑脸"
            this.result.val = 1
          }

          setTimeout(() => {
            this.clicked = false
            this.coin_landed = false
            this.resetCoinStyle()
          }, 100)
        }
      }
    },
    resetCoinStyle(){        
        this.styleobj["--coin-scale-multiplier"] = 0
        this.styleobj["--coin-y-multiplier"] = 0        
    }
  },
}
</script>

<style>
.body {
  perspective: 3000rpx;
  width: 100vw;
  height: 100vh;
  perspective-origin: 50% 50%;
  overflow: hidden;
}

.content {
  padding: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  
  align-items: flex-end;
  justify-content: center;
  -webkit-font-smoothing: antialiased;
  transform-style: preserve-3d;
  background: none;
  transform: rotateY(0deg);
}

.desk {
  background: #715af3;
  z-index: 1;
  height: 3000rpx;
  width: 3000rpx;
  overflow: hidden;
  position: absolute;
  left: 50%;
  top: 50%;
  margin-left: -1501rpx;
  margin-top: -1500rpx;
  transition: opacity 200ms linear 100ms, transform 200ms linear;
  background-size: 150rpx 150rpx;
  background-image:
    linear-gradient(90deg, rgba(255, 255, 255, 0.4) 1px, transparent 1px),
    linear-gradient(0, rgba(255, 255, 255, 0.4) 1px, transparent 1px);
  transform: rotateX(20deg) translateZ(-50rpx);
  transform-style: preserve-3d;
}

.desk.clicked {
  transform: rotateX(30deg) translateZ(-50rpx);
}

.desk.clicked.coin-landed {
  transform: rotateX(20deg) translateZ(-50rpx);
}

.desk .desk-mark {
  width: 480rpx;
  height: 480rpx;
  background: none;
  position: absolute;
  left: 50%;
  top: 50%;
  margin-left: -240rpx;
  margin-top:-240rpx;
  border-radius: 50%;
  border: 1rpx solid rgba(255, 255, 255, 0.15);
}

.desk .desk-mark2 {
  width: 340rpx;
  height: 340rpx;
  background: none;
  position: absolute;
  left: 50%;
  top: 50%;
  margin-left: -170rpx;
  margin-top:-165rpx;
  border-radius: 50%;
  border: 1rpx solid rgba(255, 255, 255, 0.15);
}

.desk .tip {
  text-align: center;
  width: 200rpx;
  height: 50rpx;
  line-height: 50rpx;
  margin-left: -100rpx;
  display: inline-block;
  position: absolute;
  top: 50%;
  left: 50%;
  font-weight: 600;
  color: wheat;
  margin-top: 220rpx;
  z-index: 3;
}

.desk .result {
  text-align: center;
  width: 200rpx;
  height: 50rpx;
  line-height: 50rpx;
  margin-left: -100rpx;
  display: inline-block;
  position: absolute;
  font-size: 100rpx;
  top: 50%;
  left: 50%;
  font-weight: 600;
  color: wheat;
  margin-top: 320rpx;
  z-index: 3;
}

.coin {
  background: none;
  --coin-y-multiplier: 0;
  --coin-scale-multiplier: 0;
  --coin-rotation-multiplier: 0;
  z-index: 100;
  bottom: calc(var(--coin-y-multiplier) * 8rpx + 50%);
  height: 312rpx;
  width: 312rpx;
  right: 50%;
  margin-right: -156rpx;
  margin-bottom: -178rpx;
  position: absolute;
  transform-origin: 50% 50%;
  transform-style: preserve-3d;
  transform: scale(calc(1 + var(--coin-scale-multiplier))) rotateX(calc(var(--coin-rotation-multiplier) * 1deg + 20deg)) 
}

.coin-front {
  border: 1px dashed rgba(0, 0, 0, 0.15);
  border-radius: 50%;
  position: absolute;
  left: 0;
  height: 312rpx;
  width: 312rpx;
  background: radial-gradient(circle at 50% 50%, rgba(245,88,123,0.9) 60%, transparent 100%);
  transform: translateZ(20rpx);
  transform-style: preserve-3d;
}

.coin-front::after {
  position: absolute;
  left: 0;
  border-radius: 50%;
  content: '';
  width: 312rpx;
  height: 312rpx;
  background-repeat: no-repeat;
  background-image: url("/static/simle.svg");
  background-size: 100% 100%;
  background-position: center;
  transform-style: preserve-3d;
}

.coin-back {
  border: 1px dashed rgba(0, 0, 0, 0.15);
  border-radius: 50%;
  position: absolute;
  left: 0;
  width: 312rpx;
  height: 312rpx;
  background: radial-gradient(circle at 50% 50%, rgba(245,88,123,0.9) 60%, transparent 100%);
  transform: translateZ(-20rpx) rotateY(180deg);
  transform-style: preserve-3d;
}

.coin-back::after {
  position: absolute;
  left: 0;
  border-radius: 50%;
  content: '';
  width: 312rpx;
  height: 312rpx;
  background-repeat: no-repeat;
  background-image: url("/static/star.svg");
  background-size: 80% 80%;
  background-position: center;
  transform-style: preserve-3d;
}

.coin-edge {
  transform-style: preserve-3d;
}

.coin-edge .coin-edge-face {
  transform-style: preserve-3d;
  background-color: rgba(245,88,123,0.9);
  position: absolute;
}
</style>
