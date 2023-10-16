<template>
    <v-toolbar style = "position : absolute; top : 0px; left : 0px; height: 7%">
            <v-btn @click = "backToMenu()">
                <v-icon> mdi-arrow-left-bold-circle</v-icon>
            </v-btn>
            <v-app-bar-title>{{name}}</v-app-bar-title>
            <v-btn @click = "togglePause()">
                <v-icon>mdi-pause-octagon</v-icon>
            </v-btn>
            <v-btn @click = "scores()">
                HighScores
            </v-btn>
        </v-toolbar>
        
</template>

<script>
    import * as THREE from 'three'
    import {CSS2DRenderer, CSS2DObject} from 'three/examples/jsm/renderers/CSS2DRenderer' 
    export default{
        name: "SpaceShoot",
        props: ['name','gameId', 'user', 'userId'],
        mounted () {     //cand se incarca pagina se apeleaza ce e in mounted/
            this.init()
        },
        data () {
            return{
                keyboard: {},
                playerSpeed: 0.4,
                spawnPoint: new THREE.Vector3(0, -0.45, 1),
                time: 0,
                time1: 0,
                playerState: 0,
                playerCounter: 0,
                request: 0,
                indicator: 0,
                isPaused: false,
                score: 0,
                diamondCollected: false,
                score1: 0,
                sprites: [],
                playerState: 0,
                playerCounter: 0,
            }
        },
        methods: {
            init () {
                //Event listener
                window.addEventListener('keydown', this.keyDown) //Cand e apasata tasta
                window.addEventListener('keyup', this.keyUp) //Cand nu e apasata
                //Create scene/
                this.scene = new THREE.Scene()

                //Clock
                this.clock = new THREE.Clock()

                //Create camera
                this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
                this.camera.position.y = 1.5
                this.camera.position.z = 5
                this.camera.lookAt(new THREE.Vector3(0, 1.5, 0))

                //Background
                var textureLoader = new THREE.TextureLoader()
                
                var backgroundTexture = textureLoader.load('/WEBDEV_POZE/mazerunner.jpg')
                backgroundTexture.wrapS = THREE.RepeatWrapping
                
                this.backgroundMesh = new THREE.Mesh(
                    new THREE.PlaneGeometry(20, 10, 100, 100),
                    new THREE.MeshBasicMaterial({
                        map: backgroundTexture
                    })
                )

                this.backgroundMesh.position.y = 2
                this.scene.add(this.backgroundMesh)  //Adaugam ce am facut in scena

                //Player
                //var playerTexture = textureLoader.load('/WEBDEV_POZE/rocket.png')
                this.sprites.push(textureLoader.load('/WEBDEV_POZE/m11.png'))
                this.sprites.push(textureLoader.load('/WEBDEV_POZE/m22.png'))
                this.sprites.push(textureLoader.load('/WEBDEV_POZE/m33.png'))
                this.sprites.push(textureLoader.load('/WEBDEV_POZE/m44.png'))
                this.playerMesh = new THREE.Mesh(
                        new THREE.PlaneGeometry(1, 1, 50, 50), //50 de triunghiuri pe lungime si latime
                        new THREE.MeshBasicMaterial({
                            map: this.sprites[this.playerState],
                            transparent: true
                        })
                )

                this.playerMesh.position.z = 1
                this.playerMesh.position.x = -1.5
                this.playerMesh.position.y = 1
                this.scene.add(this.playerMesh)

                //MONSTER
                this.monsterMesh = new THREE.Mesh(
                        new THREE.PlaneGeometry(1, 1, 50, 50), //50 de triunghiuri pe lungime si latime
                        new THREE.MeshBasicMaterial({
                            map: textureLoader.load('/WEBDEV_POZE/monster.png'),
                            transparent: true
                        })
                )

                this.monsterMesh.position.z = 1
                this.monsterMesh.position.x = -4
                this.monsterMesh.position.y = 1
                this.scene.add(this.monsterMesh)

                

                //WALL 1

                this.wall1Mesh = new THREE.Mesh(
                        new THREE.PlaneGeometry(100, 2, 50, 50), //50 de triunghiuri pe lungime si latime
                        new THREE.MeshBasicMaterial({
                            map: textureLoader.load('/WEBDEV_POZE/wall.jpg'),
                            transparent: true
                        })
                )

                this.wall1Mesh.position.z = 1
                this.wall1Mesh.position.x = -4
                this.wall1Mesh.position.y = 4.5
                this.scene.add(this.wall1Mesh)


                //WALL 2
                this.wall2Mesh = new THREE.Mesh(
                        new THREE.PlaneGeometry(1000, 2, 50, 50), //50 de triunghiuri pe lungime si latime
                        new THREE.MeshBasicMaterial({
                            map: textureLoader.load('/WEBDEV_POZE/wall.jpg'),
                            transparent: true
                        })
                )

                this.wall2Mesh.position.z = 1
                this.wall2Mesh.position.x = -4
                this.wall2Mesh.position.y = -1.5
                this.scene.add(this.wall2Mesh)




                //Obstacle
                this.slower = new THREE.Mesh(
                    new THREE.PlaneGeometry(1, 1, 50, 50),
                    new THREE.MeshBasicMaterial({
                        map: textureLoader.load('/WEBDEV_POZE/slowdown.png'),
                        transparent: true
                    })
                )
                this.slower.position.x = this.spawnPoint.x + 2
                this.slower.position.y = this.spawnPoint.y + 2
                this.slower.position.z = this.spawnPoint.z
                this.scene.add(this.slower)
                
                //diamond
                
                this.diamond = new THREE.Mesh(
                    new THREE.PlaneGeometry(1, 1, 50, 50),
                    new THREE.MeshBasicMaterial({
                        map: textureLoader.load('/WEBDEV_POZE/diamond.png'),
                        transparent: true
                    })
                )
                this.diamond.position.x = this.spawnPoint.x + 2
                this.diamond.position.y = this.spawnPoint.y + 3.3
                this.diamond.position.z = this.spawnPoint.z
                this.scene.add(this.diamond)

                //HEAL
                this.heal = new THREE.Mesh(
                    new THREE.PlaneGeometry(1, 1, 50, 50),
                    new THREE.MeshBasicMaterial({
                        map: textureLoader.load('/WEBDEV_POZE/heal.png'),
                        transparent: true
                    })
                )
                this.heal.position.x = this.spawnPoint.x + 2
                this.heal.position.y = this.spawnPoint.y + 0.65
                this.heal.position.z = this.spawnPoint.z
                this.scene.add(this.heal)

                
                //Text renderer
                this.timerDiv = document.createElement('div')
				this.timerDiv.id = 'timerDiv'
				this.timerDiv.textContent = 'Time: 0'
				this.timerDiv.style.backgroundColor = 'transparent'
                this.timerDiv.style.fontSize = '30px'
                this.timerDiv.style.color = 'white'
				const timerLabel = new CSS2DObject(this.timerDiv)
				timerLabel.position.set(0, 0, 0)
				this.scene.add(timerLabel)
				timerLabel.layers.set(0)

                this.scoreDiv = document.createElement('div')
				this.scoreDiv.id = 'scoreDiv'
				this.scoreDiv.textContent = 'Score: 0'
				this.scoreDiv.style.backgroundColor = 'transparent'
                this.scoreDiv.style.fontSize = '30px'
                this.scoreDiv.style.color = 'white'
				const scoreLabel = new CSS2DObject(this.scoreDiv)
				scoreLabel.position.set(0, 3, 0)
				this.scene.add(scoreLabel)
				scoreLabel.layers.set(0)

                this.labelRenderer = new CSS2DRenderer()
                this.labelRenderer.setSize(window.innerWidth * 15 / 100, window.innerHeight * 10 / 100)
                this.labelRenderer.domElement.id = "timerLabel"
                this.labelRenderer.domElement.style.position = "absolute"
                this.labelRenderer.domElement.style.left = "5%"
                this.labelRenderer.domElement.style.top = "9%"
                document.body.appendChild(this.labelRenderer.domElement)


                //Rednderer setup/
                this.renderer = new THREE.WebGL1Renderer()
                this.renderer.setSize(window.innerWidth * 90 / 100, window.innerHeight * 90 / 100)
                document.body.appendChild(this.renderer.domElement)
                //STABILIRE ECRAN:
                this.renderer.domElement.id = "canvas"
                this.renderer.domElement.style.position = "absolute"
                this.renderer.domElement.style.left = "5%"
                this.renderer.domElement.style.right = "5%"
                this.renderer.domElement.style.top = "10%"

                this.animate()
            }, 

            animate(){
                if(this.isPaused == false)
                {
                    this.request = requestAnimationFrame(this.animate)
                    this.delta = this.clock.getDelta()
                    this.backgroundMesh.material.map.offset.x += this.playerSpeed * this.delta
                    this.scoreDiv.textContent = "Diamonds: "+ this.score.toFixed(0).toString()

                    this.time += this.delta
                    
                    this.playerCounter += this.delta
                    this.timerDiv.textContent = "Time: "+ this.time.toFixed(1).toString()


                if (this.playerCounter >= 1)
                    {
                        this.playerCounter = 0
                        this.playerState += 1
                        this.playerState %= 4
                        this.playerMesh.material.map = this.sprites[this.playerState]
                    }

                    if(this.keyboard[39]){
                        if(this.playerSpeed < 0.6)
                        {
                        this.playerSpeed += 2 * this.delta
                        }
                    }else
                    {
                        //this.playerSpeed -= 5 * this.delta
                        if (this.playerSpeed > 0.1)
                        {
                            this.playerSpeed -= 1.5 * this.delta
                        }
                    }

                    if (this.keyboard[38]) { 
                            this.playerMesh.position.y += 0.1
                            this.monsterMesh.position.y += 0.1
                    }
                    if(this.keyboard[40]){
                            this.playerMesh.position.y -= 0.1
                            this.monsterMesh.position.y -= 0.1
                    }

                    //Obstacle movement
                    this.slower.position.x -= this.playerSpeed * this.delta * 10
                    this.diamond.position.x -= this.playerSpeed * this.delta * 10
                    this.heal.position.x -= this.playerSpeed * this.delta * 10


                    //Scapam de piatra dupa ce am trecut de ea ca sa nu umplem memoria
                    if(this.slower.position < -5){
                        this.slower.material.dispose()
                        this.slower.geometry.dispose()
                        this.slower.removeFromParent()
                    }

                    //Enemy collision
                    var playerBoundingBox = new THREE.Box3().setFromObject(this.playerMesh)

                    var slowerBoundingBox = new THREE.Box3().setFromObject(this.slower)
                    var collision = playerBoundingBox.intersectsBox(slowerBoundingBox)

                    //diamond collision

                     var diamondBoundingBox = new THREE.Box3().setFromObject(this.diamond)
                     var collision2 = playerBoundingBox.intersectsBox(diamondBoundingBox)
                                    
                    

                    if(collision == true)
                    {
                       // this.isPaused = true
                       // console.log("Game over")
                        //console.log(this.rock.position.x)
                        //this.firstRockPosition = this.rock.position.x
                        //console.log(this.score.toFixed(0))
                        this.monsterMesh.position.x += 0.01
                        
                        //this.playerSpeed = 0
                             
                        
                    }

                    if(collision2 == true && this.diamondCollected == false)
                    {
                       // this.isPaused = true
                        console.log("diamond collected")
                        this.score = (this.score + 1)
                        this.diamondCollected = true
                             
                    }
                    if(collision2 == false)
                    {
                        this.diamondCollected = false
                    }

                    //Wall1 collision

                    var wall1BoundingBox = new THREE.Box3().setFromObject(this.wall1Mesh)
                    var collision3 = playerBoundingBox.intersectsBox(wall1BoundingBox)

                    if(collision3 == true)
                    {
                       // this.isPaused = true
                       // console.log("Game over")
                        //console.log(this.rock.position.x)
                        //this.firstRockPosition = this.rock.position.x
                        //console.log(this.score.toFixed(0))
                        this.monsterMesh.position.x += 0.03
                        
                        //this.playerSpeed = 0
                             
                        
                    }


                    //Wall2 collision

                    var wall2BoundingBox = new THREE.Box3().setFromObject(this.wall2Mesh)
                    var collision4 = playerBoundingBox.intersectsBox(wall2BoundingBox)

                    if(collision4 == true)
                    {
                       // this.isPaused = true
                        //console.log("Game over")
                        //console.log(this.rock.position.x)
                        //this.firstRockPosition = this.rock.position.x
                        //console.log(this.score.toFixed(0))
                        this.monsterMesh.position.x += 0.03
                        
                        //this.playerSpeed = 0
                             
                        
                    }

                    //MONSTER collision

                    var monsterBoundingBox = new THREE.Box3().setFromObject(this.monsterMesh)
                    var collision5 = playerBoundingBox.intersectsBox(monsterBoundingBox)

                    if(collision5 == true)
                    {
                       // this.isPaused = true
                       console.log("Diamonds collected:")
                        //console.log(this.rock.position.x)
                        //this.firstRockPosition = this.rock.position.x
                        console.log(this.score.toFixed(0))
                        console.log("Distance ran:")
                        console.log(this.time.toFixed(0))
                        cancelAnimationFrame(this.request)
                        
                        //this.playerSpeed = 0
                             
                        
                    }

                    //HEAL collision

                    var healBoundingBox = new THREE.Box3().setFromObject(this.heal)
                    var collision6 = playerBoundingBox.intersectsBox(healBoundingBox)

                    if(collision6 == true)
                    {
                       
                        this.monsterMesh.position.x -= 0.01
                        
                        //this.playerSpeed = 0
                             
                        
                    }

                    

                    

                    var i = this.generateRandomBinary()
                    //slower spawn
                    
                        if(this.spawnPoint.x - this.slower.position.x > 7)
                        {
                            this.slower.position.x = this.spawnPoint.x + 2
                            
                             if(i == 0)
                            {
                                this.slower.position.y = this.spawnPoint.y + 2
                            }
                            else if(i == 1){
                                this.slower.position.y = this.spawnPoint.y + 3.3
                            }
                            else{
                                this.slower.position.y = this.spawnPoint.y + 0.65
                            }

                            this.slower.position.z = this.spawnPoint.z
                            this.scene.add(this.slower) 
                        
                        }

                        //diamond spawn

                        if(this.spawnPoint.x - this.diamond.position.x > 7)
                        {
                            this.diamond.position.x = this.spawnPoint.x + 2
                            if(i == 0)
                            {
                                this.diamond.position.y = this.spawnPoint.y + 0.65
                            }
                            else if(i ==1){
                                this.diamond.position.y = this.spawnPoint.y + 2
                            }
                            else{
                                this.diamond.position.y = this.spawnPoint.y + 3.3
                            }
                            this.diamond.position.z = this.spawnPoint.z
                            this.scene.add(this.diamond) 
                        
                       }

                       //heal spawn

                       if(this.spawnPoint.x - this.heal.position.x > 7)
                        {
                            this.heal.position.x = this.spawnPoint.x + 2
                            if(i == 0)
                            {
                                this.heal.position.y = this.spawnPoint.y + 3.3
                            }
                            else if(i ==1){
                                this.heal.position.y = this.spawnPoint.y + 0.65
                            }
                            else{
                                this.heal.position.y = this.spawnPoint.y + 2
                            }
                            this.heal.position.z = this.spawnPoint.z
                            this.scene.add(this.heal) 
                        
                       }
                    
                    

                    this.renderer.render(this.scene, this.camera)
                    this.labelRenderer.render(this.scene, this.camera)

                }
                
            },

            

            keyDown(event){
                this.keyboard[event.keyCode] = true;
            },

            keyUp(event){
                this.keyboard[event.keyCode] = false;
            },

            backToMenu(){
                this.$emit("changeState", 1)
                document.getElementById('canvas').remove()
                document.getElementById('timerLabel').remove()
                document.getElementById('timerDiv').remove()

            },

            scores(){
                this.$emit("changeState", 4)
                document.getElementById('canvas').remove()
                document.getElementById('timerLabel').remove()
                document.getElementById('timerDiv').remove()
            },

            generateRandomBinary() {
                return Math.floor(Math.random() * 2);
            },

            // togglePause() {
            //     this.isPaused = !this.isPaused
            //     if (this.isPaused) {
            //         cancelAnimationFrame(this.request)
            //         this.time1 = this.time
            //         this.playerSpeed = 0
            //     } else {
            //         this.animate()
            //         this.playerSpeed = 0.1
            //         this.time = this.time1
            //     }
            //},
            mounted() {
                this.init();
                this.animate(); // Start the animation loop
            },

            generateRandomBinary() {
                return Math.floor(Math.random() * 3)
            },

            togglePause() {
                this.isPaused = !this.isPaused
                if (this.isPaused) {
                    cancelAnimationFrame(this.request)
                    this.time1 = this.time
                    this.score1 = this.score
                    this.playerSpeed = 0
                } else {
                    this.animate()
                    this.time = this.time1
                    this.score = this.score1
                    this.playerSpeed = 0.4
                }
            },
        }
    }
</script>