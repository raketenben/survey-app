<template>
  <main>
    <svg id="circle" ref="circle" :viewBox="`0 0 ${size} ${size}`">
      <polyline :points="points" :fill="fill" :stroke="stroke"/>
    </svg>
  </main>
</template>

<script>
  export default {
    data() {
      return {
        samples: 100,
        state: "Vue",
        radius: 50,
        size: 200,
        weightFactor: 0.2,
        weightRange: 0.1,
        weights: [],
        fill: "purple",
        stroke:"rgb(53 0 53)",
        lock: false,
      }
    },
    computed: {
      points() {
        let points = new Array(this.samples).fill(0);

        for(let i = 0; i < this.samples; i++) {
          let angle = i * Math.PI * 2 / this.samples;
          points[i] = [angle,1];
        }

        //print weights
        for(let i = 0; i < this.weights.length; i++) {

          let closestIndex = [0,Math.abs(points[0][0]-this.weights[i])];
          for(let j = 0; j < points.length; j++) {
            let distance = Math.abs(points[j][0]-this.weights[i]);

            if(distance < closestIndex[1]) {
              closestIndex = [j,distance];
            }
          }

          for(let z = -(this.weightRange*points.length); z <= (this.weightRange*points.length); z++) {

            let index = closestIndex[0] + z;

            let normalized = Math.abs(z) / (this.weightRange*points.length);

            let factor = (Math.cos(normalized*Math.PI)+1)*0.5;

            if(index >= points.length) {
              index -= points.length;
            }
            if(index < 0) {
              index += points.length;
            }
  
            points[index][1]+= factor*this.weightFactor;
          }
          
        }

        points = points.map((point, i) => {
          let x = Math.cos(point[0]) * this.radius * point[1] + this.size/2;
          let y = Math.sin(point[0]) * this.radius * point[1] + this.size/2;

          return `${x},${y}`;
        });

        return points.join(" ")+" "+points[0];
      },
    },
    methods: {
      updateTouch(e) {
        if(this.lock) {
          return;
        }
        this.lock = true;

        //this.weights = new Array(e.touches.length).fill(0);
        
        for(let i = 0; i < e.touches.length; i++) {
          this.updateWeight(i, e.touches[i]);
        }
        
        setTimeout(() => {
          this.lock = false;
        }, 50);
      },
      updateWeight(index, touch) {
        let x = touch.clientX - window.innerWidth/2;
        let y = touch.clientY - window.innerHeight/2;

        let div = x / y;

        let angle = (Math.PI -Math.atan(div)+Math.PI/2  + ((y < 0) ? -Math.PI : 0)+Math.PI) % (Math.PI*2);

        this.weights[index] = angle;
      },
    },
    mounted() {

      window.addEventListener("touchstart", e => {
        e.preventDefault();
        this.updateTouch(e);
      }, {passive: false});

      window.addEventListener("touchmove", e => {
        e.preventDefault();
        this.updateTouch(e);
      }, {passive: false});

      window.addEventListener("mousemove", e => {
        e.preventDefault();
        
        if(e.buttons === 1) {
          this.updateWeight(0, e);
        }
        
      }, {passive: false});

      /*window.addEventListener('mousemove', e => {
        let x = e.clientX - window.innerWidth/2;
        let y = e.clientY - window.innerHeight/2;

        let div = x / y;

        let angle = (Math.PI -Math.atan(div)+Math.PI/2  + ((y < 0) ? -Math.PI : 0)+Math.PI) % (Math.PI*2);

        this.weights[0] = angle;
        console.log(angle)
      });*/
    },
  }
</script>


<style>
  * {
    padding: 0px;
    margin: 0px;
    box-sizing: border-box;
  }

  html,body,#__nuxt,main {
    height: 100%;
    width: 100%;
    color: white;

    font-family:'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif
  }

  #__nuxt {
    background-size: 100% 100%;
    background-position: 0px 0px,0px 0px,0px 0px;
    background-image: radial-gradient(75vw 75vw at 0% 86%, #510051FF 0%, #FF07C000 99%),radial-gradient(75vw 75vw at 89% 100%, #004C6EFF 0%, #00FFFF00 99%),linear-gradient(0deg, #000000FF 0%, #000000FF 100%);
  }

  div {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  div.container {
    border: 2px solid rgb(90 43 90 / 63%);
    border-radius: 1rem;
  }

  div.wrapper {
    width: 80%;
    height: 80%
  }

  #circle {
    height: 100%;
    aspect-ratio: 1;
    width: auto;
  }

  main {
    display: flex;
    flex-flow: column;
  }

</style>