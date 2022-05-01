<template>
    <svg id="circle" ref="circle" :viewBox="`0 0 200 200`">
      <path id="path" :d="points" fill="url(#fill)" stroke="url(#outline)" />
      <defs>
        <linearGradient id="fill" x1="0%" y1="0%" x2="100%" y2="0%">
          <stop offset="0%"   stop-color="#6384FFFF">
            <animate attributeName="stop-color" values="#6384FFFF; #AD25D4FF;  #AD25D4FF; #6384FFFF" dur="8s" repeatCount="indefinite"></animate>
          </stop>
          <stop offset="100%" stop-color="#AD25D4FF">
            <animate attributeName="stop-color" values="#AD25D4FF; #6384FFFF; #6384FFFF; #AD25D4FF" dur="8s" repeatCount="indefinite"></animate>
          </stop>
        </linearGradient>
        <linearGradient id="outline" x1="0%" y1="0%" x2="100%" y2="0%">
          <stop offset="100%"   stop-color="#6384FFFF">
            <animate attributeName="stop-color" values="#6384FFFF; #AD25D4FF;  #AD25D4FF; #6384FFFF" dur="8s" repeatCount="indefinite"></animate>
          </stop>
          <stop offset="0%" stop-color="#AD25D4FF">
            <animate attributeName="stop-color" values="#AD25D4FF; #6384FFFF; #6384FFFF; #AD25D4FF" dur="8s" repeatCount="indefinite"></animate>
          </stop>
        </linearGradient>
      </defs>
    </svg>
</template>

<script>
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
    props: ['weights','weight'],
    data: () => ({
        weightRange: 0.1,
        baseLine: null,
    }),
    created() {
        const size = 200;
        const samples = 200;
        const radius = 70;

        this.baseLine = new Array(samples).fill(0);

        for(let i = 0; i < samples; i++) {
            let angle = i * Math.PI * 2 / samples;

            let x = Math.cos(angle) * radius + size/2;
            let y = Math.sin(angle) * radius + size/2;
            
            this.baseLine[i] = [x,y];
        }
    },
    computed: {
        weightFactor() {
            return this.weight || 15;
        },
        adderLine(){
            let adderLine = new Array(this.baseLine.length).fill([0,0]);

            const range = this.weightRange*this.baseLine.length;

            const length = this.baseLine.length;

            for (let i in this.weights){
                let weight = this.weights[i];

                for(let z = -range; z <= range; z++) {

                    let index = Math.round( (weight[0]*length)/(Math.PI*2) + z);

                    if(index >= length) index -= length;
                    if(index < 0) index += length;

                    let normalized = Math.abs(z) / (this.weightRange*length);
                    let factor = (Math.cos(normalized*Math.PI)+1)*0.5;

                    adderLine[index] = [
                        adderLine[index][0]+Math.cos(weight[0])*weight[1]*factor*this.weightFactor,
                        adderLine[index][1]+Math.sin(weight[0])*weight[1]*factor*this.weightFactor
                    ];
                }
            }

            return adderLine
        },
        points() {
            let points = this.baseLine.map(([bx,by],i) => {
                let [x,y] = [
                    bx + this.adderLine[i][0],
                    by + this.adderLine[i][1]
                ];
                return `${x} ${y}`;
            });

            return `M ${points.join(" L ")} Z`;
        }
    }
})
</script>

<style scoped>
  #path {
    stroke: #80008087;
    stroke-width: 4px;
  }
</style>