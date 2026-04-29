<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Spotify Style Player</title>

<style>
body {
  margin:0;
  background:transparent;
  font-family: Arial, sans-serif;
}

/* ===== CONFIG AREA IS INSIDE JS BELOW ===== */

.player {
  display:flex;
  align-items:center;
  padding:14px;
  border-radius:18px;
  box-sizing:border-box;
  box-shadow:0 12px 40px rgba(0,0,0,0.45);
  width:100%;
}

.cover {
  border-radius:10px;
  object-fit:cover;
}

.content {
  margin-left:12px;
  flex:1;
  display:flex;
  flex-direction:column;
  justify-content:center;
}

.title {
  font-size:14px;
  font-weight:600;
  margin-bottom:10px;
  white-space:nowrap;
  overflow:hidden;
  text-overflow:ellipsis;
}

.controls {
  display:flex;
  align-items:center;
  gap:10px;
}

.play {
  width:34px;
  height:34px;
  border-radius:50%;
  border:none;
  background:white;
  display:flex;
  align-items:center;
  justify-content:center;
  cursor:pointer;
}

.play svg {
  width:18px;
  height:18px;
  fill:black;
}

.bar {
  flex:1;
  height:5px;
  background:#333;
  border-radius:999px;
  position:relative;
  cursor:pointer;
}

.progress {
  height:100%;
  width:0%;
  border-radius:999px;
}

.knob {
  width:10px;
  height:10px;
  background:white;
  border-radius:50%;
  position:absolute;
  top:50%;
  transform:translate(-50%,-50%);
  left:0%;
}

.time {
  font-size:12px;
  opacity:0.7;
  width:42px;
  text-align:right;
}
</style>
</head>

<body>

<div id="player" class="player"></div>
<div id="yt" style="display:none;"></div>

<script src="https://www.youtube.com/iframe_api"></script>

<script>
/* =========================
   🎛️ CONFIG (EDIT THIS)
========================= */

const CONFIG = {
  videoId: "VIDEO_ID",
  title: "Your Song Title",
  cover: "https://img.youtube.com/vi/VIDEO_ID/hqdefault.jpg",

  size: {
    width: "100%",
    height: "90px"
  },

  colors: {
    background: "#121212",
    accent: "#1db954",
    text: "#ffffff"
  }
};

/* =========================
   BUILD UI
========================= */

const root = document.getElementById("player");

root.style.width = CONFIG.size.width;
root.style.height = CONFIG.size.height;
root.style.background = CONFIG.colors.background;
root.style.color = CONFIG.colors.text;

root.innerHTML = `
<img class="cover" src="${CONFIG.cover}" style="width:64px;height:64px;">
<div class="content">
  <div class="title">${CONFIG.title}</div>

  <div class="controls">
    <button class="play" id="playBtn">
      <svg id="playIcon" viewBox="0 0 24 24">
        <path d="M8 5v14l11-7z"></path>
      </svg>
      <svg id="pauseIcon" viewBox="0 0 24 24" style="display:none;">
        <path d="M6 5h4v14H6zm8 0h4v14h-4z"></path>
      </svg>
    </button>

    <div class="bar" id="bar">
      <div class="progress" id="progress"></div>
      <div class="knob" id="knob"></div>
    </div>

    <div class="time" id="time">0:00</div>
  </div>
</div>
`;

/* =========================
   YOUTUBE PLAYER
========================= */

let player;
let playing = false;
let duration = 0;
let dragging = false;

const playBtn = document.getElementById("playBtn");
const playIcon = document.getElementById("playIcon");
const pauseIcon = document.getElementById("pauseIcon");

const bar = document.getElementById("bar");
const progress = document.getElementById("progress");
const knob = document.getElementById("knob");
const time = document.getElementById("time");

function format(s){
  const m = Math.floor(s/60);
  const r = Math.floor(s%60);
  return `${m}:${r.toString().padStart(2,"0")}`;
}

function onYouTubeIframeAPIReady(){
  player = new YT.Player("yt", {
    height:"0",
    width:"0",
    videoId: CONFIG.videoId,
    playerVars:{controls:0},
    events:{
      onReady:()=>{
        duration = player.getDuration();
        loop();
      }
    }
  });
}

/* PLAY / PAUSE */
playBtn.onclick = () => {
  if(!player) return;

  if(playing){
    player.pauseVideo();
    playIcon.style.display="block";
    pauseIcon.style.display="none";
  } else {
    player.playVideo();
    playIcon.style.display="none";
    pauseIcon.style.display="block";
  }

  playing = !playing;
};

/* LOOP */
function loop(){
  if(player && player.getCurrentTime && !dragging){
    const t = player.getCurrentTime();
    const p = (t/duration)*100;

    progress.style.width = p+"%";
    knob.style.left = p+"%";
    time.textContent = format(t);
  }

  requestAnimationFrame(loop);
}

/* SEEK */
function seek(e){
  const r = bar.getBoundingClientRect();
  let x = (e.touches?e.touches[0].clientX:e.clientX) - r.left;
  x = Math.max(0,Math.min(r.width,x));

  const ratio = x/r.width;

  progress.style.width = (ratio*100)+"%";
  knob.style.left = (ratio*100)+"%";

  player.seekTo(ratio*duration,true);
}

bar.onclick = seek;

knob.onmousedown = ()=>dragging=true;
document.onmouseup = ()=>dragging=false;
document.onmousemove = e=>dragging&&seek(e);

knob.ontouchstart = ()=>dragging=true;
document.ontouchend = ()=>dragging=false;
knob.ontouchmove = e=>dragging&&seek(e);
</script>

</body>
</html>
