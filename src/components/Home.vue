<template>
  <el-container>
    <el-main>
      <p style="font-family: 'Times New Roman';font-size: 35px;font-weight: bold;color: #003E3E" align="center">Sketch to Human Face Demo</p>
      <el-dialog
          :visible.sync="visible"
          width="60%"
      >
        <div style="font-size: 30px;font-family: 'Times New Roman';text-align: left" slot="title">
          Guidance
        </div>
        <div style="font-size: 20px;font-family: 'Times New Roman';text-align: left">
          <a style="font-weight: bold">Step1:</a> Draw a freehand drawing of a human face or select an example on the bottom;<br>
          <a style="font-weight: bold">Step2:</a> Select Gender, Skin and Hair Color;<br>
          <a style="font-weight: bold">Step3:</a> Click 'Transfer' to get a real face picture.
        </div>
        <div slot="footer">
          <el-button type="primary" @click="visible = !visible">
            OK
          </el-button>
        </div>
      </el-dialog>
        <el-row>
          <div align="center" style="margin-bottom: 50px">
              <el-button type="info" @click="visible = !visible" plain>Guidance</el-button>
            <el-popover placement="top-start" width="200">
              <el-slider v-model="value2" v-show="isDraw" :min="3" :max="20"></el-slider>
              <div style="font-size: 10px">Brush Size</div>
              <div class="BrushSize" id="BrushSize"></div>
              <el-button @click="Draw" :type=isDraw slot="reference" style="margin-right: 10px;margin-left: 10px">Draw</el-button>
            </el-popover>

              <el-popover placement="top-start" width="200">
                <el-slider v-model="value1" v-show="isEraser" :min="1" :max="100"></el-slider>
                <div style="font-size: 10px">Eraser Size</div>
                <div class="EraserSize" id="EraserSize"></div>
                <el-button @click="Eraser" :type="isEraser" slot="reference" style="margin-right: 10px">Eraser</el-button>
              </el-popover>

              <el-button @click="Del">Clear</el-button>
              <el-button @click="redoDraw">Redo</el-button>
              <el-button @click="undoDraw">Undo</el-button>
              <el-button @click="Transfer" type="success" style="margin-top: 10px">Transfer</el-button>
              <el-button @click="Save" :disabled="!isShow">Save</el-button>

            <div style="font-family: 'Times New Roman';font-size: 20px;margin-top: 20px">
              <el-row :gutter="24">
                <el-col :span="8" style="text-align: center">
                  <a style="font-weight: bold">Gender:</a>
                  <el-radio-group v-model="gender" style="margin-right: 20px;margin-left: 10px">
                    <el-radio :label="0"><a style="font-size: 20px">Female</a></el-radio>
                    <el-radio :label="1"><a style="font-size: 20px">Male</a></el-radio>
                  </el-radio-group>
                </el-col>
                <el-col :span="8">
                  <a style="font-weight: bold;margin-left: 20px">Skin Tone:</a>
                  <el-radio-group v-model="skin" style="margin-right: 20px;margin-left: 10px">
                    <el-radio :label="0"><a style="font-size: 20px">Darker</a></el-radio>
                    <el-radio :label="1"><a style="font-size: 20px">Brighter</a></el-radio>
                  </el-radio-group>
                </el-col>
                <el-col :span="8">
                  <a style="font-weight: bold;margin-left: 20px">Hair Color:</a>
                  <el-radio-group v-model="hair" style="margin-left: 10px">
                    <el-radio :label="0"><a style="font-size: 20px">None Black</a></el-radio>
                    <el-radio :label="1"><a style="font-size: 20px">Black</a></el-radio>
                  </el-radio-group>
                </el-col>
              </el-row>
            </div>
            <br>
            <div style="font-family: 'Times New Roman';font-size: 20px">
              <el-popover
                  :width="600"
                  trigger="hover"
                  placement="top-start"
              >
                <div style="font-family: 'Times New Roman';font-size: 16px">
                  The Dilation Value is between 0 and 1.
                  <br>
                  The smaller the Dilation Value, the higher the contrast between the hand-drawn drawing and
                  <br>
                  the generated drawing;
                </div>
                <span class="el-icon-info" slot="reference"></span>
              </el-popover>

              Dilation Value:
              <el-input-number size="mini" v-model="dilation" :precision="2" :step="0.1" :max="1"></el-input-number>
            </div>


            </div>
          <el-col :span="12">
            <el-card align="center" style="height: 550px">
              <canvas id="canvas" width="512" height="512" style="border-style: dashed"></canvas>
            </el-card>
          </el-col>
          <el-col :span="12">
            <el-card style="height: 550px">
              <div style="border-style: dashed;height: 512px;width: 512px;margin: auto">
                <el-image v-show="isShow" style="height: 512px;width: 512px" :src="'data:image/png;base64,'+this.image"></el-image>
              </div>
            </el-card>
          </el-col>
        </el-row>
      <el-divider></el-divider>
      <div>
        <div style="font-size: 30px;font-family: 'Times New Roman'">
          <div style="text-align: left">Select Examples
          </div>
        </div>
        <br>
        <vue-select-image :dataImages="examples" @onselectimage="onSelectExample" :h="210" :w="210" ref="select">
        </vue-select-image>
      </div>
    </el-main>
  </el-container>
