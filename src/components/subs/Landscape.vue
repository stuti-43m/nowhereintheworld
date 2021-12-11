<template>
    <div id="container"
      v-bind:style ="dimensions"
    > </div>
</template>

<script>
import * as Three from 'three'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js'
import { noise } from "./../../helpers/perlin.js";
//import {RenderPass} from 'three/examples/jsm/postprocessing/RenderPass.js'
//import {UnrealBloomPass} from 'three/examples/jsm/postprocessing/UnrealBloomPass.js'
//import {EffectComposer} from 'three/examples/jsm/postprocessing/EffectComposer.js'
//let clock;
let mainColour;
let subColour;
let colourS;
let colourL;
let theScene
let camera; 
let renderer;
//let effectComposer;

let terrain;
let ballMesh;
let planeGeom;
let mainText;
let timeText;
let gmtText;
let gmtOffText;
let uniformsC;
let light=[];

let container;
let vcount;
let compcolour;
let font;
//let count = 0;
let t =0;
export default {
  name: 'Landscape',
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null,
      sun: null,
      //terrain: null,
      sky:null,
    }
  },
  props: ['time','hour','minute','gmthour','gmtmin','sign'],
  watch: {
        time: function() {
          this.updateScene();
        }
  },
  methods: {
        generateColours: function() {
             //random colours
            mainColour = Math.floor(Math.random()*360);
            subColour = (mainColour + 30)%360;
            colourS = (Math.floor(15+Math.random()*40))%50;
            colourL = (Math.floor(15+Math.random()*40))%90;
            compcolour= (mainColour+200)%360;
        },
        init: async function() {
            this.generateColours();
            //clock = new Three.Clock()
            container = document.getElementById('container');
            theScene = new Three.Scene();
            camera = new Three.PerspectiveCamera(45, container.clientWidth/container.clientHeight, 1, 2100);
            camera.position.set(0, 0, 50);
            camera.layers.enable(1);
            renderer = new Three.WebGLRenderer({antialias: true});
            renderer.setSize(container.clientWidth, container.clientHeight);
            /** COMPOSER */

           

            const ambientLight = new Three.HemisphereLight(
                0.3 // intensity
            )
            ambientLight.position.x = 0;
            ambientLight.position.y = 4;
            ambientLight.position.z = 0;

            //directional light
            const mainLight = new Three.DirectionalLight(0xe6961c, 0.5);
            mainLight.position.set(0, 4, 0);
            mainLight.target.position.set(300, 400, 200);
            theScene.add(ambientLight, mainLight);
                
            renderer.gammaInput = true
            renderer.gammaOutput = true
            renderer.toneMappingExposure = Math.pow( 0.9, 4.0 ) 
            container.appendChild(renderer.domElement);
            this.addLights();
            this.generateTerrain();
            this.generateBall();
            this.generateSky();
            await this.addtext();
            renderer.gammaInput = true
            renderer.gammaOutput = true
            renderer.toneMappingExposure = Math.pow( 0.9, 4.0 ) 
           renderer.setAnimationLoop(() => {
                this.render();
            })
        },


        //helpers
        
        updateScene: function() {
            //
            //console.log(this.props.time);
            this.generateColours();
            //clear geometry
            terrain.geometry.dispose();
            terrain.material.dispose();
            theScene.remove( terrain );
            ballMesh.geometry.dispose();
            ballMesh.material.dispose();
            theScene.remove( ballMesh );
            this.generateTerrain();
            this.generateBall();
            //update lights
            for(let i=0;i<light.length;i++) {
                light[i].color = new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`);
            }
            //update landscape colour 
            uniformsC.color1.value = new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`); 
            uniformsC.color2.value = new Three.Color(`hsl(${subColour}, 50%, 50%)`); 

            //test font 
            console.log(this.sign);
            this.updateText({
                time:this.getText(this.hour.hour)+':'+this.getText(this.minute.minute),
                offest:this.sign.sign+this.getText(this.gmthour.gmthour)+':'+this.getText(this.gmtmin.gmtmin)
            })

        },
        getText: function(val) {
            val = val.toString();
            if(val.length == 1) {
                return '0'+val;
            } else {
                return val.toString();
            }
        },
        updateText: function(textObj) {
            if(font) {
                if(timeText) {
                    timeText.geometry.dispose();
                    //timeText.material.dispose();
                    theScene.remove( timeText );
                } 
                
                if(gmtText) {
                    gmtText.geometry.dispose();
                    //gmtText.material.dispose();
                    theScene.remove( gmtText );
                }
                
                if(gmtOffText) {
                    gmtOffText.geometry.dispose();
                    //gmtText.material.dispose();
                    theScene.remove( gmtOffText );
                }
                let yoff = 0.25;
                let timeTextGeom = new TextGeometry(textObj.time, {
                    font:font,
                    size:1,
                    height:0.02
                })

                timeText = new Three.Mesh(timeTextGeom,[
                    new Three.MeshBasicMaterial({ color: "#ffffff",reflectivity:0 })
                ])
                timeText.geometry.computeBoundingBox ();
                console.log('BOUNDING BOX',timeText.geometry.boundingBox);
                let xoff = timeText.geometry.boundingBox.max.x
                timeText.position.x = -1.3;
                timeText.position.y = -0.1+yoff;
                timeText.position.z = 40;
                //timeText.layers.enable(1);
                theScene.add(timeText);
                let gmtGeom = new TextGeometry("GMT", {
                    font:font,
                    size:0.17,
                    height:0.02
                })
                gmtText = new Three.Mesh(gmtGeom,[
                    new Three.MeshBasicMaterial({ color: "#ffffff" })
                ])
                gmtText.position.x = xoff-1.2;
                gmtText.position.y = 0.69+yoff;
                gmtText.position.z = 40;
                theScene.add(gmtText);

                let gmtOffGeom = new TextGeometry(textObj.offest, {
                    font:font,
                    size:0.12,
                    height:0.02
                })
                gmtOffText = new Three.Mesh(gmtOffGeom,[
                    new Three.MeshBasicMaterial({ color: "#ffffff" })
                ])
                gmtOffText.position.x = xoff-0.8;
                gmtOffText.position.y = 0.74+yoff;
                gmtOffText.position.z = 40;
                theScene.add(gmtOffText)
            }
        },
        async addtext() {
            //let yoff = 0.25;3
            let fontloader = new FontLoader();
            await fontloader.load('/font/digital.json',function(result) {
                font = result;
                let mainTextGeon = new TextGeometry("NOWHERE IN THE WORLD BUT HERE", {
                    font:result,
                    size:0.13,
                    height:0.02
                })
                mainText = new Three.Mesh(mainTextGeon,[
                    new Three.MeshBasicMaterial({ color: "#ffffff" })
                ])
                mainText.position.x = -1;
                mainText.position.y = 3;
                mainText.position.z = 40;
                theScene.add(mainText);

                let mainTextGeomInv = new TextGeometry("NOWHERE IN THE WORLD", {
                    font:result,
                    size:0.13,
                    height:0.02
                })
                let mainTextInv = new Three.Mesh(mainTextGeomInv,[
                    new Three.MeshBasicMaterial({ color: "#ffffff" })
                ])
                mainTextInv.position.x = 1.;
                mainTextInv.position.y = -2.5;
                mainTextInv.position.z = 40;
                mainTextInv.rotation.z = -Math.PI;
                theScene.add(mainTextInv)
            })
        },
        addLights() {
            let pl1 = new Three.PointLight( new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 1,3 );
            pl1.position.set( -12, 0, 30 );

            let pl2 = new Three.PointLight( new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 1,3 );
            pl2.position.set( -4, 6, 30 );

            let pl3 = new Three.PointLight( new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 1,3 );
            pl3.position.set( 0, 6, 30 );

            let pl4 = new Three.PointLight( new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 1,3 );
            pl4.position.set( 6, 0, 30 );

            let pl6 = new Three.PointLight( new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 1,3 );
            pl6.position.set( 12, 0, 30 );

            let pl5 = new Three.PointLight( new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 1,3 );
            pl5.position.set( -16, 10, 6 );

            theScene.add( pl1);
            theScene.add( pl2);
            theScene.add( pl3);
            theScene.add( pl4);
            light.push(pl1);
            light.push(pl2);
            light.push(pl3);
            light.push(pl4);
            light.push(pl5);
            light.push(pl6);
            //theScene.add( pl5);
            //theScene.add( pl6);
        },
        generateTerrain() {
            //let colour = pick a random colour
            vcount = Math.floor(Math.random()*5000+2000)
            planeGeom = new Three.PlaneGeometry( container.clientWidth,container.clientHeight/40,vcount,500);
            let planeMaterial = new Three.MeshPhongMaterial({
                color:new Three.Color(`hsl(${mainColour}, ${(colourS)}%, ${colourL}%)`), 
                specular:  new Three.Color(`hsl(${subColour}, ${(colourS)}%, ${colourL}%)`),
                shininess: 4,
                side:      Three.DoubleSide,
                flatShading:true
            });
            terrain = new Three.Mesh( planeGeom, planeMaterial );
            terrain.rotation.x =  Math.PI / 1.4;
            terrain.position.y =  -4;
            terrain.position.z =  30;
            theScene.add( terrain );
        },
        generateSky() {
            let skyGeom = new Three.PlaneGeometry( container.clientWidth,15,75,75);
            uniformsC = {
                color1: {
                //value: new Three.Color("#044d23")
                value: new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`)
                },
                color2: {
                //value: new Three.Color("#01170a")
                value: new Three.Color(`hsl(${subColour}, 50%, 50%)`)
                }
            };
            let skyMaterial = new Three.ShaderMaterial({
                uniforms: uniformsC,
                vertexShader: `
                    varying vec2 vUv;

                    void main() {
                    vUv = uv;
                    gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
                    }
                `,
                fragmentShader: `
                    uniform vec3 color1;
                    uniform vec3 color2;
                
                    varying vec2 vUv;
                    
                    void main() {
                    
                    gl_FragColor = vec4(mix(color1, color2, vUv.y), 1.);
                    }
                `
            });
            let sky = new Three.Mesh( skyGeom, skyMaterial );
            sky.position.y = 10;
                        //terrain.rotation.x =  -Math.PI / 2.8;
            theScene.add( sky );
        },
        generateBall() {
            let ball = new Three.SphereGeometry( 4, 32, 16 );
            let ballMaterial = new Three.MeshPhongMaterial({
                color:     new Three.Color(`hsl(${compcolour}, 80%, 40%)`), 
                specular:  new Three.Color(`hsl(${(compcolour+30)%360}, 100%, 40%)`),
                shininess: 15,
                side:      Three.DoubleSide,
                smoothShading:true
            });
            ballMesh = new Three.Mesh( ball, ballMaterial );
            ballMesh.position.y =5;
            ballMesh.layers.enable(2);
            let pl = new Three.PointLight(new Three.Color(`hsl(${mainColour}, ${colourS}%, ${colourL}%)`), 9,3000 );
            pl.position.x = ballMesh.position.x;
            pl.position.y = ballMesh.position.y;
            pl.position.z = 30;
            //ballMesh.position.x = (Math.random()-0.5)*15;
            theScene.add(ballMesh  );
        },
        animateLandscape() {
            //noise.seed(Math.random());
           let pos = planeGeom.getAttribute("position");
            let pa = pos.array;
            const hVerts = planeGeom.parameters.heightSegments + 1;
            const wVerts = planeGeom.parameters.widthSegments + 1;
            for (let j = 0; j < hVerts; j++) {
                for (let i = 0; i < wVerts; i++) {
                    const ex = 1.1;
                    pa[3 * (j * wVerts + i) + 2] =
                    (noise.simplex3(i / 100, j / 100,t) +
                        noise.simplex3((i + 200) / 50, j / 50,t) * Math.pow(ex, 1) +
                        noise.simplex3((i + 400) / 25, j / 25,t) * Math.pow(ex, 2) +
                        noise.simplex3((i + 600) / 12.5, j / 12.5,t) * Math.pow(ex, 3) +
                        +(0.06*noise.simplex3((i + 800) / 6.25, j / 6.25,t) * Math.pow(ex, 4))) /
                    2;
                }
            }
            //count++;
            //console.log(count)
            planeGeom.attributes.position.needsUpdate = true;
        },
        animateText() {
            if(timeText) {
                timeText.position.y = timeText.position.y+0.008*Math.sin(t*10)
            }
        },
        render () {
            //var time = Date.now() / 1000;
            //console.log(theScene.children[2])
           
            t = t+0.03;
            this.animateLandscape();
            this.animateText();
           renderer.render(theScene, camera);
        }
    },
    mounted() {
        this.init();
    },
    computed: {
        dimensions() {
            return {   height: '900px',
                      width: '1400px' };
      }
    }
}
</script>
<style scoped>
  
</style>