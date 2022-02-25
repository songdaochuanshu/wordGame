<script setup lang="ts">
import { useSound } from '@vueuse/sound'
import buttonSfx from '../assets/button.wav'

const { play } = useSound(buttonSfx)
const isPlay = ref(true)
const setPlay = () => {
  isPlay.value = !isPlay.value
}

const global: g = {
  cvs: null,
  ctx: null,
  score: 0,
  lives: 5,
  appearTime: 1000,
  moveTime: 10,
  flagRun: false,
  letters: "abcdrfghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ",
  letterArr: []//letter,point.x,point.y
};


interface g {
  cvs: HTMLCanvasElement | any
  ctx: CanvasRenderingContext2D | any
  score: number
  lives: number
  appearTime: number
  moveTime: number
  flagRun: boolean
  letters: string
  letterArr: Array<any>
}

const btnText = ref('开始')

onMounted(() => {
  //当输入键盘内容时
  document.onkeypress = function (e) {
    for (var i = 0, len = global.letterArr.length; i < len; i++) {
      if (String.fromCharCode(e.charCode) == global.letterArr[i].letter) {
        global.letterArr.splice(i, 1)
        global.score++
        isPlay.value ? play() : ''
        let score = document.getElementById('score') as HTMLElement
        score.innerHTML = global.score + ''
        if (global.score % 10 == 0 && global.appearTime > 10) {
          global.appearTime -= 40
        } else if (global.score % 20 == 0 && global.moveTime > 0) {
          global.moveTime--
        }
        break
      }
    }
  }
})


const start = () => {
  if (!global.flagRun) {
    global.flagRun = true
    startGame()
    btnText.value = '暂停'
  } else {
    alert("游戏正在进行！")
  }
}


const init = () => {
  global.cvs = document.getElementById("wordCanvas")
  global.ctx = global.cvs.getContext('2d')
  global.cvs.width = window.innerWidth - 50
  global.cvs.height = window.innerHeight - 30
  global.ctx.font = "Bold 25px verdana"
  global.ctx.shadowColor = "#00b0f0"
  global.ctx.shadowBlur = 15
  global.ctx.fillStyle = "#fff"
  let score = document.getElementById('score') as HTMLElement
  let lives = document.getElementById('lives') as HTMLElement
  score.innerHTML = global.score + ''
  lives.innerHTML = global.lives + ''
  drawText()
}

const startGame = () => {
  init()
  getLetter()
  moveDown(1)
}

//获取单词
const getLetter = () => {
  var le = {
    letter: global.letters[Math.round(Math.random() * 104) % 52],
    point: {
      x: Math.round(Math.random() * 40) % 30 * ((window.innerWidth - 50) / 30),
      y: 0
    }
  }
  global.letterArr.push(le)
  if (global.flagRun) {
    setTimeout(() => {
      getLetter()
    }, global.appearTime)
  }
}

//单词向下移动
const moveDown = (speed: number) => {
  for (var i = 0, len = global.letterArr.length; i < len; i++) {
    global.letterArr[i].point.y += speed;
    if (global.letterArr[i].point.y >= window.innerHeight - 30) {
      global.lives--
      let lives = document.getElementById('lives') as HTMLElement
      lives.innerHTML = global.lives + ''
      delete global.letterArr[i]
      if (global.lives <= 0) {
        alert("Game Over!")
        global.flagRun = false
        btnText.value = '开始'
      }
    }
  }
  global.letterArr = compressArr(global.letterArr)
  drawText();
  if (global.flagRun) {
    setTimeout(function () {
      moveDown(1)
    }, global.moveTime)
  }
}

//画出单词
const drawText = () => {
  global.ctx.clearRect(0, 0, global.cvs.width, global.cvs.height)
  for (var i = 0, len = global.letterArr.length; i < len; i++) {
    global.ctx.fillText(global.letterArr[i].letter, global.letterArr[i].point.x, global.letterArr[i].point.y)
  }
}

//压缩数组
const compressArr = (arr: any[]) => arr.filter((x: null | undefined) => x != undefined && x != null)

</script>

<template>
  <div id="content">
    <div class="game">
      <button btn @click="start">{{ btnText }}</button>
      <button btn ml-3 @click="setPlay">{{ isPlay ? '关闭音效' : '开启音效' }}</button>
      <span ml-3>分数：</span>
      <span id="score"></span>
      <span ml-3>生命：</span>
      <span id="lives"></span>
    </div>
    <canvas id="wordCanvas"></canvas>
  </div>
</template>



<style>
#content {
  position: absolute;
}
.game {
  position: absolute;
  left: 0;
  top: 0;
}
</style>