</template>

<script>
import VueSelectImage from 'vue-select-image';
import { fabric } from "fabric";
import axios from "axios";


require('vue-select-image/dist/vue-select-image.css')
export default {
  name: "Home",
  components:{ VueSelectImage},
  data(){
    return{
      publicPath: process.env.BASE_URL,
      canvas: null,
      image: '',
      isDraw: '',
      isEraser: '',
      log: [],
      isShow: false,
      exception: false,
      isRedoing: false,
      isSelect: false,
      imageSelected: [],
      isExamples:false,
      examples: [],
      value1: 10,
      value2: 3,
      gender: 0,
      skin: 0,
      hair: 0,
      dilation: 0,
      visible: true
    }
  },
  mounted() {
    this.canvas = new fabric.Canvas('canvas')
    this.Draw()
    this.canvas.on('object:added', () => {
      if(this.isRedoing === false){
        this.log = [];
      }
    });
    this.getExample()
  },
  watch:{
    value1: function(){
      var ele = document.getElementById('EraserSize')
      ele.style.height = this.value1+ 'px'
      ele.style.width = this.value1 + 'px'
      this.canvas.freeDrawingBrush.width = this.value1
    },
    value2: function(){
      var ele = document.getElementById('BrushSize')
      ele.style.height = this.value2 + 'px'
      ele.style.width = this.value2 + 'px'
      this.canvas.freeDrawingBrush.width = this.value2
    }
  },
  methods:{
    Draw(){
      this.isEraser = ''
      this.isDraw = 'primary'
      this.canvas.freeDrawingBrush.width = this.value2
      this.canvas.isDrawingMode = true;
      this.canvas.freeDrawingBrush.color = '#000000'
      this.canvas.on
    },
    Del() {
      this.canvas.clear()
      this.$refs.select.removeFromSingleSelected()
    },
    Transfer(){
      var dataURL = this.canvas.toDataURL({
        width: this.canvas.width,
        height: this.canvas.height,
        left: 0,
        top: 0,
        format: 'png',
      });
      console.log(dataURL)
      dataURL = dataURL.split(',')[1]
      axios.post('/api/generate', {
        sketch: dataURL,
        attributes: [this.gender, this.hair, this.skin],
        dilation_value: this.dilation
      }).then(res=>{
        if(res.data.code === '200')
        {
          this.image = res.data.result
          this.isShow = true
        }
        else
        {
          if (this.exception)
            alert('error!')
          else
            console.log('error!')
        }
      }).catch(error=>{
        console.log(error)
      })

    },
    Eraser(){
      this.isDraw = ''
      this.isEraser = 'primary'
      this.canvas.isDrawingMode = true
      this.canvas.freeDrawingBrush.width = this.value1
      this.canvas.freeDrawingBrush.color = "#FFFFFF"
    },
    undoDraw(){
      if(this.canvas._objects.length > 0){
        this.log.push(this.canvas._objects.pop());
        this.canvas.renderAll();
      }
    },
    redoDraw(){
      if(this.log.length > 0){
        this.isRedoing = true;
        this.canvas.add(this.log.pop());
        this.canvas.renderAll();
      }
    },
    onSelectExample: function (data) {
      this.isExamples = false
      var imgObj = new Image()
      imgObj.src = data.src
      var image = new fabric.Image(imgObj)
      image.set({
        scaleX:512/512,
        scaleY:512/512
      })
      this.Del()
      this.canvas.centerObject(image)
      this.canvas.add(image)
      this.canvas.renderAll()
    },
    getExample(){
      this.examples = []
      var url = this.publicPath + 'sketch/'
      for(var i = 1; i<11;i++)
      {
        this.examples.push(
            {
              id: i,
              src: url+i.toString()+'.jpg'
            })
      }
    },
    Save(){
      const link = document.createElement('a');
      link.download = 'canvas.png';
      link.href = this.image;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }
  }
}
</script>

<style scoped>
.EraserSize{
  border: 1px solid black;
  border-radius: 50%;
  width: 10px;
  height: 10px;
  -moz-border-radius: 50%;
  -webkit-border-radius: 50%;
}
.BrushSize{
  border: 1px solid black;
  border-radius: 50%;
  width: 3px;
  height: 3px;
  -moz-border-radius: 50%;
  -webkit-border-radius: 50%;
}
</style>