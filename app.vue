<template>
  <main>
    <Items v-if="setup" @submit="submit"/>
    <div v-else id="circle" @mousedown="mouseDown" @mousemove="mouseMove" @touchmove="touchMove" @touchstart="touchStart">
      <Circle ref="circle" :weights="combinedWeights"/>
      <Option :options="options" :weights="combinedWeights" :radius="50"/>
      <!--:center="`${currentlyJoined}`"-->
    </div>
    <img v-if="deferredPrompt != null" src="/iconmonstr-arrow-down-lined.svg" @click="install" alt="Install" id="install">
    <img v-if="!setup" src="/iconmonstr-share-thin.svg" @click="share" alt="Share" id="share">
  </main>
</template>

<script>
  import Circle from './components/circle.vue'
  import Option from './components/options.vue'
  import Items from './components/items.vue'

  export default {
    data() {
      return {
        servicePrefix: 'survey-',
        weights: [], 
        roomId: null,
        peerId: null,
        peer: null,
        setup: true,
        options: null,
        isHost: false,
        deferredPrompt: null,
        connections:[],
        remoteWeights: {},
        sendTimeout: null,
        sendWaitTime: 50,
        sendBlocked: false,
      }
    },
    components: {
      Circle,
      Option,
      Items
    },
    methods: {
      sendWeights(){
        if(!this.isHost){
          if(this.sendBlocked) return;
          this.sendBlocked = true;
          setTimeout(() => {
            this.sendBlocked = false;
          }, this.sendWaitTime);

          let weights = this.weights.map((w) => {return w[0]});
          this.peer.send({type:"weights",data:weights})
        }
      },
      broadcastWeights(){
        if(this.isHost){
          this.connections.forEach(connection => {
            let allWeights = this.remoteWeights;
            allWeights[this.servicePrefix+this.roomId] = this.weights.map((w) => {return w[0]});
            connection.send({type:"weights",data:allWeights});
          });
        }
      },

      mouseDown(e){
        this.updateWeight(0, e, 1);
        this.sendWeights();
        this.broadcastWeights();
      },
      mouseMove(e){
        if(e.buttons === 1) {
          this.updateWeight(0, e, 1);
          this.sendWeights();
          this.broadcastWeights();
        }
      },
      touchMove(e){
        this.updateTouch(e);
      },
      touchStart(e){
        this.weights = [];
        this.updateTouch(e);
      },
      updateTouch(e) {
        for(let i = 0; i < e.touches.length; i++) {
          this.updateWeight(i, e.touches[i],1/e.touches.length);
        }
        this.sendWeights();
        this.broadcastWeights();
      },
      updateWeight(index, touch, weight) {
        let x = touch.clientX - window.innerWidth/2;
        let y = touch.clientY - window.innerHeight/2;

        let div = x / y;

        let angle = (Math.PI -Math.atan(div)+Math.PI/2  + ((y < 0) ? -Math.PI : 0)+Math.PI) % (Math.PI*2);

        this.weights[index] = [angle,weight];
      },
      submit(items){
        this.setup = false;
        this.options = items.map(item => [item,false]);
      },
      share(){
        navigator.share({
          title: 'Share',
          text: 'Share',
          url: `${window.location.href}?r=${this.roomId}`,
        })
      },
      install(){
        this.deferredPrompt.prompt();
        this.deferredPrompt.userChoice.then((choiceResult) => {
          this.deferredPrompt = null;
        });
      },

      addHostListener(){
        this.peer.on("open", id => {
          this.peerId = id;
        });

        this.peer.on('connection', (conn) => {
          // Handle a peer connecting to us
          conn.on("open", () => {
            console.log("connection to peer open");
            this.connections.push(conn);
            conn.send({type:"setup",data:this.options});
            this.broadcastWeights();

            conn.on("data", (message) => {
              if(message.type === "weights"){
                this.remoteWeights[conn.peer] = message.data;
                this.broadcastWeights();
              }
            });
          });

          conn.on("close", () => {
            this.connections = this.connections.filter(c => c.peer !== conn.peer);
            delete this.remoteWeights[conn.peer];
            this.broadcastWeights();
          });

        });
      },
      addClientListener(){
        this.peer.on("open", () => {
          this.peerId = this.peer.provider._id;

          this.peer.on("data", (message) => {
            console.log("data", message);
            switch (message.type) {
              case "setup":
                this.options = message.data;
                this.setup = false;
                break;
              case "weights":
                this.remoteWeights = message.data;
                break;
              default:
                console.log("unknown message type", message.type);
            }
          });
        });
      },
    },
    async mounted() {
      if(process.client){
        const Peer = (await import('peerjs')).default;

        if(this.$route.query && this.$route.query.r) {
          this.roomId = this.$route.query.r;
          this.peer = new Peer();
          this.peer = this.peer.connect(this.servicePrefix+this.roomId);
          this.addClientListener();
        }else {
          this.roomId = Math.random().toString(36).substring(2, 20);
          this.peer = new Peer(this.servicePrefix+this.roomId);
          this.$route.query = {r: this.roomId};
          this.addHostListener();
          this.isHost = true;
        }
        
        window.addEventListener('beforeinstallprompt', (e) => {
          e.preventDefault();
          this.deferredPrompt = e;
        });

        //service worker
        if('serviceWorker' in navigator) {
          navigator.serviceWorker.register('/sw.js');
        }
      }
    },
    computed: {
      installState(){
        let isStandalone = window.matchMedia('(display-mode: standalone)').matches;
        if (document.referrer.startsWith('android-app://')) {
          return 'twa';
        } else if (navigator.standalone || isStandalone) {
          return 'standalone';
        }
        return 'browser';
      },
      combinedWeights(){
        let combinedWeights = [];
        for (var key in this.remoteWeights) {
          if(key != this.peerId){
            var value = this.remoteWeights[key];
            for(let i = 0; i < value.length; i++){
              combinedWeights.push([value[i],1/value.length]);
            }
          }
        }
        return combinedWeights.concat(this.weights);
      },
      currentlyJoined(){
        let remote = Object.keys(this.remoteWeights).length;
        if(remote < 1) return 1;
        return remote;
      },
    },
    head() {
      return {
        title: 'Survey',
        meta: [
          {
            name: 'viewport',
            content: 'width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no' 
          },
          {
            hid: 'description',
            name: 'description',
            content: 'Survey'
          },
        ],
        link: [
          {
            rel:"manifest", 
            href:"/manifest.webmanifest"
          },
          {
            rel:"icon", 
            href:"/favicon.ico"
          }
        ]
      }
    }
  }
</script>


<style>
  @import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@700&display=swap');

  * {
    padding: 0px;
    margin: 0px;
    box-sizing: border-box;
    font-size: 1.4rem;
  }

  html,body,#__nuxt,main {
    height: 100%;
    width: 100%;
    touch-action: none;
    color: white;
    font-family: 'Open Sans','Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif
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

  #circle > * {
    height: 100%;
    width: 100%;
    position: absolute;
  }

  main {
    display: flex;
    flex-flow: column;
  }

  #share {
    position: absolute;
    bottom: 1rem;
    right: 1rem;
    padding: 0.5rem;
    border-radius: 0.5rem;
    border: solid 2px rgba(255, 255, 255, 0.7);
    cursor: pointer;
    width: 3rem;
    height: 3rem;
  }

  #install {
    position: absolute;
    bottom: 1rem;
    left: 1rem;
    padding: 0.5rem;
    border-radius: 0.5rem;
    border: solid 2px rgba(255, 255, 255, 0.7);
    cursor: pointer;
    width: 3rem;
    height: 3rem;
  }
</style>