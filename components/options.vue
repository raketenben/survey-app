<template>
    <svg id="options" ref="circle" :viewBox="`0 0 200 200`">
        <text 
            v-for="(option,i) in weightedOptions" 
            class="noselect" 
            :key="option[0]" 
            :x="getPosition(i)[0]" 
            :y="getPosition(i)[1]"
            :class="{favor: option[1]}"
        >
            {{option[0]}}
        </text>
        <text 
            class="noselect center" 
            x="100"
            y="100"
        >
            {{center}}
        </text>
    </svg>
</template>

<script>
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
    props: [
        'options',
        'radius',
        'center',
        'weights',
    ],
    data: () => ({
        size: 200,
    }),
    methods: {
        getPosition: function(i) {
            let interval = Math.PI*2 / this.options.length;
            let angle = i * interval+Math.PI/2;
            let x = Math.cos(angle) * this.radius + this.size/2;
            let y = Math.sin(angle) * this.radius + this.size/2;
            return [x,y];
        },
    },
    computed: {
        currentlyFavored: function() {
            let targets = new Array(this.options.length).fill(0);
            for(let i=0; i<this.weights.length; i++){
                let weight = this.weights[i];
                const weightFactor = weight[1];
                const weightAngle = weight[0];
                
                let closest = [Infinity,null];
                for(let j=0; j<this.options.length; j++){
                    let interval = Math.PI*2 / this.options.length;
                    let angle = j * interval+Math.PI/2;

                    let distance = Math.abs(angle - weightAngle);
                    //correct angle if it's over 180
                    if(distance > Math.PI){
                        distance = Math.PI*2 - distance;
                    }

                    console.log(distance)

                    let adderFactor = Math.max(interval - distance,0);
                    targets[j] += adderFactor * weightFactor;
                }
                
                targets[closest[1]] += weightFactor;
            }
            
            const max = Math.max(...targets);
            const index = targets.indexOf(max);

            console.log(targets)
            
            if(index === -1) return null;
            return index;
        },
        weightedOptions(){
            if(!this.options) return [];
            return this.options.map((option,i) => {
            if(i == this.currentlyFavored)
                return [option[0],true];
            return option;
            });
        }
    },
})
</script>

<style scoped>
    text {
        fill: purple;
        stroke: #aca9fd;
        stroke-width: 0.3px;
        text-anchor: middle;
        font-size:0.5em;
        dominant-baseline: middle;
    }

    .noselect {
        -webkit-touch-callout: none; 
        -webkit-user-select: none;
        -khtml-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

    .favor {
        stroke: white;
        font-size: 0.6em;
    }

    .center {
        font-size: 0.3em;
    }
</style>