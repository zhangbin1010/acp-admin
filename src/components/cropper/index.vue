<template>
  <div class="cropper-wrapper">
    <el-card class="cropper-card">
      <div class="img-box">
        <img class="cropper-image" :id="imgId" alt="" style="max-width: 100%" src="">
      </div>
      <div class="right-con">
        <el-row style="height: 185px;">
          <el-col :span="24">
            <div class="preview-box preview-lg">
              <div class="img-preview"></div>
            </div>
          </el-col>
        </el-row>
        <el-row style="height: 170px;">
          <div class="preview-box preview-md">
            <div class="img-preview"></div>
          </div>
          <div class="preview-box preview-sm">
            <div class="img-preview"></div>
          </div>
          <div class="preview-box preview-xs">
            <div class="img-preview"></div>
          </div>
        </el-row>
        <el-row style="margin-top: 33px;display: inline-block;">
          <div style="float: left">
            <el-upload action="image/upload" :before-upload="beforeUpload">
              <el-button size="small" style="width: 150px;" type="primary">{{ $t('forms.buttons.upload') }}
              </el-button>
            </el-upload>
          </div>
          <div style="float: left;margin-left: 20px;">
            <el-button v-show="insideSrc" style="width: 150px;" size="small" type="primary" @click="crop">
              {{ cropButtonText }}
            </el-button>
          </div>
        </el-row>
      </div>
      <div v-show="insideSrc">
        <el-row :gutter="16">
          <div class="input-box">
            <el-col :span="6">
              <el-input v-model="X" @blur="reSize" @keyup.enter.native="reSize" size="small">
                <template #prepend>X</template>
                <template #append>px</template>
              </el-input>
            </el-col>
            <el-col :span="6">
              <el-input v-model="Y" @on-blur="reSize" @keyup.enter.native="reSize" size="small">
                <template #prepend>Y</template>
                <template #append>px</template>
              </el-input>
            </el-col>
            <el-col :span="6">
              <el-input v-model="width" @on-blur="reSize" @keyup.enter.native="reSize" size="small">
                <template #prepend>{{ $t('forms.width') }}</template>
                <template #append>px</template>
              </el-input>
            </el-col>
            <el-col :span="6">
              <el-input v-model="height" @on-blur="reSize" @keyup.enter.native="reSize" size="small">
                <template #prepend>{{ $t('forms.height') }}</template>
                <template #append>px</template>
              </el-input>
            </el-col>
          </div>
        </el-row>
      </div>
      <div v-show="insideSrc">
        <div class="button-box">
          <el-button-group style="margin-right: 10px">
            <el-button type="primary" @click="reset" size="small">
              <i class="el-icon-refresh"/>
            </el-button>
            <el-button type="primary" @click="mode('move')" size="small">
              <i class="el-icon-rank"/>
            </el-button>
            <el-button type="primary" @click="mode('crop')" size="small">
              <i class="el-icon-crop"/>
            </el-button>
          </el-button-group>
          <el-button-group style="margin-right: 10px">
            <el-button type="primary" @click="shrink" size="small">
              <i class="el-icon-zoom-out"/>
            </el-button>
            <el-button type="primary" @click="magnify" size="small">
              <i class="el-icon-zoom-in"/>
            </el-button>
          </el-button-group>
          <el-button-group style="margin-right: 10px">
            <el-button type="primary" @click="rotate(-1)" size="small">
              <i class="el-icon-refresh-left"/>
            </el-button>
            <el-button type="primary" @click="rotate(1)" size="small">
              <i class="el-icon-refresh-right"/>
            </el-button>
            <el-button type="primary" @click="scale('X')" size="small">
              <i class="el-icon-d-caret icon-leftAndRight"/>
            </el-button>
            <el-button type="primary" @click="scale('Y')" size="small">
              <i class="el-icon-d-caret"/>
            </el-button>
          </el-button-group>
          <el-button-group style="margin-right: 10px">
            <el-button type="primary" @click="move(0, -moveStep)" size="small">
              <i class="el-icon-caret-top"/>
            </el-button>
            <el-button type="primary" @click="move(-moveStep, 0)" size="small">
              <i class="el-icon-caret-left"/>
            </el-button>
            <el-button type="primary" @click="move(0, moveStep)" size="small">
              <i class="el-icon-caret-bottom"/>
            </el-button>
            <el-button type="primary" @click="move(moveStep, 0)" size="small">
              <i class="el-icon-caret-right"/>
            </el-button>
          </el-button-group>
          <el-button-group>
            <el-button type="primary" @click="aspact(16/9)" size="small">16:9</el-button>
            <el-button type="primary" @click="aspact(4/3)" size="small">4:3</el-button>
            <el-button type="primary" @click="aspact(1)" size="small">1:1</el-button>
            <el-button type="primary" @click="aspact(2/3)" size="small">2:3</el-button>
            <el-button type="primary" @click="aspact(NaN)" size="small">{{ $t('forms.buttons.free') }}</el-button>
          </el-button-group>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import {nextTick} from 'vue'
import Cropper from 'cropperjs'
import './index.less'
import 'cropperjs/dist/cropper.min.css'

export default {
  name: 'Cropper',
  props: {
    src: {
      type: String,
      default: ''
    },
    moveStep: {
      type: Number,
      default: 1
    },
    cropButtonText: String
  },
  data() {
    return {
      cropper: null,
      insideSrc: '',
      X: '',
      Y: '',
      width: '',
      height: ''
    }
  },
  computed: {
    imgId() {
      return `cropper${this._uid}`
    }
  },
  watch: {
    src(src) {
      this.replace(src)
    },
    insideSrc(src) {
      this.replace(src)
    }
  },
  methods: {
    beforeUpload(file) {
      const reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onload = (event) => {
        this.insideSrc = event.target.result
      }
      return false
    },
    replace(src) {
      this.cropper.replace(src)
      this.insideSrc = src
    },
    reset() {
      this.cropper.reset()
    },
    mode(value) {
      this.cropper.setDragMode(value)
    },
    rotate(value) {
      if (value > 0) {
        this.cropper.rotate(90)
      } else {
        this.cropper.rotate(-90)
      }
    },
    shrink() {
      this.cropper.zoom(-0.1)
    },
    magnify() {
      this.cropper.zoom(0.1)
    },
    scale(d) {
      this.cropper[`scale${d}`](-this.cropper.getData()[`scale${d}`])
    },
    move(...args) {
      this.cropper.move(...args)
    },
    aspact(value) {
      this.cropper.setAspectRatio(value)
    },
    reSize() {
      const data = this.cropper.getData()
      const x = Number(this.X && this.X !== '' ? this.X : data.x)
      const y = Number(this.Y && this.Y !== '' ? this.Y : data.y)
      const width = Number(this.width && this.width !== '' ? this.width : data.width)
      const height = Number(this.height && this.height !== '' ? this.height : data.height)
      this.cropper.setData({
        x: x,
        y: y,
        width: width,
        height: height
      })
    },
    crop() {
      this.$emit('on-crop', this.cropper.getCroppedCanvas().toDataURL())
    }
  },
  mounted() {
    const obj = this
    nextTick(() => {
      let dom = document.getElementById(this.imgId)
      this.cropper = new Cropper(dom, {
        preview: '.img-preview',
        checkCrossOrigin: true,
        crop: function (data) {
          obj.X = data.detail.x.toFixed(0)
          obj.Y = data.detail.y.toFixed(0)
          obj.width = data.detail.width.toFixed(0)
          obj.height = data.detail.height.toFixed(0)
        }
      })
    })
  }
}
</script>
