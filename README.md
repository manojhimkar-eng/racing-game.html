<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>3D Racing Game</title>
<style>
body { margin:0; overflow:hidden; background:black; font-family:Arial; }
#hud {
    position:absolute; top:10px; left:10px;
    color:white; background:rgba(0,0,0,0.6);
    padding:10px;
}
#msg {
    position:absolute; inset:0;
    display:flex; align-items:center; justify-content:center;
    color:white; font-size:60px;
    background:rgba(0,0,0,0.7);
    z-index:10;
}
</style>
</head>

<body>

<div id="hud">
Speed: <span id="speed">0</span><br>
Lap: <span id="lap">1</span><br>
Checkpoint: <span id="cp">0</span>/4
</div>

<div id="msg">3</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.155.0/build/three.min.js"></script>
<script>
/* ===== SETUP ===== */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff,0.6));
const sun = new THREE.DirectionalLight(0xffffff,0.6);
sun.position.set(10,20,10);
scene.add(sun);

/* ===== GROUND ===== */
const ground = new THREE.Mesh(
    new THREE.PlaneGeometry(500,500),
    new THREE.MeshStandardMaterial({color:0x2e8b57})
);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

/* ===== CITY ===== */
for(let i=0;i<60;i++){
    const b = new THREE.Mesh(
        new THREE.BoxGeometry(
            Math.random()*5+3,
            Math.random()*20+5,
            Math.random()*5+3
        ),
        new THREE.MeshStandardMaterial({color:0x555555})
    );
    b.position.set(
        Math.random()*200-100,
        b.geometry.parameters.height/2,
        Math.random()*200-100
    );
    scene.add(b);
}

/* ===== CAR ===== */
function createCar(color){
    const car = new THREE.Group();
    const body = new THREE.Mesh(
        new THREE.BoxGeometry(2,1,4),
        new THREE.MeshStandardMaterial({color})
    );
    body.position.y=0.75;
    car.add(body);
    return car;
}

const player = createCar(0xff0000);
scene.add(player);

const ai = createCar(0x0000ff);
ai.position.set(5,0,0);
scene.add(ai);

/* ===== CHECKPOINTS ===== */
const checkpoints = [];
const cpPositions = [
    [0,0,50],
    [50,0,0],
    [0,0,-50],
    [-50,0,0]
];
cpPositions.forEach(p=>{
    const cp = new THREE.Mesh(
        new THREE.TorusGeometry(3,0.5,12,24),
        new THREE.MeshStandardMaterial({color:0xffff00})
    );
    cp.rotation.x=Math.PI/2;
    cp.position.set(p[0],1,p[2]);
    scene.add(cp);
    checkpoints.push(cp);
});

/* ===== VARIABLES ===== */
let speed=0, aiSpeed=0.3;
let lap=1, cpIndex=0;
let canMove=false;
const keys={};

addEventListener("keydown",e=>keys[e.key]=true);
addEventListener("keyup",e=>keys[e.key]=false);

/* ===== CAMERA ===== */
camera.position.set(0,6,-12);

/* ===== COUNTDOWN ===== */
let count=3;
const msg=document.getElementById("msg");
const timer=setInterval(()=>{
    count--;
    msg.innerText=count>0?count:"GO!";
    if(count<0){
        clearInterval(timer);
        msg.style.display="none";
        canMove=true;
    }
},1000);

/* ===== GAME LOOP ===== */
function animate(){
    requestAnimationFrame(animate);

    if(canMove){
        if(keys["w"]||keys["ArrowUp"]) speed+=0.02;
        if(keys["s"]||keys["ArrowDown"]) speed-=0.02;
    }

    speed*=0.97;
    speed=Math.max(-0.5,Math.min(0.5,speed));

    if(keys["a"]||keys["ArrowLeft"]) player.rotation.y+=0.04*speed*4;
    if(keys["d"]||keys["ArrowRight"]) player.rotation.y-=0.04*speed*4;

    player.translateZ(speed);

    /* AI follows checkpoints */
    const target = checkpoints[cpIndex];
    ai.lookAt(target.position);
    ai.translateZ(aiSpeed);

    /* Checkpoint detection */
    if(player.position.distanceTo(checkpoints[cpIndex].position)<5){
        cpIndex++;
        if(cpIndex>=checkpoints.length){
            cpIndex=0;
            lap++;
            if(lap>3){
                msg.innerText="YOU WIN!";
                msg.style.display="flex";
                canMove=false;
            }
        }
    }

    /* Camera */
    const camPos = new THREE.Vector3(0,6,-12).applyMatrix4(player.matrixWorld);
    camera.position.lerp(camPos,0.1);
    camera.lookAt(player.position);

    /* HUD */
    document.getElementById("speed").innerText=Math.abs(Math.round(speed*300));
    document.getElementById("lap").innerText=lap;
    document.getElementById("cp").innerText=cpIndex;

    renderer.render(scene,camera);
}
animate();

/* ===== RESIZE ===== */
addEventListener("resize",()=>{
    camera.aspect=innerWidth/innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(innerWidth,innerHeight);
});
</script>

</body>
</html><img width="1024" height="1024" alt="1768728202030" src="https://github.com/user-attachments/assets/c0f74a3f-7b92-4b7b-bac2-5c74eabdd85c" />
# racing-game.html
